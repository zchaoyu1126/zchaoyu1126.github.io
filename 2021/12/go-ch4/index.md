# Ch4 基本数据类型


关于Go语言中的类型

{{<admonition quote>}}

Go语言中的类型可以是

- 基本类型，如：整型、浮点型、布尔、字符串等
- 复合类型，如：数组、结构体等
- 引用类型，如：指针、切片、字典、函数、通道等
- 接口类型，如：接口等

以上内容摘自 https://topgoer.cn/docs/gopl-zh/gopl-zh-1d2a096ino06l

{{</admonition>}}

{{<admonition quote>}}
类型可以是基本类型，如：int、float、bool、string；结构化的（复合的），如：struct、array、slice、map、channel；只描述类型的行为的，如：interface。

结构化的类型没有真正的值，它使用 nil 作为默认值（在 Objective-C 中是 nil，在 Java 中是 null，在 C 和 C++ 中是NULL或 0）。值得注意的是，Go 语言中不存在类型继承。
以上内容摘自 https://github.com/unknwon/the-way-to-go_ZH_CN/blob/master/eBook/04.2.md
{{</admonition>}}

{{<admonition quote>}}
其他相关的分类：https://gfw.go101.org/article/type-system-overview.html
{{</admonition>}}


关于类型的说法很多，个人认为需要记住的是如下内容

值类型，如：int、float、bool、string、array、struct

引用类型，如：ptr、slice、map、channel、func、interface

值类型与引用类型的区别：

- 值类型变量直接存储值，内存通常在栈中分配
- 引用类型变量存储一个地址（默认值为nil），内存通常在堆上分配

具体地，请参考[Go 内存管理机制]()


## 1 数字

### 1.1 整型

```go
uint8         无符号 8位整型 (0 到 255)
uint16        无符号 16位整型 (0 到 65535)
uint32        无符号 32位整型 (0 到 4294967295)
uint64        无符号 64位整型 (0 到 18446744073709551615)
int8          有符号 8位整型 (-128 到 127)
int16         有符号 16位整型 (-32768 到 32767)
int32         有符号 32位整型 (-2147483648 到 2147483647)
int64         有符号 64位整型 (-9223372036854775808 到 9223372036854775807)
uint          32位操作系统上就是uint32，64位操作系统上就是uint64
int           32位操作系统上就是int32，64位操作系统上就是int64
uintptr       无符号整型，用于存放一个指针
```

**注意事项：**

- 在使用int和uint类型时，不能假定它是32位或者64位的整型，而是考虑int和uint可能在不同平台上的差异。
- uintptr没有指定具体的bit大小，但是足够容纳指针，一般只有在底层编程时才需要，特别是Go语言和C语言函数库或操作系统接口相交互的地方。
- 获取对象的长度的内建`len()`函数返回的长度可以根据不同平台的字节长度进行变化。实际使用中，map或切片的元素数量等都可以用`int`表示。
- 但是在涉及到二进制传输，读写文件的结构描述时，为了保持文件的结构不会受到不同编译目标平台字节长度的影响，不要使用`int`和`uint`。

### 1.2 浮点数

- Go语言支持两种浮点型数：`float32`和`float64`。这两种浮点型数据格式遵循`IEEE 754`标准：`float32`浮点数的最大范围约为`3.4e38`，常量定义为`math.MaxFloat32`;
- `float64`浮点数的最大范围约为`1.8e308`，常量定义为`math.MaxFloat64`
- `float32`精确到小数点后7位，`float64`精确到小数点后15位。使用`==`或`!=`来比较浮点数时应当非常小心，在正式使用前测试对于精确度要求比较高的运算。

math包中除了提供大量常用的数学函数外，还提供了IEEE754浮点数标准中定义的特殊值的创建和测试

1. `+Inf`和`-Inf` 分别表示正无穷大和负无穷大
2. `NaN`非数，一般用于表示无效的除法操作结果0/0或Sqrt(-1).

```go
var z float64
fmt.Println(z, -z, 1/z, -1/z, z/z) // "0 -0 +Inf -Inf NaN"
```

函数`math.IsNaN`用于测试一个数是否是非数NaN，math.NaN则返回非数对应的值。

虽然可以用math.NaN来表示一个非法的结果，但是测试一个结果是否是非数NaN则是充满风险的，因为NaN和任何数都是不相等的（在浮点数中，NaN、正无穷大和负无穷大都不是唯一的，每个都有非常多种的bit模式表示）：

```go
nan := math.NaN()
fmt.Println(nan == nan, nan < nan, nan > nan) // "false false false"
```

如果一个函数返回的浮点数结果可能失败，最好的做法是用单独的标志报告失败，像这样：

```go
func compute() (value float64, ok bool) {
    // ...
    if failed {
        return 0, false
    }
    return result, true
}
```

### 1.3 复数

`complex64`和`complex128`

复数有实部和虚部，`complex64`的实部和虚部为`float32`，`complex128`的实部和虚部为`float64`

- 使用字面量创建复数

```go
var c1 complex64
c1 = 1 + 2i
var c2 complex128
c2 = 2 + 3i
// 或者
// c1 := 1 + 2i
// c2 := 2 + 3i
```

- 使用`complex`函数创建复数

如果实数部分和虚数部分的类型均为float32，那么类型为`complex64`的复数c可以通过以下方式来获得

```go
c := complex(re, im)
fmt.Println(real(c))
fmt.Println(imag(c))
// 函数`real(c)`和`imag(c)`可以分别获得相应的实数和虚数部分。
```

- `+`、``、 ``、 `/` 运算符支持实数与实数、实数与复数、复数与复数的运算，但`%`不能用于复数
- 复数支持和其他数字类型一样的运算，当使用等号`==`或不等号`!=`对复数进行比较运算时，注意对精确度的把握。
- `math/cmplx`包中包含了一些操作复数的公共方法，接收的类型都是`complex128`。

### 1.4 数字字面量语法

Go1.13版本之后引入了数字字面量语法，这样便于开发者以二进制、八进制或十六进制浮点数的格式定义数字，例如：

```go
num1 := 0b00101101      // 代表二进制的 101101，相当于十进制的 45
num2 := 0o366           // 代表八进制的 377，相当于十进制的 255
num3 := 0x1p-2          // 代表十六进制的 1 除以 2²，也就是 0.25
num4 := 123_456         // 允许用 _ 来分隔数字，比如说：v := 123_456 等于 123456。
```

```go
const e = 2.71828 // (approximately)
```

很小或很大的数最好用科学计数法书写，通过e或E来指定指数部分：

```go
const Avogadro = 6.02214129e23  // 阿伏伽德罗常数
const Planck   = 6.62606957e-34 // 普朗克常数
```

注意事项

- num3 := 1p-2 会报错 'p' exponent requires hexadecimal mantissa，这种方式只能用于十六级进制
- 八进制数据通常用于POSIX操作系统上的文件访问权限标志，十六进制数字则更强调数字值的bit位模式。
- 浮点数的字面值，其中小数点前面或后面的数字都可能被省略（例如.707或1.）

## 2 字符与字符串

### 2.1 byte、rune

Go 语言的字符有以下两种：

1. `uint8`类型，或者叫 byte 型，代表了`ASCII码`的一个字符。
2. `rune`类型，代表一个 `Unicode码点`。

- 当需要处理中文、日文或者其他复合字符时，则需要用到`rune`类型。
- Unicode字符(rune类型)是和int32等价的类型，通常用于表示一个Unicode码点。
- byte是uint8类型的等价类型，byte类型一般用于强调数值是一个原始的数据而不是一个小的整数。

{{<admonition quote>}}
关于字符编码unicode utf-8等相关知识，可参考http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html
{{</admonition>}}

### 2.2 byte、rune的字面量
{{<admonition tip>}}
关于\x41、\101，\u03B2，\U00101234的用法会在下文字符串转义部分进行解释
{{</admonition>}}

**byte**

在ASCII码表中，A的值是65，而使用16进制表示则为41，所以下面的写法是等效的

```
var ch byte = 65
var ch byte = '\x41'  // \x 后跟长度为2的16进制数
var ch byte = '\101'  // \  后跟长度为3的8进制数
```

**rune**

- 在书写Unicode字符时，需要在16进制数之前加上前缀`\u`或者`\U`
- 前缀 `\u` 总是紧跟着长度为 4 的 16 进制数，前缀 `\U` 紧跟着长度为 8 的 16 进制数。

```go
var ch int = '\u0041'
var ch2 int = '\u03B2'
var ch3 int = '\U00101234'
fmt.Printf("%d - %d - %d\n", ch, ch2, ch3) // integer
fmt.Printf("%c - %c - %c\n", ch, ch2, ch3) // character
fmt.Printf("%X - %X - %X\n", ch, ch2, ch3) // UTF-8 bytes
fmt.Printf("%U - %U - %U", ch, ch2, ch3) // UTF-8 code point
//65 - 946 - 1053236
//A - β - r
//41 - 3B2 - 101234
//U+0041 - U+03B2 - U+101234
```

- `%d`：十进制表示
- `%c`：相应Unicode码点所表示的字符
- `%X`：十六进制表示，字母位大写形式
- `%U`：Unicode格式：U+1234

格式占位符将在[ch7 基本输入输出](http://zchaoyu1126.github.io/go-ch7.html)中进一步进行说明

包 `unicode` 包含了一些针对测试字符的非常有用的函数（其中 `ch` 代表字符）：

- 判断是否为字母：`unicode.IsLetter(ch)`
- 判断是否为数字：`unicode.IsDigit(ch)`
- 判断是否为空白符号：`unicode.IsSpace(ch)`

包 `utf8` 拥有更多与 rune 类型相关的函数。

### 2.3 字符串

**字符串的底层结构**

Go语言里的字符串的内部实现使用`utf-8`编码。

字符串底层的是一个byte数组+字符串长度len。"hi，我是小明"这一个字符串，将把每个`Unicode`码点使用UTF-8进行统一编码，从而得到一个字节数组。

Go编译器认为字符串通常是不可修改的，故会将字符串存储在常量区，因此尝试修改字符串内部数据的操作也是被禁止的：

```go
str := "hi, 我是小明"
str[0] = 'H'            // compile error: cannot assign to s[0]
```

不变性意味着如果两个字符串共享相同的底层数据的话也是安全的，这使得复制任何长度的字符串代价是低廉的。

同样，一个字符串s和对应的子字符串切片的操作也可以安全地共享相同的内存，因此字符串切片操作代价也是低廉的。

在这两种情况下都没有必要分配新的内存。

```go
str := "helloworld"
substr1 := str[0:5]
substr2 := str[5:]
fmt.Println(str, substr1, substr2)
str = "hiworld"
fmt.Println(str, substr1, substr2)

// helloworld hello world
// hiworld hello world
```

**字符串的遍历**

```go
// 遍历字符串
func traversalString() {
    s := "hello沙河"
    for i := 0; i < len(s); i++ { //byte
        fmt.Printf("%v(%c) ", s[i], s[i])
    }
    fmt.Println()
    for _, r := range s { //rune
        fmt.Printf("%v(%c) ", r, r)
    }
    fmt.Println()
}
```

输出：

```
104(h) 101(e) 108(l) 108(l) 111(o) 230(æ) 178(²) 153() 230(æ) 178(²) 179(³)
104(h) 101(e) 108(l) 108(l) 111(o) 27801(沙) 27827(河)
```

正如上述所说，字符串底层是一个byte数组，len(str)得到的结果是byte数组的长度，而UTF8编码下一个中文汉字（通常）由3个字节组成，所以不能简单的按照字节去遍历一个包含中文的字符串，否则就会出现上面输出中第一行的结果。此时，可以采用`for range`方式来遍历字符串，`for range`方式是遍历字符串中的`rune`，即`Unicode`码点

### 2.4 字符串转义符

```go
\r     回车符（返回至行首)
\n     换行符（直接跳到下一行的同列位置）
\t     制表符
\'     单引号
\"     双引号
\\     反斜杠
```

除了上述的字符外，还有很多Unicode字符很难直接从键盘输入，并且还有很多字符有着相似的结构（例如在中文和日文中就有很多相似但不同的字），有一些甚至是不可见的字符。为了解决这个问题，Go语言字符串字面量中的`Unicode转义字符`允许可以通过Unicode码点输入特殊的字符。

有两种方法：

- 方法1：使用UTF-8后的编码值（`\x`）。将Unicode码点值使用UTF-8编码后得到`hhhhhh`，使用`\xhh\xhh\xhh`序列表示编码`hhhhhh`，以此来插入该Unicode码点。
- 方法2：使用Unicode码点值（`\u`或`\U`）。`\uhhhh`（16bit的码点值）或`\Uhhhhhhhh`（32bit的码点值）。

例如：下面的字母串面值都表示相同的值：

```
"世界"                         // 世界的Unicode  U+4E16 U+754C
"\xe4\xb8\x96\xe7\x95\x8c"    // 将 U+4E16 U+754C转化成UTF-8编码之后的值了
"\u4e16\u754c"
"\U00004e16\U0000754c"
```

无论是方法一还是方法二，其实向底层的[]byte数组中填入的都是同样的数据，即方法一中的数值，`0xe4`,`0xb8`,`0x96`,`0xe7`,`0x95`,`0x8c`。 当需要读取数据时，会对比特流进行解码，从而获取数据。

### 2.5 rune与转义符
Unicode转义也可以使用在表示`rune`字符中。下面三个字符是等价的：

```
'世' '\u4e16' '\U00004e16'
```

此外，对于码点值小于等于`0x7F`的码点，可以使用例如`\x41`（对应字符A）的形式表示对应的码点，但是对于更大的码点，例如`'世'`则必须使用`\u`或`\U`转义形式，而不是`ch = '\xe4\xb8\x96'`。

原因：
- 能使用`\x41`表示字符'A'是因为`0x41`不仅是字符'A'的Unicode码点，也是其经过UTF-8后的编码值。
- `'\xe4\xb8\x96'` 会经历三次转义，每个`\x`分别代表了一个rune字符，且均为不合法字符。


```go
import "fmt"

int main() {
    s := "\u0041\u00FF\x41\xC3\xBF\xFF"
    fmt.Println(s)
}
// AÿAÿ�
```

关于�的说明如下：

每一个UTF8字符解码，不管是显式地调用utf8.DecodeRuneInString解码或是在range循环中隐式地解码，如果遇到一个错误的UTF8编码输入，将生成一个特别的Unicode字符`\uFFFD`，在印刷中这个符号通常是一个黑色六角或钻石形状，里面包含一个白色的问号”?”。当程序遇到这样的一个字符，通常是一个危险信号，说明输入并不是一个完美没有错误的UTF8字符串。



### 2.6 原生字符串

一个原生的字符串面值形式是使用反引号代替双引号。

- 在原生的字符串面值中，没有转义操作；全部的内容都是字面的意思，包含退格和换行，因此一个程序中的原生字符串面值可能跨越多行。
- 在原生字符串面值内部是无法直接写`字符的，可以用八进制或十六进制转义或用+**"`"**连接字符串常量完成
- 唯一的特殊处理是会删除回车以保证在所有平台上的值都是一样的，包括那些把回车也放入文本文件的系统（译注：Windows系统会把回车和换行一起放入文本文件中）。
- 原生字符串面值用于编写正则表达式会很方便，因为正则表达式往往会包含很多反斜杠。原生字符串面值同时被广泛应用于HTML模板、JSON面值、命令行提示信息以及那些需要扩展到多行的场景。

```go
const GoUsage = `Go is a tool for managing Go source code.

Usage:
    go command [arguments]
...`
```

## 3 布尔

Go语言中以`bool`类型进行声明布尔型数据，布尔型数据只有`true`和`flase`两个值

注意：

- 布尔类型变量的默认值`false`
- Go语言中不允许将整型强制转换为布尔型
- 布尔型无法参与数值运算，也无法与其他类型进行转换

## 4 类型转换

**Go语言中只有强制类型转换，没有隐式类型转换**。该语法只能在两个类型之间支持相互转换的时候使用。

强制类型转换的基本语法为：T(表达式)，其中，T表示要转换的类型。表达式包括变量、复杂算子和函数返回值等.

比如，计算直角三角形的斜边长时使用math包的Sqrt()函数，该函数接收的是float64类型的参数，而变量a和b都是int类型的，这个时候就需要将a和b强制类型转换为float64类型。

```go
func sqrtDemo() {
    var a, b = 3, 4
    var c int
    // math.Sqrt()接收的参数是float64类型，需要强制转换
    c = int(math.Sqrt(float64(a*a + b*b)))
    fmt.Println(c)
}
```

对于每种类型T，如果转换允许的话，类型转换操作T(x)将x转换为T类型。许多整数之间的相互转换并不会改变数值；它们只是告诉编译器如何解释这个值。

但是对于将一个大尺寸的整数类型转为一个小尺寸的整数类型，或者是将一个浮点数转为整数，可能会改变数值或丢失精度：

```go
f := 3.141 // a float64
i := int(f)
fmt.Println(f, i) // "3.141 3"
f = 1.99
fmt.Println(int(f)) // "1"
```

浮点数到整数的转换将丢失任何小数部分，然后向数轴零方向截断。

应该避免对可能会超出目标类型表示范围的数值做类型转换，因为截断的行为可能依赖于具体的实现：

```go
f := 1e100  // a float64
i := int(f) // 结果依赖于具体实现
```

## 5 值传递

基本类型是Go语言自带的类型，比如数值类型，浮点型，字符类型和布尔类型，它们本质上是原始类型，也就是不可改变，所以对它们进行操作，一般都会返回一个新创建的值。所以把这些值传递给函数时，其实传递的是一个值的副本。

```go
func main(){
    name:="张三"
    fmt.Println(modift(name))
    fmt.Println(name)
}

func modify(s string) string{
    s=s+s
    return s
}
```

基本类型因为是拷贝的值，并且在对它进行操作的时候，生成的也是新创建的值，所以这些类型在多线程里是安全的，不用担心一个线程的修改影响了另外一个线程的数据。
