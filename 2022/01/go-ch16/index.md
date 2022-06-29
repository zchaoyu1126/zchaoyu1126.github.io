# Ch16 defer


# ch16 defer

## defer的概念

Go语言中`defer`语句会将其后面跟随的语句进行延迟处理，当执行到该条语句时，**函数和参数表达式得到计算**，但直到包含该defer语句的函数执行完毕时，**defer后的函数才会被执行，**不论包含defer语句的函数是通过return正常结束，还是由于panic导致的异常结束。

可以在一个函数中执行多条defer语句，它们的执行顺序与声明顺序相反。

注意下述代码中输出的`num`的值，紧跟着defer后的语句输出的num值为10，而不是30

```go
package main

import "fmt"

func main(){
    defer fmt.Println("1")
    defer fmt.Println("2")
    defer fmt.Println("3")
    outputNum()
}

func outputNum() bool {
    num := 10
    defer fmt.Printf("%d with defer\n", num)
    num += 20
    fmt.Printf(("%d without defer\n", num)
    return true
}
// 30 without defer
// 10 with defer
// 3
// 2
// 1
```

## defer的用途

`defer`语句经常被用于处理成对的操作，如打开、关闭、连接、断开连接、加锁、释放锁。通过defer机制，不论函数逻辑多复杂，都能保证在任何执行路径下，资源被释放。释放资源的defer应该直接跟在请求资源的语句后。

```go
package main

import "fmt"

func main() {
    doDBOperations()
}

func connectToDB() {
    fmt.Println("ok, connected to db")
}

func disconnectFromDB() {
    fmt.Println("ok, disconnected from db")
}

func doDBOperations() bool {
    connectToDB()
    fmt.Println("Defering the database disconnect.")
    defer disconnectFromDB() //function called here with defer
    fmt.Println("Doing some DB operations ...")
    fmt.Println("Oops! some crash or network error ...")
    fmt.Println("Returning from function here!")
    return false //terminate the program
    // deferred function executed here just before actually returning, even if
    // there is a return or abnormal termination before
}
```

**使用defer并发解锁**

在下面的例子中会在函数中并发使用map，为防止竞态问题，使用`sync.Mutex`进行加锁，参见下面的代码

```go
var (
    valueByKey = make(map[string]int)
    valueByKeyGuard sync.Mutex
)

func readValue(key string) int {
    valueByKeyGuard.Lock()
    v := valueByKey[key]
    valueByKeyGuard.UnLock()
    return v
}
```

`map并不是并发安全的`，所以使用`sync.Mutex`互斥量保护map的访问，可以使用defer对上述语句进行简化

map和slice并不是并发安全的，具体的参考

[ch25 并发编程](ch25%20%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%20f0cdfb2499f74bc48b4fc5ece4619ee1.md)

```go
func readValue(key string) int {
    valueByKeyGuard.Lock()
    defer valueByKeyGuard.Unlock()
    return valueByKey[key]
}
```

**使用defer释放文件句柄**

文件的操作需要经过打开文件、获取和操作文件资源、关闭资源几个过程，如果在操作完毕后不关闭文件资源，进程将一直无法释放文件资源。

```go
func fileSize(filename string) int64 {
    f, err := os.Open(filename)

    if err != nil {
        return 0
    }

    info, err := f.Stat()
    if err != nil {
        f.Close()
        return 0
    }

    size := info.Size()
    f.Close()
    return size
}
```

使用defer简化后，

```go
func fileSize(filename string) int64 {
    f, err := os.Open(filename)
    if err != nil {
        return 0
    }
    defer f.Close()

    if info, err := f.Stat(); err == nil {
        return info.Size()
    }
    return 0
}
```

## **Defer a function call (with return value)**

[https://yourbasic.org/golang/defer/](https://yourbasic.org/golang/defer/)

```go
func foo() (result string) {
    defer func() {
        result = "Change World" // change value at the very last moment
    }()
    return "Hello World"
}
```

```go
package main

import "fmt"

func foo() (result int) {
		defer func() {
		result = 4
	}()
		result = 2 * 3
		return 1
}

func main() {
		res := foo()
		fmt.Println(res) //4
}
```

带命名返回值的函数，使用 `return 1`这种方式返回时，其实进行了两步操作，即return 1等价于

```go
res = 1
return res
```

所以defer语句，在函数返回之前，又对res进行了修改，所以最终的结果是4
