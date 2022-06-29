# Ch23 同步


# ch23 同步

Go程序可以使用通道进行多个goroutine之间的数据交换，但这仅仅是数据同步中的一种方法。在一些轻量级的场合可以使用atomic，sync.Mutex以及等待组sync.WaitGroup。

## 竞态检测

当多线程并发运行的程序竞争访问和修改同一块资源时，会发生竞态问题。

```go
package main

import (
		"fmt"
		"sync/atomic"
)

var seq int64

func main() {
		for i := 0; i < 10; i++ {
				go GenID()
		}
		fmt.Println(GenID())
}

func GenID() int64 {
		atomic.AddInt64(&seq, 1)
		return seq
}
```

使用原子操作函数atomic.AddInt64()对seq进行加1操作，但是在这里故意没有使用atomic.AddInt64()的返回值作为GenID()的返回值，所以会造成一个竞态问题。

编译运行时，加入`-race`参数，即`go run -race main.go`

会发生宕机错误，提示信息为`Warning: DATA RACE`

应修改为

```go
func GenID() int64 {
		return atomic.AddInt64(&seq, 1)
}
```

## 互斥锁

atomic: 对变量进行原子操作

sync.Mutex: 对一段代码加上互斥锁，对共享内存进行保护

```go
package main

import (
		"fmt"
		"sync"
)

var count int
var countGuard sync.Mutex

func GetCount() int {
		countGuard.Lock()
		defer countGuard.Unlock()
		return count
}

func SetCount(x int) {
		countGuard.Lock()
		count = x
		countGuard.Unlock()
}

func main() {
		SetCount(1)
		fmt.Println(GetCount())
}
```

一般情况下，建议将互斥锁的粒度设置的越小越好，从而降低因为共享访问时等待的时间。

一旦发生加锁，如果另外一个goroutine尝试继续加锁则会发生阻塞，直到这个countGuard被解锁。

```go
package main

import (
		"fmt"
		"sync"
		"time"
)

var count int
var countGuard sync.Mutex

func GetCount() int {
		countGuard.Lock()
		defer countGuard.Unlock()
		return count
}

func SetCount(x int) {
		countGuard.Lock()
		time.Sleep(time.Second * 5)
		count = x
		countGuard.Unlock()
}

func main() {
		go func() {
				SetCount(5)
		}()
		go func() {
				fmt.Println("waitting...")
				fmt.Println(GetCount())
		}()
		var quit int
		fmt.Scan(&quit)
}
```

## 读写互斥锁

在读多写少的环境中，可以优先使用读写互斥锁。

GetCount(1)这个gorouine睡眠了5秒，在这5秒内并没有释放读锁，而执行GetCount(0)的goroutine并没有受此影响而阻塞。

```go
package main

import (
		"fmt"
		"sync"
		"time"
)

var count int
var countGuard sync.RWMutex

func GetCount(opt int) int {
		countGuard.RLock()
		if opt == 1 {
				time.Sleep(time.Second * 5)
		}
		defer countGuard.RUnlock()
		return count
}

func SetCount(x int) {
		countGuard.Lock()
		count = x
		countGuard.Unlock()
}

func main() {
		go func() {
				fmt.Println(1, GetCount(1))
		}()
		go func() {
				fmt.Println(0, GetCount(0))
		}()
		var quit int
		fmt.Scan(&quit)
}
```

## 等待组

```
(wg *WaitGroup) Add(delta int)   等待组的计数器+delta
(wg *WaitGroup) Done()           等待组的计数器-1
(wg *WaitGroup) Wait()           当等待器计数器不等于0时阻塞直到变成0
```

```go
package main

import (
		"fmt"
		"net/http"
		"sync"
)

func main() {
		var wg sync.WaitGroup
		urls := []string{
				"https://www.github.com/",
				"https://www.qiniu.com",
				"https://www.golangtc.gom",
		}
		for _, url := range urls {
				wg.Add(1)
				go func(url string) {
						defer wg.Done()
						_, err := http.Get(url)
						fmt.Println(url, err)
				}(url)
				// 这样是不允许的，url会一直改变，导致请求的网址都是同一个
				// go func() {
				// 	defer wg.Done()
				// 	_, err := http.Get(url)
				// 	fmt.Println(url, err)
				// }()
		}
		wg.Wait()
		fmt.Println("over")
}
```
