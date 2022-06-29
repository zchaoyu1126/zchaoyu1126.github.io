# Ch6 常量与变量




## 1 常量

- 每个常量的潜在类型都是基础类型：布尔、字符串或数字
- 常量的声明语句定义了常量的名字和数值
- 常量的值在运行期间不可修改
- 常量表达式的值在编译期计算，而不是运行期
- 常量间的运算、包括算术运算、逻辑运算和比较运算的结果都是常量，对常量的类型转换操作或以下函数调用结果也是常量：`len`、`cap`、`real`、`imag`、`complex`、和`unsafe.Sizeof`
- 常量或常量表达式的值可以构成类型的一部分，例如数组长度

### 1. 1 Typed and Untyped Constants

{{<admonition quote>}}
以下英文内容摘抄自：[https://blog.csdn.net/pengpengzhou/article/details/107561792](https://blog.csdn.net/pengpengzhou/article/details/107561792)
{{</admonition>}}

众所周知，Go是强类型语言，在编译期间每一个变量的类型都是已知或可推导的。此外，即使是`int64`类型和`int32`类型的变量相加也需要进行强类型转换。

```go
var myFloat float64 = 21.54
var myInt int = 562
var myInt64 int64 = 120

// var res1 = myFloat + myInt // Not allowed
// var res2 = myInt + myInt64 // Not allowed
var res1 = myFloat + float64(myInt)
var res2 = myInt + int(myInt64)
```

但，如下的代码却是合法的，不需要对10进行强制类型转换，其中的原因在于Golang对于常量的处理

```go
var myInt32 int32 = 10
var myInt int = 10
var myFloat64 float64 = 10
var myComplex complex64 = 10
```

### 1.2 Untyped Constants

Golang中的任何一个常量，除非人为设置一个类型，否则无论是有名常量（用const定义的）还是无名常量（字面量）都是`untyped`

```
1                   // untyped integer constant
4.5                 // untyped floating-point constant
true                // untyped boolean constant
"Hello"             // untyped string constant

// 或者如下定义
const a = 1         // untyped integer constant
const f = 4.5       // untyped floating-point constant
const b = true      // untyped boolean constant
const s = "Hello"   // untyped string constant
```

You may be confused that  I'm using terms like integer constant, and I'm also saying that it is untyped.

Well yes, the value `1` is an integer, `4.5` is a float, and `"Hello"` is a string. But they are just values. They are not given a **fixed** type yet, like `int32` or `float64` or `string`, that would force them to obey Go’s strict type rules.

The fact that the value `1` is untyped allows us to assign it to any variable whose type is **compatible** with integers（string and boolean are not included)

```
var myInt int = 1
var myFloat float64 = 1
var myComplex complex64 = 1
```

### 1.3 Constants and Type inference：Default Type

Go supports type inference. That is, it can infer the type of a variable from the value that is used to initialize it. So you can declare a variable with an initial value, but without any type information, and Go will automatically determine the type

```go
var a = 5  // Go compiler automatically infers the type of the variable `a`
```

But how does it work? Given that constants in Golang are untyped, what will be the type of the variable `a` in the above example? Will it be `int8` or `int16` or `int32` or `int64` or `int`?

Well, it turns out that every untyped constant in Golang has a **default type**. The default type is used when we assign the constant to a variable that doesn’t have any explicit type available.

Following are the default types for various constants in Golang

```
integers (10, 76)           int
floats (3.14, 7.92)         float64
complex numbers (3+5i)      complex128
characters (`'a'`, `'♠'`)  rune
booleans (true, false)      bool
strings (“Hello”)           sting
```

So, in the statement `var a = 5`, since no explicit type information is available, the default type for integer constants is used to determine the type of `a`, which is `int`.

### 1.4 Typed Constants

In Golang, Constants are typed when you **explicitly** specify the type in the declaration like this

```go
const typedInt int = 1  // Typed constant
```

Just like variables, all the rules of Go’s type system applies to typed constant. For example, you cannot assign a typed integer constant to a float variable

```go
var myFloat64 float64 = typedInt  // Compiler Error
```

With typed constants, you lose all the flexibility that comes with untyped constants like assigning them to any variable of compatible type or mixing them in mathematical operations. So you should declare a type for a constant only if it’s absolutely necessary. Otherwise, just declare constants without a type.

### 1.5 Constant Expressions

The fact that constants are untyped (unless given a type explicitly) allows you to mix them in any expression freely.

So you can have a contant expression containing a mix of various untyped constants as long as those untyped constants are compatible with each other

```go
const a = 5 + 7.5 // Valid
const b = 12/5    // Valid
const c = 'z' + 1 // Valid

const d = "Hey" + true // Invalid (untyped string constant and untyped boolean constant are not compatible with each other)

```

The evaluation of constant expressions and their result follows certain rules. Let’s look at those rules

**Rules for constant expressions**

- A comparison operation between two untyped constants always outputs an untyped boolean constant.

  ```go
  const a = 7.5 > 5        // true (untyped boolean constant)
  const b = "xyz" < "uvw"  // false (untyped boolean constant)
  
  ```

- For any other operation (except shift)

  - If both the operands are of the same type (ex - both are untyped integer constants), the result is also of the same type. For example, the expression `25/2` yields `12` not `12.5`. Since both the operands are untyped integers, the result is truncated to an integer.

  - If the operands are of different type, the result is of the operand’s type that is broader as per the rule: `integer < rune < floating-point < complex`.

    ```go
    const a = 25/2      // 12 (untyped integer constant)
    const b = (6+8i)/2  // (3+4i) (untyped complex constant)
    
    ```

- Shift operation rules are a bit complex. First of all, there are some requirements

  - The right operand of a shift expression must either have an unsigned integer type or be an untyped constant that can represent a value of type `uint`.
  - The left operand must either have an integer type or be an untyped constant that can represent a value of type `int`

  **The rule -** If the left operand of a shift expression is an untyped constant, the result is an untyped integer constant; otherwise the result is of the same type as the left operand.

  ```go
  const a = 1 << 5          // 32 (untyped integer constant)
  const b = int32(1) << 4   // 16 (int32)
  
  const c = 16.0 >> 2       // 4 (untyped integer constant) - 16.0 can represent a value of type `int`
  const d = 32 >> 3.0       // 4 (untyped integer constant) - 3.0 can represent a value of type `uint`
  
  const e = 10.50 << 2      // ILLEGAL (10.50 can't represent a value of type `int`)
  const f = 64 >> -2        // ILLEGAL (The right operand must be an unsigned int or an untyped constant compatible with `uint`)
  
  ```


{{<admonition abstract>}}
在上述的英文内容中，说明了常量的两种类型 `Untyped Constants` 和 `Typed Constants`

Untyped Constants：实质上就是指字面量，即100，’a’，3.1415等。

Typed Constants：通过常量声明，将字面量和一个标识符绑定，同时指定特定的类型。

在Go中，对每种类型的字面量有一个预设的类型，将字面量与特定类型的标识符相绑定时，需要考虑兼容性。此外在常量表达式的计算中也需要考虑字面量之间是否可以相互兼容。
{{</admonition>}}


### 1.6 字面量

字面量，比如1，2.3，true，“Hello"，字面量也常称为未命名常量

```
integer constants        100, 67413
floating-point constants 4.56, 128.372
boolean constants        true, false
rune constants           'C', 'ä'
complex constants        2.7i, 3 + 5i
string constants         "Hello", "Rajeev"

```

### 1.7 常量声明格式

如果标识符后不加类型，会根据右边的值自动推导类型

```go
const pi = 3.1415926
const e = 2.7182
const str string = "abc"

// 或
const (
    pi float64 = 3.1415926
    e float64 = 2.7182
    str string = "abc"
)

// 支持并行赋值
const a, b, c = 1, 2, 3

```

**批量声明**：const同时声明多个常量时，如果省略了值，则表示与上一行相同

```go
const (
    n1 = 100
    n2
    n3
)

```

**注意事项：**

不可以用自定义的函数给常量赋值，但内置函数可以，例如`len()`

```go
const testConst = "test"
const c1 = len(testConst)

const c1 = len([]int{2, 3, 4}) // len(([]int literal)) (value of type int) is not constant
const c2 = getNumber()         // getNumber() (value of type int) is not constant
func getNumber() int {
    return 1
}

```

### 1.8 iota常量生成器

- `iota`是go语言的常量计数器，只能在常量的表达式中使用。
- `iota`在const关键字出现时将被重置为0。
- const中每新增一行常量声明将使`iota`计数一次(iota可理解为const语句块中的行索引)。 使用iota能简化定义，在定义枚举时很有用。

**常见`iota`示例:**

- 枚举

  ```go
  type Weekday int
  
  const (
      Sunday Weekday = iota
      Monday
      Tuesday
      Wednesday
      Thursday
      Friday
      Saturday
  )
  
  ```

- 使用`_`跳过某些值

  ```go
  const (
      n1 = iota //0
      n2        //1
      _
      n4        //3
  )
  
  
  ```

- 在`iota`中插队

  ```go
  const (
      n1 = iota //0
      n2 = 100  //100
      n3 = iota //2
      n4        //3
  )
  const n5 = iota //0
  
  ```

- 使用`iota`定义数量级

  ```go
  const (
      _ = 1 << (10 * iota)
      KiB // 1024                         2^10
      MiB // 1048576                      2^20
      GiB // 1073741824                   2^30
      TiB // 1099511627776                2^40
      PiB // 1125899906842624             2^50
      EiB // 1152921504606846976          2^60
      ZiB // 1180591620717411303424       2^70
      YiB // 1208925819614629174706176    2^80
  )
  
  // 注意KB和KiB的区别
  const (
      KB = 1000
      MB = 1000 * KB
      GB = 1000 * MB
      PB = 1000 * GB
  )
  
  ```

- 多个`iota`定义在一行

  ```go
  const (
      a, b = iota + 1, iota + 2 //1,2 iota = 1
      c, d                      //2,3 iota = 2
      e, f                      //3,4 iota = 3
  )
  
  ```

- 取消`iota`

  ```go
  // 赋值一个常量时，之后没赋值的常量都会应用上一行的赋值表达式
  const (
      a = iota  // a = 0
      b         // b = 1
      c         // c = 2
      d = 5     // d = 5
      e         // e = 5
  )
  
  ```

### 1.9 其他

以下代码中，常量的定义并没有一个明确的基础类型，此时，编译器会为这些没有明确基础类型的数字常量**提供比基础类型更高精度的算术运算**

- ~~反斜杠`\`可以在常量表达式中作为多行的连接符使用~~
- 数字型的常量无需担心类型转换问题，它们都是非常理想的数字
- 当常量赋值给一个精度过小的数字型变量时，可能会因为无法正确表示而导致溢出，会在编译期间引发错误

```go
const PI = 3.1415926
const Ln2 = 0.693147180559945309417232121458176568075500134360255254120680009
// 报错，可能是已经移除了这种语法
// const Ln2 = 0.693147180559945309417232121458\
//          176568075500134360255254120680009
const Log2E = 1/Ln2 // this is a precise reciprocal
const Billion = 1e9 // float constant
const hardEight = (1 << 100) >> 97

```

## 2 变量

**不同于PHP和Python, Golang是静态类型语言**

- 变量必须先声明后使用
- Go语言在声明变量的时候，会自动对变量对应的内存区域进行初始化操作。每个变量会被初始化成其类型的默认值，例如： 整型和浮点型变量的默认值为`0`。 字符串变量的默认值为`空字符串`。 布尔型变量默认为`false`。 切片、函数、指针变量的默认为`nil`。

### 2.1 变量作用域

- 一个变量（常量、类型或函数）在程序中都有一定的作用范围，称之为作用域。
- 如果一个变量在函数体外声明，则被认为是全局变量，可以在整个包（被导出后可以在外部包）使用。
- 在函数体内声明的变量称之为局部变量，它们的作用域只在函数体内，参数和返回值变量也是局部变量。一般情况下，局部变量的作用域可以通过代码块（用大括号括起来的部分）判断。
- 可以在某个代码块的内层代码块中使用相同名称的变量，则此时外部的同名变量将会暂时隐藏。

关于作用域的更多信息请参考如下文章[ch14 包与变量作用域](http://zchaoyu1126.github.io/go-ch14.html)

### 2.2 声明格式

需要注意的是，Go 和许多编程语言不同，它在声明变量时将变量的类型放在变量的名称之后。

首先，它是为了避免像 C 语言中那样含糊不清的声明形式，例如：`int* a, b;`。在这个例子中，只有 a 是指针而 b 不是。如果你想要这两个变量都是指针，则需要将它们分开书写。其次，这种语法能够按照从左至右的顺序阅读，使得代码更加容易理解。

**标准声明格式**

```go
var name string

```

**批量声明格式**

```go
var(
    name string  //""
    age int      //0
    isOk bool    //false
)
// 这种因式分解关键字的写法一般用于声明全局变量

```

### 2.3 初始化

**可在声明变量的时候为其指定初始值。变量初始化的标准格式如下：**

```go
var 变量名 类型 = 表达式

```

举个例子：

```go
var name string = "Q1mi"
var age int = 18

```

**一次初始化多个变量**

```go
var name,age = "QImi",20

```

### 2.4 类型推导

有时会将变量的类型省略，这个时候编译器会根据等号右边的值来推导变量的类型完成初始化

```go
// 变量类型在编译时实现自动推断
var name = "Qimi"
var age = 18

// 变量类型在运行时实现自动推断
var (
    HOME = os.Getenv("HOME")
    USER = os.Getenv("USER")
    GOROOT = os.Getenv("GOROOT")
)

```

### 2.5 短变量声明

> 这是使用变量的首选形式，但只能被用在函数体内，不可以用于全局变量的声明与赋值。
>
>
> 使用操作符 `:=` 可以高效地创建一个新的变量，称之为初始化声明。

**在函数内部**，可以使用更简略的 := 方式声明并初始化变量

```go
package main

import "fmt"

var m=100

func main(){
    m := 10   //局部变量
    n := 20
    fmt.Println(m,n)
}


```

注意：局部变量声明之后必须使用，单纯的赋值也是不够的，全局变量允许声明但不使用

### 2.6 匿名变量

在使用多重赋值时，如果想要忽略某个值，可以使用`匿名变量（anonymous variable）`。

匿名变量用一个下划线`_`表示，例如：

```go
func foo() (int, string) {
    return 10, "Q1mi"
}
func main() {
    x, _ := foo()
    _, y := foo()
    fmt.Println("x=", x)
    fmt.Println("y=", y)
}


```

匿名变量不占用命名空间，不会分配内存，所以匿名变量之间不存在重复声明。

(在`Lua`等编程语言里，匿名变量也被叫做哑元变量。)

### 2.7 init函数中的变量初始化

- 变量除了可以在全局声明中初始化，也可以在 init 函数中初始化。
- 这是一类非常特殊的函数，它不能够被人为调用，而是在每个包完成初始化后自动执行，并且执行优先级比 main 函数高。
- 每个源文件都只能包含一个 init 函数。初始化总是以单线程执行，并且按照包的依赖关系顺序执行。
- 一个可能的用途是在开始执行程序之前对数据进行检验或修复，保证程序状态的正确性。

```go
package trans

import "math"

var Pi float64

func init() {
    Pi = 4 * math.Atan(1) // init() function computes Pi
}
```

```go
package main

import (
   "fmt"
   _ "chao/trans"
)

var twoPi = 2 * trans.Pi

func main() {
   fmt.Printf("2*Pi = %g\n", twoPi) // 2*Pi = 6.283185307179586
}
```

init 函数也经常被用在当一个程序开始之前调用后台执行的 goroutine，如下面例子中的 `backend()`：

```go
func init() {
   // setup preparations
   go backend()
}
```


