# Ch17 匿名函数与闭包


# ch17 error,panic与recover

## **运行时错误**

Go语言的错误处理思想及设计包含以下特征

- 一个可能造成错误的函数，需要返回值中返回一个错误接口（error）。如果调用是成功的，错误接口将返回nil，否则返回错误。
- 在函数调用后需要检查错误，如果发生错误，则进行必要的错误处理。

Go没有类似Java或.Net中的异常处理机制，虽然可以使用defer、recover、panic进行模拟，但官方并不推荐。Go的设计者认为其他语言的异常机制已被过度使用，上层逻辑需要为函数发生的错误付出太多的资源。Go语言希望开发者将错误处理是为正常开发必须实现的环节，正确地处理每一个可能发生错误的函数，同时Go语言使用返回值返回错误的机制，也能大幅降低编译器、运行时处理错误的复杂度，让开发者真正地掌握错误的处理。

[ch34 错误处理机制](ch34%20%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86%E6%9C%BA%E5%88%B6%2052f3c28960354df8b83c62a3fc6b2584.md)

### **error接口**

```go
type error interface {
    Error() string
}
```

所有符合Error() string格式的方法，都能实现错误接口，该方法返回错误的具体描述。

[ch18 自定义类型与结构体](ch18%20%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B%E4%B8%8E%E7%BB%93%E6%9E%84%E4%BD%93%2076cc67414a6f417592c955fab74ebf63.md)

[ch19 接口](ch19%20%E6%8E%A5%E5%8F%A3%20d0c5e5fb1a054e34a7404085ea76be7d.md)

### **自定义错误**

**方法一:** 使用errors包进行错误的定义，格式如下

```
var err = errors.New("this is an error")
```

错误字符串由于相对固定，故一般在包作用域声明，应尽量减少在使用时直接使用`errors.New()`返回

```go
package main

import (
    "errors"
    "fmt"
)

var errDivisionByZero = errors.New("division by zero")

func Div(dividend, divisor float64) (float64, error) {
    if divisor == 0 {
        return math.NaN(), errDivisionByZero
    }
    return dividend/divisor, nil
}

func main() {
    res, nil := test.Div(20, 0)
    fmt.Println(res, nil)
}
```

**拓展**

`errors.New()`的底层代码

```go
func New(text string) error {
    return &errorString(text)
}

// errorString实现了error接口
func (e *errorString) Error() string {
    return e.s
}

func errorString struct {
    s string
}
```

**方法二:** 自定义结构体，记录一些文件名，行数等信息，使得error信息更加丰富

```go
package main

import (
    "math"
    "fmt"
    "runtime"
)

type DivError struct{
    FileName string
    Line     int
}

func (d *DivError) Error() string {
    return fmt.Sprintf("%s:%d division by zero", d.FileName, d.Line)
}

func newDivError(filename string, line int) error {
    return &DivError{FileName:filename, Line:line}
}

func Div(dividend, divisor float64) (float64, error) {
    if divisor == 0 {
        _, filename, line, _ := runtime.Caller(1)
        return math.NaN(), newDivError(filename, line)
    }
    return dividend/divisor, nil
}

func main() {
    res, nil := Div(20, 0)
    fmt.Println(res, nil)
}
```

func Caller(skip int) (pc uintptr, file string, line int, ok bool)

Caller报告当前go协程调用栈所执行的函数的文件和行号信息。实参skip为上溯的栈帧数，0表示Caller的调用者（Caller所在的调用栈）。（由于历史原因，skip的意思在Caller和Callers中并不相同。）函数的返回值为调用栈标识符、文件名、该调用在文件中的行号。如果无法获得信息，ok会被设为false。

### error与**类型断言**

> 类型断言将在接口中详细说明，在此只需记住 var.(type) 可以获取变量的类型
> 

如果一个函数返回多个error，例如如下代码，可用类型断言判断类型

```go
package main

import (
    "math"
    "fmt"
    "runtime"
)

type InputError struct {
    FileName string
    Line     int
    Num      float64
}

func (i *InputError) Error() string {
    return fmt.Sprintf("%s:%d %.2g input error", i.FileName, i.Line, i.Num)
}

func newInputError(filename string, line int, num float64) *InputError {
    return &InputError{FileName:filename, Line:line, Num:num}
}

type DivError struct{
    FileName string
    Line     int
}

func (d *DivError) Error() string {
    return fmt.Sprintf("%s:%d division by zero", d.FileName, d.Line)
}

func newDivError(filename string, line int) error {
    return &DivError{FileName:filename, Line:line}
}

func Div(dividend, divisor float64) (float64, error) {
    if dividend < 0 {
        _, filename, line, _ := runtime.Caller(1)
        return math.NaN(), newInputError(filename, line, dividend)
    } else if divisor < 0 {
        _, filename, line, _ := runtime.Caller(1)
        return math.NaN(), newInputError(filename, line, divisor)
    }
    if divisor == 0 {
        _, filename, line, _ := runtime.Caller(1)
        return math.NaN(), newDivError(filename, line)
    }
    return dividend/divisor, nil
}

func main() {
    res, err := Div(20, 0)
    switch detail := err.(type) {
        case *InputError:
        fmt.Printf("%s:%d %.2g, input doesn't fit\n", detail.FileName, detail.Line, detail.Num)
        case *DivError:
        fmt.Printf("%s:%d division is zero\n", detail.FileName, detail.Line)
        default:
        fmt.Println("other error")
    }
    fmt.Println(res)
}
```

> https://blog.csdn.net/shida_csdn/article/details/88057989
> 
> 
> [https://stackoverflow.com/questions/23172219/cannot-type-switch-on-non-interface-value](https://stackoverflow.com/questions/23172219/cannot-type-switch-on-non-interface-value)
> 

## **panic**

Go语言可以在程序中手动触发宕机，让程序崩溃，以便开发者及时发现错误。

panic的参数可以是任意类型，后文提到的recover会接收从panic中发出的内容。

```go
package main

func main() {
    panic("crash")
}
```

### **主动触发panic**

regexp是Go语言的正则表达式包，正则表达式需要编译后才能使用，而且编译必须是成功的。

```go
1. func Compile(exptr string) (*Regexp, error)
2. func MustCompile(str string) *Regexp
```

第一个函数适用于在编译错误时获得编译错误并进行处理，同时继续后续执行的环境。

第二个函数适用于无需处理正则表达式错误的情况，其代码为

```go
func MustCompile(str string) *Regexp {
    regexp, error := Complie(str)
    if error != nil {
        panic(`regexp: Compile(` + quote(str) + `): ` + error.Error())
    }
    return regexp
}
```

手动宕机进行报错的方式不是一种偷懒的方式，反而能迅速报错，终止程序继续运行，防止更大的错误产生。

不过，如果任何错误都用宕机处理，也不是良好的设计，因此应根据需要来决定是否使用当即进行报错。

### **延迟语句**

当`panic`发生时，`panic`后面的代码不会被执行，但是在`panic`前的`defer`语句会在宕机前发生作用。

利用该特性，可以用来在宕机发生前进行一些必要的处理。

```go
package main

import "fmt"

func main() {
    defer fmt.Println("test1")
    defer fmt.Println("test2")
    panic("crash")
}
```

```go
test2
test1
panic: crash

goroutine 1 [running]:
main.main()
        E:/workspace_go/main.go:8 +0xe5
```

## **recover**

无论是由runtime层抛出的panic，还是在代码中主动触发的panic，都可以配合defer和recover实现错误的捕捉和恢复，让代码在发生崩溃后允许继续运行。

```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("Start.....")
    defer func() {
        err := recover()
        if err != nil {
            fmt.Println("recover")
        }
    }()
    panic("crash")

    // unreachable code
    fmt.Println("End.....")
}
```

当panic触发崩溃时，main函数将结束运行，此时defer后的闭包将会发生调用，调用结束后，整个程序结束。

下面的代码实现了一个ProtectRun()函数，该函数传入一个匿名函数或闭包后的执行函数，当传入函数以任何形式发生Panic崩溃后，可以将崩溃的错误打印出来，同时允许后面的代码继续运行，不会造成整个进程的崩溃。

```go
package main

import (
    "fmt"
    "runtime"
)

func ProtectRun(entry func())  {
    defer func() {
        err := recover()
        if err != nil {
            switch err.(type) {
                case runtime.Error:
                fmt.Println("runtime error: ", err)
                default:
                fmt.Println("error: ", err)
            }
        }
    }()

    entry()
}

func main() {
    fmt.Println("Start.....")

    ProtectRun(func() {
        fmt.Println("before crash")
        panic("crash")
        fmt.Println("after crash")
    })

    ProtectRun(func() {
        fmt.Println("before crash")
        // runtime error
        var a *int
        *a = 1
        fmt.Println("after crash")
    })
    fmt.Println("End.....")
}
```

```
Start.....
before crash
error:  crash
before crash
runtime error:  runtime error: invalid memory address or nil pointer dereference
End.....
```
