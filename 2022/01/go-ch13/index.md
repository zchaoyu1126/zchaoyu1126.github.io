# Ch13 函数


# ch13 函数

- Go语言中的函数没有默认参数。
- Go不支持函数重载特性，主要原因是函数重载需要进行多余的类型匹配影响性能。
  
    Go从1.18开始支持泛型，具体地参见
    
    [ch32 泛型](ch32%20%E6%B3%9B%E5%9E%8B%207b5640325501426d947381765da6001b.md)
    
- 从广义上说，Go语言中有三种类型的函数，分别是普通的带有名字的函数、匿名函数或lambda函数、与结构体相关的方法。
  
    可以把方法类比为C++中的成员函数。
    

## **命名规范**

返回某个对象的函数或者方法，一般都是用名词，不用`Get...`。如果用于修改某个对象，则使用`SetName`。一般使用驼峰命名法，不用下划线分割多个名称。

## **函数声明**

```go
// 代码1、非命名返回值
func functionName1(param type) type {
    ...
    return ret
}

// 代码2、命名返回值
func functionName2(param1, param2 type) (ret type) {
    ...
    return
}

// 3、多值返回，可变参数
func functionName3(params ...type) (int, error) {
    ...
    return cnt, nil
}
```

### **支持多值返回**

Go语言支持函数多值返回，也就是说定义的函数可以返回多个值，常见的有返回`error`类型，以供判断是否执行成功。

```go
func main(){
    file, err :=os.Open("/usr/tmp")
    if err != nil{
        log.Fatal(err)
        return
    }
    fmt.Println(file)
}
```

### **命名返回值和非命名返回值**

在声明函数时，若未明确返回值名称，则称之位为未命名返回值，如上述代码`1`；相对应地在声明时，明确了返回值名称，则称之为命名返回值，如上述代码`2`。

```go
package main

import (
    "fmt"
)

var num1 int = 10
var num2, num3 int

func main() {
    num2, num3 = resultWithDefinedRet(num1)
    fmt.Println(num2, num3)
    num2, num3 = resultWithUndefinedRet(num1)
    fmt.Println(num2, num3)
}

func resultWithDefinedRet(num int) (x, y int) {
    x, y = num*2, num*3
    return
}

func resultWithUndefinedRet(num int) (int, int) {
    return num*2, num*3
}
```

- 尽量使用命名返回值，会使代码更清晰、更简短，同时更加容易读懂
- 两种方式看似简单没有什么区别，但当`defer`和`return`结合时，就会出现很多问题，具体内容将在引入`defer`介绍。
  
    [ch16 defer](ch16%20defer%20b4ec22b09ec34eeaab5b6de5d7068bea.md)
    

### **支持可变参数**

可变参数的定义，在类型前面加上省略号`...`即可

```go
package main

import (
    "fmt"
)

func main() {
    res1 := sum(1, 2, 3)
    res2 := sum(7, 9, 1)
    fmt.Println(res1, res2)
}

func sum(arr ...int) (ret int) {
    for _, num := range arr {
        ret += num
    }
    return
}
```

- 固定参数和可变参数同时出现，可变参数放在最后
- 可变参数本质上是一个数组，所以可以像使用数组一样使用它，比如例子中的for range循环。
- 可通过`...`将`slice`中的数据转变为函数中需要的可变参数

```go
package main

import (
    "fmt"
)

func main() {
    strs := []string{"hi, ", "Joe.", "hi, ", "Anna."}
    output1(strs...)
    output2(strs)
}

func output1(strs ...string) {
    for _, str := range strs {
        fmt.Print(str)
    }
    fmt.Printf("\noutput1:%T\n", strs)
}

func output2(strs []string) {
    for _, str := range strs {
        fmt.Print(str)
    }
    fmt.Printf("\noutput2:%T\n", strs)
}
// hi, Joe.hi, Anna.
// output1:[]string
// hi, Joe.hi, Anna.
// output2:[]string
```

- 如果变长参数的类型没有指定，可以使用空接口`interface{}`,结合反射与switch结构确定参数的类型
  
    [ch19 接口](ch19%20%E6%8E%A5%E5%8F%A3%20d0c5e5fb1a054e34a7404085ea76be7d.md)
    
    [ch24 反射](ch24%20%E5%8F%8D%E5%B0%84%20d71899dbfd8b478785fa0f1270e3ba7b.md)
    

## **特殊函数**

- 一个保重除了main()、init()函数外，其他所有类型的函数都可以有参数和返回值。
- init 函数，一个包的初始化函数，在引入一个包后，最先执行的就是`init`函数，被不同源文件多次引入时，也只会执行一次。
- main 函数是每一个可执行程序所必须包含的，一般来说都是在启动后第一个执行的函数（如果有 `init`函数则会先执行该函数）。
- main 函数既没有参数，也没有返回类型。

## **内置函数**

- close函数：主要用来关闭channel
- len函数：len用于返回某个类型的长度或数量（字符串、数组、切片、map和管道）
- cap函数：cap是容量的意思，用于返回某个类型的最大容量（只能用于切片和map)
- new，分配内存，分配值类型，比如int, struct，返回的是指针
- make，分配内存，主要用来分配引用类型，比如chan,map,slice
- pannic和recover，用来做错误处理
- copy、append：用于复制和连接切片
- print、println：底层打印函数，在部署环境中建议使用fmt包
- complex、real、imag：用于创建和操作复数

## **注意事项**

- 函数值之间不能相互比较（nil例外）
- 函数不能在其它函数里面声明（不能嵌套），不过可以通过使用匿名函数来破除这个限制
- 可以返回其他函数的函数和接收其他函数作为参数的函数均被称为高阶函数，这是函数式语言的特点。显然，Go语言具有一些函数式语言的特性。
- 目前 Go 没有泛型（generic）的概念，也就是说它不支持那种支持多种类型的函数。不过在大部分情况下可以通过接口（interface），特别是**空接口与类型选择**与或者通过使用**反射**来实现相似的功能。使用这些技术将导致代码更为复杂、性能更为低下，所以在非常注意性能的的场合，最好是为每一个类型单独创建一个函数，而且代码可读性更强。
  
    Go从1.18开始支持泛型，具体参见
    

## 匿名函数

[ch15 匿名函数与闭包](ch15%20%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0%E4%B8%8E%E9%97%AD%E5%8C%85%2048dbb81e124e4c4d9b19bf169f4c96b6.md)

## 方法

[ch18 自定义类型与结构体](ch18%20%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B%E4%B8%8E%E7%BB%93%E6%9E%84%E4%BD%93%2076cc67414a6f417592c955fab74ebf63.md)
