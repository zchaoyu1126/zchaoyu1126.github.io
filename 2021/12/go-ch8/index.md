# Ch8 控制结构



## 1 条件结构和分支结构

### 1.1 if-else

**习惯用法一**

  ```go
var prompt = "Enter a digit, e.g. 3 "+ "or %s to quit."

func init() {
	if runtime.GOOS == "windows" {
		prompt = fmt.Sprintf(prompt, "Ctrl+Z, Enter")
	} else { //Unix-like
		prompt = fmt.Sprintf(prompt, "Ctrl+D")
	}
}
  ```



**习惯用法二  ok-pattern模式**

if语句可包含一个初始化语句，例如给一个变量赋值

```go
if val := 10; val > max {
	// do something
}
```

使用简短方式`:=`声明的变量`val`，其作用域只存在于if语句中，包括if后的else语句。

如果变量`val`在if结构之前已经存在，那么在if语句块中，该变量原来的值会被隐藏。

```go
if value := process(data); value > max {
	// do something
}
```

```go
value, err := pack1.Function1(param1)
if err != nil {
	fmt.Printf("An error occured in pack1.Function1 with parameter %v", param1)
	return err
}
```

使用习惯语法二，可将上述代码简化为，这种方式常称为`ok-pattern`模式

```go
if value, err := pack1.Function1(param1); err != nil {
	fmt.Printf("An error occured in pack1.Function1 with parameter %v", param1)
	return err
}
```



### 1.2 switch结构

```go
switch var1 {
    case value1:
    ...
    case value2:
    ...
    default:
    ...
}
```

- `value1`和`value2`必须是相同类型的值或计算结果为相同类型的表达式
- 与C语言一样，`case`后可跟多个可能符合条件的值，使用逗号分割
- Golang使用快速的查找算法来测试switch条件与case分支的匹配清空，直到算法匹配到某个case或进入default为止
- 某个分支的代码执行完毕后，会退出整个switch代码块。如果希望执行后续分支的代码，可以使用`fallthrough`关键字
- 在 `case ...:` 语句之后，不需要使用花括号将多行语句括起来，当代码块只有一行时，可以直接放置在 `case` 语句之后。

**习惯用法一**

```go
package main

import "fmt"

func main() {
    var num1 int = 100

    switch num1 {
    case 98, 99:
        fmt.Println("It's equal to 98")
    case 100:
        fmt.Println("It's equal to 100")
    default:
        fmt.Println("It's not equal to 98 or 100")
    }
}
```

**习惯用法二**

switch 语句的第二种形式是不提供任何被判断的值（实际上默认为判断是否为 true），然后在每个 case 分支中进行测试不同的条件。当任一分支的测试结果为 true 时，该分支的代码会被执行。这看起来非常像链式的 `if-else` 语句，但是在测试条件非常多的情况下，提供了可读性更好的书写方式。

```go
package main

import "fmt"

func main() {
    var num1 int = 7

    switch {
    case num1 < 0:
        fmt.Println("Number is negative")
    case num1 > 0 && num1 < 10:
        fmt.Println("Number is between 0 and 10")
    default:
        fmt.Println("Number is 10 or greater")
    }
}
```

**习惯用法三**

switch 语句的第三种形式是包含一个初始化语句，这种形式可以非常优雅地进行条件判断。在下面这个代码片段中，变量 a 和 b 被平行初始化，然后作为判断条件：

```go
switch a, b := x[i], y[j]; {
    case a < b: t = -1
    case a == b: t = 0
    case a > b: t = 1
}
```

此外，如果初始化语句出现赋值失败的情况，例如如下的语句，这种情况需要自己额外注意会不会引入意向不到的BUG。

```go
package main

import "fmt"

var scores = map[string]int{"Tony": 95, "Tom": 59, "Sarah": 79}

func main() {
    switch score := scores["Lisa"]; {
    case score > 90:
        fmt.Println(score)
        fmt.Println("excellent")
    case score > 80:
        fmt.Println("Pretty good")
    case score > 60:
        fmt.Println("not bad")
    default:
        fmt.Println("failed")
    }
}
// Lisa doesn't exist, but this program still output failed otherwise 'unknown student'.
```

## 2 迭代或循环结构

### 2.1 for

**习惯用法一 变量i**

```go
package main

import "fmt"

func main() {
    for i := 0; i < 5; i++ {
        fmt.Printf("This is the %d iteration\n", i)
    }
}
```

{{<admonition warning>}}

特别注意, 永远不要在循环体内修改计数器，这在任何语言中都是非常差的实践！

{{</admonition>}}

得益于 Go 语言具有的平行赋值的特性,==可以在循环中同时使用多个计数器：==

```go
for i, j := 0, N; i < j; i, j = i+1, j-1 {
    // do something
}
```

**习惯用法二 基于条件判断的迭代**

for 结构的第二种形式是没有头部的条件判断迭代（类似其它语言中的 while 循环），基本形式为：`for 条件语句 {}`。

```go
package main

import "fmt"

func main() {
    var i int = 5

    for i >= 0 {
        i = i - 1
        fmt.Printf("The variable i is now: %d\n", i)
    }
}
```

**习惯用法三 无限循环**

条件语句是可以被省略的，如 `i:=0; ; i++` 或 `for { }` 或 `for ;; { }`（`;;` 会在使用 `gofmt` 时被移除）：这些循环的本质就是无限循环。最后一个形式也可以被改写为 `for true { }`，但一般情况下都会直接写 `for { }`。

如果 for 循环的头部没有条件语句，那么就会认为条件永远为 true，因此循环体内必须有相关的条件判断以确保会在某个时刻退出循环。想要直接退出循环体，可以使用 break 语句或 return 语句直接返回。

无限循环的经典应用是服务器，用于不断等待和接受新的请求。

### 2.2 for range

这是 Go的一种迭代结构，它可以迭代任何一个集合（包括数组和 map等）。

一般形式为：`for ix, val := range coll { }`。
{{<admonition warning>}}

要注意的是，`val` 始终为集合中**对应索引的值拷贝**，因此它一般只具有只读性质，对它所做的任何修改都不会影响到集合中原有的值，但**如果 val 为指针，则会产生指针的拷贝，依旧可以修改集合中的原值**。

{{</admonition>}}

```go
package main

import "fmt"

func main() {

    arr := []int{0, 1, 2, 3}
    for i, val := range arr {
        val = val * i
        fmt.Printf("in the loop, %dth element of arr is %d\n", i, val)
    }
    for i, val := range arr {
        fmt.Printf("after modify operation, %dth element of arr is %d\n", i, val)
    }

    matrix := [][]int{{1, 2, 3}, {4, 5, 6}}
    for _, row := range matrix {
        row[0] += row[0]
    }
    fmt.Println(matrix)
}

//in the loop, 0th element of arr is 0
//in the loop, 1th element of arr is 1
//in the loop, 2th element of arr is 4
//in the loop, 3th element of arr is 9
//after modify operation, 0th element of arr is 0
//after modify operation, 1th element of arr is 1
//after modify operation, 2th element of arr is 2
//after modify operation, 3th element of arr is 3
//[[1 2 3] [16 5 6]]
```

### 2.3 break、continue

- break：作用范围为该语句最后出现的内部结构，如for、switch、select
- continue： 忽略剩余的循环体，直接进行下一次是否进入循环的判断，只用于for

### 2.4 label、goto

正常continue情况下，continue只忽视内层循环，还应输出`i is: 0, and j is : 5`的结果，但这里直接使 i 变成下一个循环的值，而此时 j 的值就被重设为 0。

```go
func main() {
LABEL1:
    for i := 0; i <= 5; i++ {
        for j := 0; j <= 5; j++ {
            if j == 4 {
                continue LABEL1
            }
            fmt.Printf("i is: %d, and j is: %d\n", i, j)
        }
    }
}
//i is: 0, and j is: 0
//i is: 0, and j is: 1
//i is: 0, and j is: 2
//i is: 0, and j is: 3
//i is: 1, and j is: 0
//i is: 1, and j is: 1
//i is: 1, and j is: 2
//i is: 1, and j is: 3
//..
```

将 continue 改为 break，则不会只退出内层循环，而是直接退出外层循环。

```go
func main() {
LABEL1:
    for i := 0; i <= 5; i++ {
        for j := 0; j <= 5; j++ {
            if j == 4 {
                break LABEL1
            }
            fmt.Printf("i is: %d, and j is: %d\n", i, j)
        }
    }
}
// i is: 0, and j is: 0
// i is: 0, and j is: 1
// i is: 0, and j is: 2
// i is: 0, and j is: 3
```

- 标签和goto语句是不被鼓励的，只建议使用 goto 语句来跳出无限读取循环并关闭相应的客户端链接。
- 如果必须使用 goto，应当只使用正序的标签（标签位于 goto 语句之后），但注意标签和 goto 语句之间不能出现定义新变量的语句，否则会导致编译失败。

```go
// compile error goto2.go:8: goto TARGET jumps over declaration of b at goto2.go:8
package main

import "fmt"

func main() {
    a := 1
    goto TARGET // compile error
    b := 9
    TARGET:
    b += a
    fmt.Printf("a is %v *** b is %v", a, b)
}
```
