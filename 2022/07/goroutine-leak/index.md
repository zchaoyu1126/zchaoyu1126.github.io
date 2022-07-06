# Goroutine 泄漏


Golang优秀的GC机制，虽然可以让其不需要像C++中那样手动释放对象，从而避免内存泄漏，但goroutine的生命周期需要开发者格外注意。如果一个goroutine意外阻塞无法退出，就会造成goroutine泄漏的问题。随着占用的资源越来越多，只能重启服务器暂时缓解，还是得从根源上解决问题。本文对goroutine的泄漏场景进行了总结，是对煎鱼大佬博文的梳理，内容并非原创，参考原文已贴在末尾。

<!--more--> 

## 0 泄漏原因

- Goroutine内执行与channel或mutex有关的读写操作，由于长时间得不到响应阻塞。(2.1，2.2)
- Goroutine 内的业务逻辑进入长时间等待，有不断新增的 Goroutine 进入等待。(2.3)

## 1 排查方法

- 调用 `runtime.NumGoroutine()` 方法来获取 Goroutine 的运行数量
- 使用PProf工具
- 在实际生产过程中，更多的是使用PProf工具来进行分析是否发生了内存泄漏，具体请参考如下链接。

[PProf工具使用](https://www.notion.so/PProf-ef9b8a6c2db64bd9a51f9be9f1c66c06)

## 2 泄漏场景

### 2.1 通道使用不当

**2.1.1 通道为nil**

```go
package main

import (
		"fmt"
		"runtime"
		"time"
)

func send(ch chan<- int) {
		ch <- 1
		ch <- 2
}

func receive(ch <-chan int) {
		<-ch
		<-ch
}

func main() {
		var ch chan int  // 应修改为 ch := make(chan int)
		go send(ch)
		go receive(ch)
	
		time.Sleep(time.Second)
		fmt.Println(runtime.NumGoroutine())
}
// 3
```

**2.1.2 无人消费数据导致通道阻塞**

```go
package main

import (
		"fmt"
		"runtime"
		"time"
)

func send(ch chan<- int) {
		ch <- 1
		ch <- 2
}

func receive(ch <-chan int) {
		<-ch
}

func main() {
		ch := make(chan int)
		go send(ch)
		go receive(ch)
	
		time.Sleep(time.Second)
		fmt.Println(runtime.NumGoroutine())
}
// 2
// send向通道中发送了两个值，但只被消费了一个
// 所以go send(ch)启动的协程被阻塞，结果会输出2
```

下面，使用pprof工具进行分析

```go
package main

import (
		"net/http"
		_ "net/http/pprof"
)

func send(ch chan<- int) {
		ch <- 1
		ch <- 2
}

func receive(ch <-chan int) {
		<-ch
}

func pprofService() error {
		return http.ListenAndServe(":6060", nil)
}

func main() {
		done := make(chan error)
		ch := make(chan int)
	
		go func() {
				done <- pprofService()
		}()
	
		for i := 0; i < 100; i++ {
				go send(ch)
		}
	
		for i := 0; i < 100; i++ {
				go receive(ch)
		}
	
		<-done
}
// 按上述程序所示，启动100个send协程后，由于没法完全消费数据，
// 所以造成goroutine的大量泄漏，其火焰图如下所示
```

{{< image src="flamegraph.png" caption="PProf 火焰图" src_s="flamegraph.png" src_l="flamegraph.png" >}}

**2.1.3 通道无法读取数据导致阻塞**

```go
package main

import (
		"fmt"
		"runtime"
		"time"
)

func send(ch chan<- int) {
		ch <- 1
		close(ch)
}

func receive(ch <-chan int) {
		<-ch
		<-ch
}

func main() {
		ch := make(chan int)
		go send(ch)
		go receive(ch)
	
		time.Sleep(time.Second)
		fmt.Println(runtime.NumGoroutine())
}
// 2
```

### 2.2 互斥锁或同步锁使用不当

**2.2.1 互斥锁使用不当**

```go
package main

import (
		"fmt"
		"runtime"
		"sync"
		"time"
)

var counter int
var mutex sync.Mutex

func main() {
		for i := 0; i < 5; i++ {
				go func() {
						mutex.Lock()
						// defer mutex.Unlock()
						counter++
				}()
		}
	
		time.Sleep(time.Second)
		fmt.Println(runtime.NumGoroutine())
}
// 5
// 第一个协程加锁之后没有解锁，counter++完之后直接退出
// 使得后面四个协程无法对counter加锁，从而造成阻塞
```

**2.2.2 同步锁使用不当**

```go
package main

import (
		"fmt"
		"sync"
)

func main() {
		var wg sync.WaitGroup
		for i := 0; i < 5; i++ {
				wg.Add(6)
				go func(x int) {
						defer wg.Done()
						fmt.Println("goroutine: ", x)
				}(i)
		}
		wg.Wait()
}
// deadlock
// 由于waitgroup使用不当，直接造成主协程阻塞（死锁）
```

### 2.3 奇怪的慢等待

```go
func main() {
    for {
        go func() {
            _, err := http.Get("https://www.xxx.com/")
            if err != nil {
                fmt.Printf("http.Get err: %v\n", err)
            }
            // do something...
    }()

    time.Sleep(time.Second * 1)
    fmt.Println("goroutines: ", runtime.NumGoroutine())
    }
}
// http.Client默认没有设置超时时间
// 如果Get请求一直得不到满足，那么该协程就一直无法退出，从而造成泄漏
```

## 3 参考资料

[https://segmentfault.com/a/1190000040161853?utm_source=sf-similar-article](https://segmentfault.com/a/1190000040161853?utm_source=sf-similar-article)

[https://www.php.cn/be/go/490491.html](https://www.php.cn/be/go/490491.html)
