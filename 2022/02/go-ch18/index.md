# Ch18 自定义类型与结构体


# ch18 自定义类型与结构体

## 自定义类型

Go语言的关键字type可以将各种基本类型定义为自定义类型，而且每种自定义类型可以拥有自己的方法。

例如,

```go
package main

import "fmt"

type MyInt int32

func (m MyInt) Add(x MyInt) MyInt {
		return m + x
}
func main() {
		var x MyInt = 2
		var y MyInt = 3
		fmt.Println(x.Add(y))
}
```

`type 类型名 struct{}` 可以理解为，将`struct{}`结构体定义为类型名的类型。结构体是一种复合的基本类型，通过type定义为自定义类型后，使结构体更便于使用。

## **结构体定义**

```go
type identifier struct {
    field1 type1
    field2 type2
    ...
}
```

- 结构体是复合类型、值类型
- `type T struct { a, b int}`也合法
- 如果字段在代码中从来也不会用到，可以命名为`_`
- 结构体的字段可以是任何类型，甚至是结构体本身，也可以是函数或接口

## **实例化、初始化**

Point结构体定义为

```go
type Point struct {
    X, Y int
}
```

### **基本形式**

**先声明，后初始化**

```go
var p Point
p.X, p.Y = 10, 20
```

**实例化时，使用结构体字面量进行初始化**

```go
p1 := Point{X:20, Y:15}
p2 := &Point{X:10, Y:20}
```

- 其中， `&Point{10, 20}`是一种简写，底层仍然会调用`new()`
- 若不指定变量，则必须与定义顺序一致

```go
type Interval struct {
    start int
    end   int
}
intr := Interval{0, 3}            (A)
intr := Interval{end:5, start:1}  (B)
intr := Interval{end:5}           (C)
```

### **new函数**

```go
point := new(Point)
point.X, point.Y = 10, 20
```

这里，使用new获得的变量其实是`*Point`类型，但由于Go提供的语法糖，使得开发者访问结构体指针的成员变量时更加方便。该语法糖自动地将`point.X`转换为`(*point).X`

无论是一个**结构体类型**还是一个**结构体类型指针**，都可以使用`name.fieldName`来引用结构体字段

```go
type myStruct struct { i int }
var v myStruct    // v是结构体类型变量
var p *myStruct   // p是指向一个结构体类型变量的指针
v.i
p.i
```

![0018-1.png](ch18%20%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B%E4%B8%8E%E7%BB%93%E6%9E%84%E4%BD%93%2076cc67414a6f417592c955fab74ebf63/0018-1.png)

## **结构体内存布局**

Go 语言中，结构体和它所包含的数据在内存中是以**连续块的形式**存在的，即使结构体中嵌套有其他的结构体，这在性能上带来了很大的优势。下面的例子清晰地说明了这些情况：

```go
type Rect1 struct {Min, Max Point}
tyep Rect2 struct {Min, Max *Point}
```

![0018-2.png](ch18%20%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B%E4%B8%8E%E7%BB%93%E6%9E%84%E4%BD%93%2076cc67414a6f417592c955fab74ebf63/0018-2.png)

[Go内存分配](Go%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%20175abebb474545b18f94bdb266a6380f.md)

## **结构体工厂**

Go语言不支持面向对象编程语言中的构造方法，但是在Go中可以为类型定义一个工厂，按惯例，以`new`或`New`开头，假设定义了如下`File`结构体类型

```go
type Cat struct {
    Color   string
    Name    string
}
```

下面是该结构体类型对应的工厂方法，返回一个指向结构体实例的指针

```go
func NewCat(color, name string) *Cat {
    return &Cat{color, name}
}
```

调用方法

```go
cat := NewCat("yellow", "mimi")
```

## **方法**

方法的声明和函数类似，方法是一种作用于特定类型变量的函数。这种特定类型变量叫做接收器（Receiver)。

- 接收器的概念类似于其他语言中的this或self
- 在Go语言中，接收器的类型可以是任何类型，不仅仅是结构体，任何类型都可以拥有方法。
  
    注：不能是基础类型，也不能是接口。基础类型可以通过type自定义类型之后使用。
    

### **添加方法**

格式如下：

```go
func (接收器变量 接收器类型) 方法名(参数列表) (返回参数) {
    函数体
}
```

接受器变量：命名时，官方建议使用接收器类型名的第一个小写字母

接收器类型：可以是指针类型和非指针类型

方法名、参数列表、返回参数：同函数

```go
type person struct {
    name string
}

func (p person) String() string {
    return "the person name is" + p.name
}
```

现在，类型person有了一个String方法，可通过如下方式进行调用

```go
func main(){
    p := person{name:"张三"}
    fmt.Println(p.String())
}
```

### **接收器**

- 接收器可以是（几乎）任何类型，不仅仅是结构体类型，任何类型都可以有方法，甚至是函数类型，可以是int、bool、string或数组的**别名类型**，但不能是一个接口类型
- 在Go中，类型的代码和绑定在它上面的方法的代码可以不放置在一起，可以存放在不同的源文件中，**但必须在同一个包中**。也就是说，**不能够给别的包中定义的类型追加方法**，所以无法为`int`追加方法，但是可以为`int`重新命名后，追加方法。
- **别名类型没有原始类型上已经定义的方法**
  
    ```go
    package main
    
    import "fmt"
    
    type MyInt int
    
    func (m MyInt) IsZero() bool {
        return m == 0
    }
    
    func (m MyInt) Add(other MyInt) MyInt {
        return other + m
    }
    
    func main() {
        var a MyInt = 3
        fmt.Println(a.IsZero())
        fmt.Println(a.Add(MyInt(2)))
    }
    ```
    
- 类型T(或*T)上的所有方法的集合叫它的方法集(method set)
- 方法是函数，不允许方法重载。但具有同样名字的方法可以在2个或以上接收者类型上存在。
  
    ```go
    func (a *denseMatrix) Add(b Martix) Matrix
    func (a *denseMatrix) Add(b, c Matrix) Matrix  //不允许
    func (a *sparseMatrix) Add(b Matrix) Matrix //允许
    ```
    

**指针类型与非指针类型接收器**

在调用方法的时候，本质上都是值传递，只不过一个这个值副本，一个是指向这个值指针的副本。指针具有指向原有值的特性，所以修改了指针指向的值，也就修改了原有的值。

```go
package main

import "fmt"

type person struct {
    name string
}

func main() {
    p := person{name:"Tom"}
    //p := person{name:"Tom"}
    p.modify()
    p.output()
}

func (p person) output() {
    fmt.Println("the person name is " + p.name)
}

func (p *person) modify() {
    p.name = "Tony"
}
```

**自动取指针与自动解引用**

在本节曾提到一个语法糖，这里`stu.name`其实应写为`(*stu).name`，记为**自动解引用**，下面介绍**自动取指针**。

```go
package main

type student {
    name string
    age  int
}

func main() {
    stu := new(student)
    stu.name = "Lily"
    stu.age = 20
}
```

在指针类型与非指针类型接收器中的例子中，在调用接收器为指针类型的方法时，使用的是一个变量，并不是一个指针类型，其标准写法为，

```go
(&p).modify()
```

如果没有强制使用指针进行调用，Go编译器会自动取指针，这是Go支持的一个语法糖。

同样的，如果是非指针类型接收器，使用指针类型调用，Go编译器会自动解引用。

```go
(*p).String()
```

**区别**

当需要**修改接收者中的值时**或**接收者拷贝代价比较大时**用指针接收器

### **方法和函数的统一调用**

无论时普通函数还是结构体的方法，只要签名一致，就都可以被赋值于同一签名的函数变量。

```go
package main

import "fmt"

type class struct{
}

func (c *class) Do(v int) {
    fmt.Printf("call method do: %d\n", v)
}

func funcDo(v int) {
    fmt.Printf("call function do: %d\n", v)
}

func execute(v int, entry func(int)) {
    entry(v)
}

func main() {
    c := new(class)
    execute(100, c.Do)
    execute(100, funcDo)
}
```

### **方法与未导出字段**

`firstName`和`lastName`被设置为未导出字段，在包外无法访问，可通过导出的方法对其进行修改与访问。

```go
package person

type Person struct {
    firstName string
    lastName  string
}

func (p *Person) FirstName() string {
    return p.firstName
}

func (p *Person) SetFirstName(newName string) {
    p.firstName = newName
}

```

```go
package main

import (
    "./person"
    "fmt"
)

func main() {
    p := new(person.Person)
    // p.firstName undefined
    // (cannot refer to unexported field or method firstName)
    // p.firstName = "Eric"
    p.SetFirstName("Eric")
    fmt.Println(p.FirstName()) // Output: Eric
}

```

对象的字段（属性）不应该由2个或2个以上的不同线程在同一时间去改变。如果在程序中发生这种情况，为了安全并发访问，可以使用包sync或者使用`goroutines`和`channels`

## **类型内嵌和结构体内嵌**

结构体允许其成员字段在声明时没有字段名而只有类型，这种形式的字段被称为**类型内嵌**或**匿名字段**。

没有字段名的变量，在初始化和调用时直接使用类型，因此同种类型的匿名字段只能有一个。

### **类型内嵌**

```go
package main

import "fmt"

type student struct{
    name string
    string
    int
}

func main(){
    var stu1 student = student {
        name:"haojie",
        string:"test",
        int:6,
    }
    fmt.Println(stu1.string)
}
```

### **结构体内嵌**

普通的结构体嵌套如下，

```go
type address struct{
    province string
    city     string
}
type student struct{
    name string
    add address
}

func main(){
    stu1 := student{
        name:"haojie",
        add:address{
            province:"河北",
            city:"xiongan",
        }
    }
}
```

但，**结构体中的匿名字段可以是结构体**，此时可以直接访问匿名结构体中的所有成员，这种方式被成为**结构体内嵌**

```go
package main

import "fmt"

type address struct {
    province string
    city     string
}
type people struct {
    name string
    address
}

func main() {
    // 内嵌结构体的初始化方式
    p1 := people{
        name: "chao",
        address: address{
            "zhejiang",
            "jiaxing",
        },
    }
    // 这样不行
    // p2 := people {
    //  name : "hui",
    //  province : "henan",
    //  city : "luohe",
    // }
    fmt.Println(p1.province)
    //fmt.Println(p2.province)
}
```

在people中没有找到province变量，便去匿名结构体中寻找，但如果两个匿名结构体中都含有相同变量，则编译器无法识别会报错。

- 无论结构体有多少层嵌入结构体，结构体实例访问任意一级的嵌入结构体成员时都只用给出字段名。
- 内嵌结构体字段仍然可以使用详细的字段进行一层层访问，内嵌结构体的字段名就是它的类型名。

```go
package main

import "fmt"

type contract struct {
    phone string
    wechat string
    qq string
}

type address struct {
    province string
    city     string
}

type people struct {
    age int
    name string
    contract
}

type student struct {
    id int
    address
    people
}

func main() {
    stu := student {
        id : 1,
        address : address {
            province : "zhejiang",
            city : "jiaxing",
        },
        people : people {
            age : 23,
            name : "chao",
            contract : contract {
                phone : "159xxxx6132",
                wechat : "mcgt80611",
                qq : "864654710",
            },
        },
    }
    fmt.Println(stu.qq)
    fmt.Println(stu.address.province)
    fmt.Println(stu.people.contract.phone)
}
```

### **结构体内嵌模拟“继承”**

当一个匿名类型被内嵌在结构体中时，匿名类型的可见方法也同样被内嵌，这在效果上等同于外层类型继承了这些方法。

```go
package main

import "fmt"

type animal struct {
    name string
}

func (a animal) move() {
    fmt.Printf("%s is moving"\n, a.name)
}

type dog struct {
    feetNum int
    animal
}

func (d dog) bark() {
    fmt.Printf("%s is barking\n", d.name)
}

func main() {
    d := dog{
        feetNum: 4,
        animal: animal{
            name: "旺财",
        },
    }
    d.bark()
    d.move()
}
```

### **多重继承**

多重继承指的是类型获得多个父类型行为的能力，它在传统的面向对象语言中通常是不被实现的（C++和Python例外）。因为在类继承层次中，多重继承会给编译器引入额外的复杂度。但是在Go语言中，通过在类型中嵌入所有必要的父类型，可以很简单的实现多重继承。

练习：假设有一个类型`CameraPhone`，通过它可以`Call()`，也可以`TakeAPicture()`，但是第一个方法属于类型`Phone`，第二个方法属于类型`Camera`，请编写相关代码。

```go
package main

import "fmt"

type Phone struct {
    cpuKind    string
    memorySize int
}

type Camera struct {
    foucs int
    depth float64
}

type SmartPhone struct {
    prize float64
    Phone
    Camera
}

func main() {
    sp := SmartPhone{3999.0, Phone{"qilin", 1024}, Camera{6, 123.7}}
    fmt.Println(sp)
    sp.Call()
    sp.TakeAPicture()
}

func (p *Phone) Call() {
    fmt.Println("calling")
}

func (c *Camera) TakeAPicture() {
    fmt.Println("taking")
}
```

关于利用结构体与面向对象程序设计的相关问题，将在后续再系统地介绍。

[Go面向对象](Go%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%2035dcd5cec8dc4b619b013008a3cf705f.md)

### **命名冲突**

当两个字段拥有相同的名字（可能是继承来的名字）时该如何处理？

- 外层名字会覆盖内层的名字，但是二者的内存空间都保留，这提供了一种重载字段或方法的方式
- 如果相同的名字在同一级别出现了两次，如果这个名字被程序使用了，将会引发一个错误。没有办法来解决这种二义性，必须由程序员自己修正。

**规则一**

```go
type A struct {
    a int
}
type B struct {
    a float32
    A
}

func testStruct() {
    var b B
    b.a = 5.4
    fmt.Println(b)
    b.A.a = 12
    fmt.Println(b)
}
//{5.4 {0}}
//{5.4 {12}}

```

**规则二**

```go
type A struct {
    a int
}

type B struct {
    a, b int
}

type C struct {
    s1 A
    s2 B
}

type D struct {
    A
    B
}

func testStruct() {
    var c C
    c.s1.a = 5
    c.s2.a = 3
    fmt.Println(c)
    var d D
    d.a = 4
    fmt.Println(d)
}
//{{5} {3 0}}

//.\main.go:54:3: ambiguous selector d.a
```

## **结构体标签**

### **JSON序列化与反序列化**

[json](Go%E5%B7%A5%E5%85%B7%E5%8C%85%208443f12a86ef4d75bfeab38a6b16e34e/json%200add934e6a264c47b5841f14625cb8d8.md)

JSON是一种轻量级的数据交换格式，JSON键值对是用来保存JS对象的一种方式。

```go
// Student json
type Student struct {
    ID     int
    Gender string
    Name   string
}

func main() {
    stu1 := Student{
        ID:     1,
        Gender: "male",
        Name:   "chao",
    }
    v, err := json.Marshal(stu1)
    if err != nil {
        fmt.Println("error")
    } else {
        fmt.Println(v)
        fmt.Println(string(v))
        fmt.Printf("%#v\n", string(v))
    }
    str := "{\"ID\":1,\"Gender\":\"male\",\"Name\":\"chao\"}"
    stu2 := new(Student)
    json.Unmarshal([]byte(str), stu2)
    fmt.Println(stu2)
}
```

### **JSON标签**

**如何把序列化结果中的ID改成小写的？如果将结构体中的变量改成小写，json包将无法访问这些数据。**

- 解决方法：使用结构体Tag

结构体中的字段除了有名字和类型外，还可以有一个可选的标签（tag）：它是一个附属于字段的字符串，可以是文档或其他的重要标记。

标签的内容不可以在一般的编程中使用，只有包 `reflect` 能获取它。

将在反射中深入的探讨 `reflect`包，它可以在运行时自省类型、属性和方法，比如：在一个变量上调用 `reflect.TypeOf()` 可以获取变量的正确类型，如果变量是一个结构体类型，就可以通过 Field 来索引结构体的字段，然后就可以使用 Tag 属性。

[ch24 反射](ch24%20%E5%8F%8D%E5%B0%84%20d71899dbfd8b478785fa0f1270e3ba7b.md)

```go
// Student json
type Student struct {
    ID     int    `json:"id"`
    Gender string `json:"gender"`
    Name   string `json:"name"`
}
```

### 多个结构体标签的写法

如下所示，在````字符串内，使用空格符分隔

```go
type Student struct {
    ID     int    `json:"id" gorm:"primary_key`
    Gender string `json:"gender"`
    Name   string `json:"name"`
}
```

## **类型的String()方法**

使用`String()`方法来定制类型的字符串形式的输出，即一种可阅读性和打印性的输出。

如果类型定义了`String()`方法，它会被用在`fmt.Printf()`中生成默认的输出，等同于使用格式化描述符`%v`产生的输出。

在`fmt.Print()`和`fmt.Println()`中也会自动使用`String()`方法

```go
package main

import (
    "fmt"
    "strconv"
)

type TwoInts struct {
    a int
    b int
}

func main() {
    two1 := new(TwoInts)
    two1.a = 12
    two1.b = 10
    fmt.Printf("two1 is: %v\n", two1)
    fmt.Println("two1 is:", two1)
    fmt.Printf("two1 is: %T\n", two1)
    fmt.Printf("two1 is: %#v\n", two1)
}

func (tn *TwoInts) String() string {
    return "(" + strconv.Itoa(tn.a) + "/" + strconv.Itoa(tn.b) + ")"
}
//two1 is: (12/10)
//two1 is: (12/10)
//two1 is: *main.TwoInts
//two1 is: &main.TwoInts{a:12, b:10}
```

当广泛使用一个自定义类型时，最好为它定义 `String()`方法。从上面的例子也可以看到，格式化描述符 `%T` 会给出类型的完全规格，`%#v` 会给出实例的完整输出，包括它的字段（在程序自动生成 `Go` 代码时也很有用）。

例如：实现栈（stack）数据结构

```go
package main

import "fmt"
import "strconv"

const LIMIT = 20

type stackArr struct {
    arr [LIMIT]int
    top int
}

func newStackArr() *stackArr {
    return &stackArr{arr: [LIMIT]int{}, top: -1}
}

type Stack struct {
    *stackArr
}

func NewStack() *Stack {
    return &Stack{newStackArr()}
}

func (s *Stack) Pop() int {
    res := -1
    if !s.Empty() {
        res = s.arr[s.top]
        s.arr[s.top] = 0
        s.top--
    } else {
        fmt.Println("error, the stack is empty")
    }
    return res
}

func (s *Stack) Push(num int) {
    if !s.Full() {
        s.top++
        s.arr[s.top] = num
    } else {
        fmt.Println("error, the stack is full")
    }
}

func (s *Stack) Empty() bool {
    return s.top == -1
}

func (s *Stack) Top() int {
    if !s.Empty() {
        return s.arr[s.top]
    }
    fmt.Println("error, the stack is empty, no top value")
    return -1
}

func (s *Stack) Full() bool {
    return s.top == LIMIT-1
}

func (s *Stack) String() string {
    str := "{"
    for i, num := range s.arr {
        str += "[" + strconv.Itoa(i) + ":" + strconv.Itoa(num) + "]"
    }
    str += "}"
    return str
}

func main() {
    s := NewStack()
    s.Push(5)
    s.Push(4)
    s.Push(68)
    fmt.Println(s)
    s.Pop()
    fmt.Println(s)
    s.Pop()
    s.Pop()
    s.Pop()
    s.Top()
}
```

## **垃圾回收和SetFinalizer**

- Go开发者不需要写代码来释放程序中不再使用的变量和结构占用的内存，在Go运行时有一个独立的进程，即垃圾收集器（GC）会处理这些事情。

[ch33 垃圾回收与变量逃逸分析](ch33%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E4%B8%8E%E5%8F%98%E9%87%8F%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90%20860c699cfcf840f090738a8815ecfe8a.md)

- 可以通过`runtime`包访问GC进程

[runtime](Go%E5%B7%A5%E5%85%B7%E5%8C%85%208443f12a86ef4d75bfeab38a6b16e34e/runtime%20475d96c5f67a434282c820e15b62dd08.md)

通过调用`runtime.GC()`函数可以显式的触发GC，但这只在某些罕见的场景下才有用，比如当内存资源不足时调用`runtime.GC()`，它会在此函数执行的点上立即释放一大片内存，此时程序可能会有短时的性能下降（因为GC进程在执行）

```go
var m runtime.MemStats
runtime.ReadMemStats(&m)
fmt.Printf("%d Kb\n", m.Alloc / 1024)
```

上述程序会给出已分配内存的总量，单位是Kb

如果需要在一个对象obj被从内存移除前执行一些特殊操作，比如写到日志文件里，可以通过如下方式调用函数来实现

```go
runtime.SetFinalizer(obj, func(obj *typeObj))
```

`func(obj *typeObj)` 需要一个 `typeObj` 类型的指针参数 `obj`，特殊操作会在它上面执行。`func` 也可以是一个匿名函数。

在对象被 GC 进程选中并从内存中移除以前，`SetFinalizer` 都不会执行，即使程序正常结束或者发生错误。

> [https://studygolang.com/pkgdoc](https://studygolang.com/pkgdoc)
> 
> 
> 终止器会在obj变为不可接触之后的任意时间被调度执行。不保证终止器会在程序退出前执行，因此一般终止器只用于在长期运行的程序中释放关联到某对象的非内存资源。例如，当一个程序丢弃一个os.File对象时没有调用其Close方法，该os.File对象可以使用终止器去关闭对应的操作系统文件描述符。但依靠终止器去刷新内存中的I/O缓冲如bufio.Writer是错误的，因为缓冲不会在程序退出时被刷新。
>
