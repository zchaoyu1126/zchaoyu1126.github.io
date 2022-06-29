# Ch9 指针


与C++不同的是，Go语言为程序员提供了控制数据结构指针的能力，但却不支持指针运算。

通过给予程序员基本内存布局，Go 语言允许控制特定集合的数据结构、分配的数量以及内存访问模式，这些对构建运行良好的系统是非常重要的。

## 1 指针的含义

声明变量的过程就是向系统申请内存的过程。如果在程序中声明了一个变量，那么操作系统会分配一块内存，此后就可以通过变量名在这块内存中存储、修改值。

该内存在操作系统中是有地址的，可以通过取地址符`&`来获得这个地址，把这个地址信息存在另一个变量中，这个变量就叫做指针。

- 内存地址通常用十六进制表示，32位机器4个字节，64位机器8个字节
- 当一个指针被定义后没有分配到任何变量时，它的值位`nil`
- `*ptr`，可获得`ptr`指向地址上所存储的值，这被称为反引用或解引用（dereference）

```go
package main

import "fmt"

func main() {
    var num = 5
    fmt.Printf("An integer: %d, its location in memory: %p\n", num, &num)

    var numPtr *int
    numPtr = &i1
    fmt.Printf("The value at memory location %p is %d\n", numPtr, *numPtr)
}

//An integer: 5, its location in memory: 0x24f0820
//The value at memory location 0x24f0820 is 5
```

下面的程序，通过指针修改了原字符串`s`

```go
package main

import "fmt"

func main() {
    s := "good bye"
    var p *string = &s
    *p = "ciao"
    fmt.Printf("Here is the pointer p: %p\n", p) // prints address
    fmt.Printf("Here is the string *p: %s\n", *p) // prints string
    fmt.Printf("Here is the string s: %s\n", s) // prints same string
}

//Here is the pointer p: 0x2540820
//Here is the string *p: ciao
//Here is the string s: ciao
```

在本节开头提到`不支持指针运算`指的是`p++,p--`这种,这些运算是不被允许的。

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {1, 2, 3, 4 ,5};
    for(int *p = arr; p < arr+5; p++) {
        cout << *p << endl;
    }
    return 0;
}
```

**注意事项**
{{<admonition warning>}}
不能获取常量（字面量也是常量）的地址。
{{</admonition>}}
```go
const i = 5
ptr := &i //error: cannot take the address of i
ptr2 := &10 //error: cannot take the address of 10
```
{{<admonition warning>}}
对空指针的解引用不合法，会使程序崩溃。
{{</admonition>}}

```go
package main

func main() {
    var p *int = nil
    *p = 0
}
// in Windows: stops only with: <exit code="-1073741819" msg="process crashed"/>
// runtime error: invalid memory address or nil pointer dereference
```

## 2 指针的利弊

**好处**：传参时，可以传一个指针，而不是变量的拷贝，可以减少内存占用和提高效率。

**坏处**：指针的过度频繁使用也可能导致性能下降。指针也可以指向另一个指针，并且可以进行任意深度的嵌套，导致可以有多级的间接引用，但在大多数情况这会使代码结构不清晰。

## 3 语法糖

> 语法糖（Syntactic sugar），也译为糖衣语法，是由英国计算机科学家彼得·约翰·兰达（Peter J. Landin）发明的一个术语，指计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用。通常来说使用语法糖能够增加程序的可读性，从而减少程序代码出错的机会。
> 

在大多数情况下 Go 语言可以使程序员轻松创建指针，并且隐藏间接引用，即自动解引用。在之后的结构体之中体现的更加明显，这是Go语言支持的一个语法糖。

stu是一个student结构体的指针，要调用成员变量，应为(*stu).age，但Go语言内部支持stu.age这种方式，能支持这种方式调用的本质原因是Go不支持对指针的修改，具体会在结构一章再次进行说明。

[ch18 自定义类型与结构体](http://zchaoyu1126.github.io)

```go
packge main

import "fmt"

type Student struct {
    Age  int
    Name string
}

func main() {
    stu := &Student{19, "Tony"}
    fmt.Println(stu.age)
    fmt.Println((*stu).age)
}
```
