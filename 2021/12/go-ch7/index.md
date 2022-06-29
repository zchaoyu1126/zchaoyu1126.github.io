# Ch7 输入与输出



在初学时，初步掌握命令行参数、使用`Scan`族函数读取`Stdin`，使用`Print`族函数向`Stdout`输出即可，关于输入输出更多的操作，请参考[ch28 进阶输入输出](http://zchaoyu1126.github.io/go-ch28.html)

## 1 命令行参数

`os`以跨平台的方式，提供了一些与操作系统交互的函数和变量，程序的命令行参数可以从os包中的Args变量获取。`os.Args`是一个字符串类型的切片`[]string`，其中`os.Args[0]`是文件名，其他元素是程序启动时传给它的参数。

```go
package main

import (
    "os"
    "fmt"
)

func main() {
    str := ""
    for _, v := range os.Args {
        str += " "
        str += v
    }
    fmt.Println(str)
}
```

## 2 Scan族函数

可使用`fmt`包提供的`Scan`开头的函数，从键盘和标准输入`os.Stdin`中读取输入

注意！`Scan类无法读取空格`，空格会被视为两个参数之间的分隔

- `Scan`: 读入，直到全部得到参数

```go
package main

import "fmt"

func main() {
    var a string
    var b int
    fmt.Scan(&a, &b)
    fmt.Println(a, b)
}
// input: hello world 直接结束了，不给输入int值的机会
// output: hello 0
```

- `Scanf`：必须严格按照此输入模式

	若格式化串内含有回车，则在键盘输入时，必须有相应的回车相对应。
	
	
	格式化串内的空格数目和输入中的空格数目没有影响。

```go
package main

import "fmt"

func main() {
    var a string
    var b int
    fmt.Scanf("%s \n %d", &a, &b)
    fmt.Println(a, b)
}
// input: hello
//        1
// output: hello 1

// input: hello 1
// output: hello 0
```

- `Scanln`扫描来自标准输入中一行的文本，将空格分隔的值依次存放到后续的参数内，直到碰到换行。

	如果扫描到的值小于参数数目，则直接结束，未被赋值的参数仍为原来的值（或默认值）。
	
	如果扫描到的值多于参数数目，会出现奇怪的问题，见下文代码。

```go
package main

import "fmt"

func main() {
    var a, b, c int
    fmt.Scanln(&a, &b, &c)
    fmt.Println(a, b, c)
}
// input: 1 2
// output: 1 2 0  2之后遇到回车不会继续等待输入，结束Scanln语句

// input: 1 2 3
// output: 1 2 3

// input: 1 2 3 4
// output: 1 2 3
```

```go
package main

import "fmt"

func main() {
    var a, b, c, d, e int
    fmt.Scanln(&a, &b, &c)
    fmt.Println(a, b, c)
    fmt.Scanln(&d, &e)
    fmt.Println(d, e)
}

// input: 1 2 3 4 5
// output: 1 2 3
//         5 0
// 出现此结果的原因，可能是Scanln将输入缓冲区中的4修改成为了换行符
// 因此，为了正确的输入，输入4，5时必须在换行符之后
```

## 3 Print族函数

可使用`fmt`包提供的`Print`开头的函数，可向键盘和标准输出`os.Stdout`中进行输出

- `Printf`，输出格式化的字符串
  
    常见占位符如下，
    
    `%b`、`%d`、`%o`、`%x(%X)`：二进制、十进制、八进制、十六进制表示
    
    `%c`：相应Unciode码点所表示的字符
    
    `%s`：字符串
    
    `%t`：布尔
    
    `%g`：浮点数
    
    `%p`：指针，十六进制表示
    
    `%T`：值的类型
    
    `%v`：值的默认格式

```go
bool:                    %t
int, int8 etc.:          %d
uint, uint8 etc.:        %d, %x if printed with %#v
float32, complex64, etc: %g
string:                  %s
chan:                    %p
pointer:                 %p
```

​		
具体示例如下：

```go
package main

import "fmt"

func main() {
    a := 20
    b := 3.14
    c := "hi, Bruce"
    d := true
    e := 3 + 4i
    f := 's'
    fmt.Printf("a:%v, %T, %d\n", a, a, a)
    fmt.Printf("b:%v, %T, %f\n", b, b, b)
    fmt.Printf("c:%v, %T, %s\n", c, c, c)
    fmt.Printf("d:%v, %T, %t\n", d, d, d)
    fmt.Printf("e:%v, %T, %g\n", e, e, e)
    fmt.Printf("f:%v, %T, %c\n", f, f, f)
}

// a:20, int, 20
// b:3.14, float64, 3.140000
// c:hi, Bruce, string, hi, Bruce
// d:true, bool, true
// e:(3+4i), complex128, (3+4i)
// f:115, int32, s
```

- `Print`等价于对每一个操作数都应用`%v`
- `Println` 应用`%v`输出到控制台并换行
