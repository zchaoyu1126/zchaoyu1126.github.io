# Ch22 通道


# ch22 通道

单纯地将函数并发执行是没有意义的，函数与函数间需要交换数据才能体现并发执行函数的意义。使用共享内存在不同的goroutine中容易发生竞态问题，为了保证数据交换的正确性，必须使用互斥量对内存进行加锁，这种做法势必造成性能问题。

Go提倡使用通信来共享内存，而不是用共享内存来通信。

## **通道的特性**

Go语言中的通道是一种特殊的类型，在任何时候，同时只能有一个goroutine访问通道进行发送和获取数据。

总是遵循先进先出的规则，保证收发数据的顺序。

## **声明与创建**

通道是引用类型，空值为nil，需要结合make使用

```go
var 通道变量 chan 通道类型

ch1 := make(chan int)
ch2 := make(chan interface{})
```

## **发送与接收数据**

### **发送数据**

把数据往通道中发送时，如果对方一直没有接收，那么发送操作将持续阻塞。

```go
package main

func main() {
    ch := make(chan int)
    ch <- 0
}
// fatal error: all goroutines are asleep - deadlock!
```

### **接收数据**

通道接收同样使用`<-`

- 每次只能接收一个元素
- 通道的收发操作在两个不同的goroutine间进行
  
    由于通道的数据在没有接收方处理时，数据发送方会持续阻塞，因此通道的接收必定是在另外一个goroutine中进行。
    
- 接收方将持续阻塞直到发送方发送数据

```go
data := <-ch
```

```go
package main

import "time"
import "fmt"

func main() {
    ch := make(chan int)
    go func() {
        time.Sleep(time.Second*5)
        fmt.Println("开始发送")
        ch <- 20
    }()

    data := <- ch
    fmt.Println("接收完毕, 数据为", data)
}
```

### **循环接收数据**

通道的数据接收可以借用for range语句进行多个元素的接收操作，格式如下：

```go
for data := range ch {

}
```

使用range读取channel值时，若chan没有关闭或者手动break，则会一直等待channel的值，并且阻塞后面语句的执行。两个例子如下

- 例1
  
    ```go
    package main
    
    import "fmt"
    
    func main() {
        ch := make(chan int)
    
        go func() {
            for i := 3; i >= 0; i-- {
                ch <- i
            }
        }()
    
        for data := range ch {
            fmt.Println(data)
            if data == 0 {
                break
            }
        }
    }
    ```
    
- 例2
  
    ```go
    package main
    
    import "fmt"
    import "time"
    
    func main() {
        ch := make(chan int)
        go func() {
            for v := range ch {
                fmt.Println(v)
            }
            fmt.Println("over")
        } ()
        fmt.Println("main start")
        time.Sleep(time.Second*2)
        ch <- 2
        time.Sleep(time.Second*3)
        fmt.Println("main over")
    }
    ```
    

## **通道用于并发同步**

这里的通道相当于信号量的作用，当goroutine结束之后，通知main函数的goroutine，然后main函数所以的goroutine被唤醒，继续执行。

```go
package main

import "fmt"

func main() {
    ch := make(chan int)
    go func() {
        func.Println("start goroutine")
        // do something
        fmt.Println("exit goroutine")
				ch <- 0
    }()
    fmt.Println("wait goroutine")
    <- ch
		// do something
    fmt.Println("all done")
}
```

```go
quit := make(chan bool)
go func() {
    for {
        select {
            case <- quit:
            return
            default:
            // Do other stuff
        }
    }
}()

// Do stuff

// Quit goroutine
close(quit)
```

## **单向通道**

Go的通道可以在声明时约束其操作方向，如只是发送或者只是接收。这种被约束方向的通道被称作单向通道。单向通道有利于代码接口的严谨性。

只能发送的通道类型为`chan<-`，只能接收的通道类型为`<-chan`，格式如下

```go
var 通道实例 chan<- 元素类型
var 通道实例 <-chan 元素类型
```

```go
ch1 := make(chan int)
var chSendOnly chan<- int = ch1
var chRecvOnly <-chan int = ch1

ch2 := make(<-chan int)
```

## **带缓冲的通道**

在无缓冲通道的基础上，为通道增加一个有限大小的存储空间形成带缓冲通道。

带缓冲通道在发送时无需等待接收方接收即可完成发送过程，并且不会发生阻塞，只有当存储空间满是才会发生阻塞。同时，如果缓冲通道中有数据，接收时将不会发生阻塞，直到通道中没有数据可读时，通道将会再度阻塞。

无缓冲通道其实可以看作长度永远为0的带缓冲通道。

```go
ch := make(chan int, 3)
```

## **通道的多路复用**

多路复用时通信和网络中的一个专业术语，多路复用通常表示在一个信道上传输多路信号或数据流的过程和技术。报话机同一时刻，只有一边进行收或者发的单边通信，而电话可以在说话的同时听到对方说完毕，类似的还有网线、光纤等。

Go语言中提供了select关键字，可以让使用通道完成多路复用，也就是说，同时响应多个通道的操作。select中的每个case都响应一个通道的收发过程。当收发完成时，就会触发case中响应的语句。多个操作在每次select中挑选一个进行响应。

```go
select {
		case 操作1：
				响应操作1
		case 操作2:
				响应操作2
		...
		default:
				操作
}
```

- 示例1：模拟远程过程调用
  
    ```go
    package main
    
    import (
        "errors"
        "fmt"
        "time"
    )
    
    func main() {
        ch := make(chan string)
        go RPCServer(ch)
    
        recv, err := RPCClient(ch, "hi")
        if err != nil {
            fmt.Println(err)
        } else {
            fmt.Println(recv)
        }
    }
    
    func RPCServer(ch chan string) {
        for {
            data := <-ch
            fmt.Println("server received data:", data)
            ch <- "roger"
        }
    }
    
    func RPCClient(ch chan string, req string) (string, error) {
        ch <- req
        select {
            case ack := <-ch:
            return ack, nil
            case <-time.After(time.Second):
            return "", errors.New("time out")
        }
    }
    ```
    
- 示例2：计时器与通道的应用1
  
    ```go
    package main
    
    import (
    	"fmt"
    	"time"
    )
    
    func main() {
    		ch1 := make(chan int)
    		ch2 := make(chan int)
    
    		go func(ch chan int) {
    				time.Sleep(time.Second * 2)
    				ch <- 2
    		}(ch1)
    	
    		go func(ch chan int) {
    				time.Sleep(time.Second * 3)
    				ch <- 3
    		}(ch2)
    	
    		select {
    		case val := <-ch1:
    				fmt.Println(val)
    		case val := <-ch2:
    				fmt.Println(val)
    		case <-time.After(time.Second):
    				fmt.Println("Time out")
    		default:
    				fmt.Println("none")
    		}
    }
    // none
    // 没有default分支，输出Time out
    ```
    
- 示例3：计时器与通道的应用2
  
    ticker，打点计时器
    
    timer，计时器
    
    计时任务的同步实现方式，因为select响应一次之后就会结束，所以得使用for循环，但是使用break无法退出外层循环，故使用goto 与 label。
    
    ```go
    package main
    
    import (
    		"fmt"
    		"time"
    )
    
    func main() {
    		ticker := time.NewTicker(time.Millisecond * 500)
    		stopper := time.NewTimer(time.Second * 2)
    	
    		var i int
    		for {
    				select {
    				case <-ticker.C:
    						i++
    						fmt.Println("tick", i)
    				case <-stopper.C:
    						goto StopHere
    				}
    		}
    StopHere:
    		fmt.Println("done")
    }
    ```
    
- 示例4：计时器与通道的应用3
  
    计时任务的异步执行方式，time.AfterFunc()中注册了一个回调函数，回调函数中通过通道发送已经完成任务的信号
    
    ```go
    package main
    
    import (
    		"time"
    )
    
    func main() {
    		exit := make(chan int)
    		time.AfterFunc(time.Second, func() {
    			exit <- 1
    		})
    		<-exit
    }
    ```
    

## **通道的关闭**

chan关闭时机：调用close(ch)且其中的数据全部被消费，或者主线程关闭。

- 被关闭的通道ch，并不会立刻被GC回收，并且不会被置为nil，但无法向其发送数据

```go
package main

import (
		"fmt"
)

func main() {
		ch := make(chan int, 3)
		ch <- 1
		ch <- 2
		close(ch)
		fmt.Println(cap(ch), len(ch))
		ch <- 3
}
// 3 2
// panic: send on closed channel
```

- 从已经关闭的通道接收数据不会阻塞，如果ch没有close，则for range会阻塞

```go
package main

import (
		"fmt"
)

func main() {
		ch := make(chan int, 3)
		ch <- 1
		ch <- 2
		close(ch)
		for i := range ch {
			fmt.Println(i)
		}
		val, ok := <-ch
		fmt.Println(val, ok)
		// 向已关闭，且无数据的通道继续读取数据，不会发生阻塞，但返回值为零值和false
}
// 1
// 2
// 0 false
```
