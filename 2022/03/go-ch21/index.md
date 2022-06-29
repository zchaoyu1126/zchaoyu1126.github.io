# Ch21 协程


# ch21 协程

- Go语言的并发通过goroutine特性完成。goroutine类似于线程，但是可以根据需要创建多个goroutine并发工作。
- goroutine是由Go语言的运行时调度完成，而线程则是由操作系统调度完成。
- Go语言提供channel以便于在多个goroutine间进行通信。
- goroutine和channel是Go语言秉承的CSP（Communicating Sequential Process)并发模式的重要实现基础。
- Go程序从main包的main函数开始，在程序启动时，Go程序就会为main()函数创建一个默认的goroutine。

## **创建goroutine**

Go程序中使用go关键字为一个函数创建一个goroutine，一个函数可以被创建多个goroutine，一个goroutine必定对应一个函数。

```go
go functionName(params type)
```

注：返回值会被忽略，如果要通信，使用channel

```go
package main

import (
    "fmt"
    "time"
)

func running() {
    var times int
    for {
        times++
        fmt.Println("tick", times)
        time.Sleep(time.Second)
    }
}

func main() {
    go running()
    var input string
    fmt.Scanln(&input)
}
```

使用`go running()`之后，会创建一个协程，每个一秒输出一个tick，在用户的输入读取完毕后，整个程序结束，goroutine被终止。

Go程序在启动时，runtime会默认为main()函数创建一个goroutine，main()函数的goroutine中执行到go running()语句时，归属于running()函数的goroutine被创建，running()函数开始在自己的goroutine中执行，此时，main()函数继续执行，两个goroutine通过GO程序的调度机制同时运行。

## **匿名函数创建goroutine**

```go
package main
import (
    "fmt"
    "time"
)

func main() {
    go func() {
        var times int

        for {
            times++
            fmt.Println("ticks", times)
            time.Sleep(time.Second)
        }
    }()

    var input string
    fmt.Scanln(&input)
}
```

注意事项：

1. 所有goroutine在main()函数结束时会一同结束
2. goroutine的调度依赖于调度器的实现和运行环境
3. golang无法kill 一个 goroutine，但可以通过一个chan给它发送消息让它退出，更多的信息见
   
    [ch25 并发编程](ch25%20%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%20f0cdfb2499f74bc48b4fc5ece4619ee1.md)
    
    - 使用通道让goroutine退出的示例如下
      
        ```go
        package main
        
        import (
        		"fmt"
        		"time"
        )
        
        func main() {
        		exist := make(chan int)
        		go func(exist chan int) {
        				for {
        						select {
        						case <-time.After(time.Second):
        								fmt.Println("hi")
        						case <-exist:
        								goto End
        				}
        		}
        		End:
        				fmt.Println("finished")
        		}(exist)
        	
        		time.Sleep(time.Second * 5)
        		exist <- 2
        		time.Sleep(time.Second * 2)
        }
        ```
        

## **调整并发的运行性能**

Go程序运行时(runtime)实现了一个小型的任务调度器，这套调度器的工作原理类似于操作系统调度线程，Go程序调度器可以高效地将CPU资源分配给每一个任务。

传统逻辑中，开发者需要维护线程池中线程与CPU核心数量的对应关系。同样的，Go中可以通过设置runtime.GOMAXPROCS()函数做到，`runtime.GOMAXPROCS(runtime.NumCPU())`

[ch40 goroutine调度机制](ch40%20goroutine%E8%B0%83%E5%BA%A6%E6%9C%BA%E5%88%B6%20616893eae87143efbdd221a1932c4905.md)
