# Ch14 包与变量作用域


# ch14 包与变量作用域

## **作用域**

在正式引入匿名函数与闭包之前，需要了解变量的作用域与生命周期两个概念。

这是两个完全不同的概念，

- 作用域：源代码中可以有效使用这个名字的范围，是一个编译时的属性。
- 生命周期：程序运行时变量存在的有效时间段，在此时间区域内它可以被程序的其他部分引用，是一个运行时的概念。

### **作用域级别**

- 语句块内部声明的变量在外部是无法被访问的，此语句块即内部所声明变量的作用域。
    - 语句块：由花括弧所包含的一系列语句，例如函数体或循环体包裹的一系列语句。
- 变量的作用域可分为以下四个级别
    - 元作用域：在函数内部由`switch、if、for`等引入的作用域
        - 在switch、for、if后的初始化条件中声明的变量，其作用域范围为整个switch、if、for语句块
        - 对于for和if，每个语句块，即花括号包裹的部分，代表一个更小的作用域；对于switch，每个分支代表一个更小的作用域。
    - 函数作用域：在函数，如`main`函数中声明的变量，其作用域为整个函数体
    - 包作用域：任何在函数外声明的变量可在同一个包的任何源文件中进行访问，其作用域为整个包。
    - 全作用域：特别地，当在函数外声明的**变量首字母大写**时，该变量可通过`packageName.VarName`的方式在包外进行访问。
        - **变量或者函数**以**大写字母**开头时，此时称该变量或函数是在包外**可见的或可导出的，即它们可在包外被访问**，**具体内容参考下文中的包**。

![0014_1.png](ch14%20%E5%8C%85%E4%B8%8E%E5%8F%98%E9%87%8F%E4%BD%9C%E7%94%A8%E5%9F%9F%204e8d121d02a54446b6a381d330f13a4c/0014_1.png)

- 一个程序可能包含多个同名变量，但只要属于不同的作用域就不会产生冲突，例如
    - 循环内声明一个的局部变量（元作用域）可以和包级的变量（包作用域）同名
    - 在两个不同的循环内，各声明一个名为`x`的变量，此时也不会产生冲突，`x(循环A中)`与`x'(循环B中)`在各自的元作用域内有效
    - 对于全作用域，如果在包`pkg1`和`pkg2`中各有一个名为`StuNum`的可导出变量（具体内容参考包），此时也不会发生冲突，因为在使用时必须加上包名，即`pkg1.StuNum`，`pkg2.StuNum`
- 编译器解析变量时，会查找该变量的定义，从当前所在的作用域出发，最内部作用域中的声明会被首先找到，此时就屏蔽了外部作用域的同名声明。

### **具体实例**

- 含如此多同名变量，是非常糟糕的编程习惯，在平时应避免这种情况的发生。

`test1.go`

```go
package test

import "fmt"

var Num int = 20
var x int = 30

func TestScope1() {
    fmt.Println("the x in main package is ", x)
    x := 10
    fmt.Println("the x in function is ", x)
    // from now on, x refers to the x declared in the function
    for x := 0; x < 5; x++ {
        // in this loop x refers to the variation declared last line
        fmt.Println("the x in loop is ", x)
    }
    fmt.Println("the x in function is still", x)

    if x := 2*x; x > 10 {
        // declare varation tmp just for testing wheather can input tmp in else block or not
        tmp := x;
        fmt.Println("the x in if block is", x)
        fmt.Println(tmp, " plus 5 is", tmp+5)
    } else {
        fmt.Println("try to get the value of tmp")
        // fmt.Println(tmp)
        // .\main.go:26:15: undefined: tmp
    }
}

func print6X() {
    fmt.Println("use printX output x * 6 is", x*6)
}
```

`test2.go`

```go
package test

import "fmt"

func TestScope2() {
    print6X()
    fmt.Println("output x in TestScope2, x is ", x)
}
```

`main.go`

```go
package main

import (
    "fmt"
    "programs/test"
)

func main() {
    fmt.Println("Num in pkg test is ", test.Num)
    test.TestScope1()
    test.TestScope2()
}
```

```
Num in pkg test is  20
the x in main package is  30
the x in function is  10
the x in loop is  0
the x in loop is  1
the x in loop is  2
the x in loop is  3
the x in loop is  4
the x in function is still 10
the x in if block is 20
20  plus 5 is 25
use printX output x * 6 is 180
output x in TestScope2, x is  30
```

## **包(package)**

- Go语言中的包和其他语言中的库，包等概念类似，目的都是为了支持模块化、封装、单独编译和代码重用。
- 一个包由位于同名目录下的一个或多个.go源代码文件组成。
- 每个包都对应一个独立的名字空间。例如，在`image`包中的`Decode`函数和在`unicode/utf16`包中的 `Decode`函数是不同的。要在外部引用该函数，必须显式使用`image.Decode`或`utf16.Decode`形式访问。
- 包可以**控制变量或函数等在包外是否可见**来**隐藏内部实现信息**

### **可见性**

变量或者函数以**大写字母**开头时，此时称该变量或函数是在包外可见的或可导出的，即它们可在包外被访问。

### **导入包**

- 在Go语言程序中，每个包都有一个全局唯一的导入路径。
- 按照惯例，将导入路径的最后一个字段作为包名
- 使用GOPATH模式时，编译器会依据`$GOROOT/src/importpath`依次来寻找包，此外，不建议使用相对路径来导入包。
- 使用GoModule时，假设在根目录下使用`go mod init minidb`进行了初始化，则导入此根目录下的包路径为`minidb/packageName`

**单行导入**

```go
import "path1/pkg1"
import "path2/pkg2"
```

**多行导入**

```go
import (
    "path1/pkg1"
    "path2/pkg2"
)
```

**导入后自定义引用的包名**

```go
import customName "path/to/package"
```

指定新的包名，来避免导入重名包。

```go
import (
    "crypto/rand"
    mrand "math/rand" // alternative name mrand avoids conflict
)
```

**匿名导入包**

只导入包，让其执行`init`函数，而不使用任何包内的结构和类型，也不调用包内的任何函数

```go
import _ "path/to/package"
```

### **包的初始化**

在某些情况下，在调用已经封装好的模块时，往往需要人为地执行一个初始化函数，例如在初学数据结构时，会经常遇到`StackInit`，`QueueInit`，`GraphInit`等函数。这些函数需要开发者自己手动调用，那么这个过程可能发生遗漏或者错误。

由此，迫切地希望被引用的包，能够获得启动代码的通知，在程序启动时做一些自行执行一些初始化工作。基于此，Go语言中设置了`init`函数来完成此项功能。

- 每个`.go`文件都可包含一个`init`函数
- `init`函数会在程序执行前(`main`函数执行前)被自动调用
- 调用顺序与包中变量的初始化顺序如下图所示
  
    ![0014_2.png](ch14%20%E5%8C%85%E4%B8%8E%E5%8F%98%E9%87%8F%E4%BD%9C%E7%94%A8%E5%9F%9F%204e8d121d02a54446b6a381d330f13a4c/0014_2.png)
    
- 同一个包中的多个`init`函数调用顺序不可预期
- `init`函数不能被其他函数调用
