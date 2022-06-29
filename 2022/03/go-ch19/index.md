# Ch19 接口


# ch19 接口

传统的派生式接口及类关系构建的模式，让类型间拥有强耦合的父子关系。这种关系一般会以“类派生图”的方式进行。经常可以看到大型软件极为复杂的派生树。随着系统功能的不断增加，这棵“派生树”会变得越来越复杂。

对于Go语言来说，非侵入式设计让实现者的所有类型都是平行的、组合的。如何组合则留到使用者编译时再确认，因此使用Go语言时，不需要同时也不可能有“类派生图”，开发者唯一需要关注的就是“我需要什么”，以及“我能实现什么“。

> 接口的实现原理https://www.jianshu.com/p/70003e0f49d1
> 

## **声明接口**

接口是一种约定，它是一个抽象的类型，与见到的具体的类型如int、map、slice等不一样。

具体的类型，可以知道它是什么，并且可以知道可以用它做什么；但是接口不一样，接口是抽象的，它只有一组接口方法，并不知道它的内部实现，虽然不知道接口是什么，但是知道可以利用它提供的方法做什么。

通过如下格式定义接口，其中`Namer`就是一个接口类型

```go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```

## **命名规范**

按照约定，接口的名字有方法名加`[e]r`后缀组成，例如`Printer`，`Reader`、`Writer`、`Logger`、`Converter`等等。还有一些不常用的方式（当后缀`er`不合适时），比如`Recoverable`，此时接口名以`able`结尾，或者以`I`开头

## **接口类型的实现**

接口是一个抽象类型，用来定义行为的类型，但这些定义的行为不是由接口直接实现，而是**通过方法由用户定义的类型实现**。如果用户定义的类型，实现了接口类型声明的所有方法，那么这个用户定义的类型就实现了这个接口，称实现了某一接口的类型为**实体类型**，该实体类型的值可以赋值给接口类型的值。

如上所述，如果要实现一个接口必须实现这个接口提供的所有方法。具体实现时，**实体类型**可以采用**值接收者**或**指针接收者**两种方式实现接口中定义的方法，但这两种方式是有区别的，具体如下。

- 值接收者实现
  
    ```go
    package main
    
    import "fmt"
    
    type animal interface{
        printInfo()
    }
    
    type cat struct {
        name string
        age int
    }
    
    func main(){
        c := cat {
            name : "mimi",
            age : 5,
        }
        var a animal = c
        invoke(a)
    }
    
    // 类型cat使用值接收者实现接口中定义的方法
    func (c cat) printInfo(){
        fmt.Printf("this cat's name is %s and it's age is %d\n", c.name, c.age)
    }
    
    func invoke(a animal) {
        a.printInfo()
    }
    ```
    
    `invoke`函数接收一个`animal`接口类型，例子中传递参数的时候，也是以类型cat的值c传递的，运行程序可以正常执行。现在稍微改造一下，将`*cat`类型赋值给`animal a`，并作为参数传递。
    
    ```go
    func main(){
        c := &cat {
            name : "mimi",
            age : 5,
        }
        var a animal = c
        invoke(a)
    }
    ```
    
    运行程序，发现程序可以正常运行。
    
    **实体类型以值接收者实现接口的时候，不管是实体类型的值，还是实体类型值的指针，都被认为实现了该接口**。
    
- 指针接收者实现
  
    ```go
    func main(){
        c := cat {
            name : "mimi",
            age : 5,
        }
        var a animal = c
        invoke(a)
    }
    
    // 类型cat使用指针接收者实现接口中定义的方法
    func (c *cat) printInfo(){
        fmt.Printf("this cat's name is %s and it's age is %d\n", c.name, c.age)
        // c.name == (*c).name, c.age == (*c.age)
    }
    ```
    
    此时编译器已经提示`cat`没有实现`animal`接口，因为`printInfo`方法是指针接收者，所以值`c`不能作为接口类型animal传参使用。若修改为`var a animal = &c`，则运行成功。
    
    **实体类型以指针接收者实现接口的时候，只有指向这个类型的指针才被认为实现了这个接口。**
    

Go 语言规范定义了接口方法集的调用规则：

- 类型 T 的可调用方法集包含接受者为 T 的所有方法，不包含接受者为* T 的方法
- 类型 *T 的可调用方法集包含接受者为 *T 或 T 的所有方法集

## **内部实现**

Go语言中的接口可以有值，一个接口类型的变量或一个接口值，例如`var ai Namer`，`ai`是一个多字数据结构，它的初始值是`nil`

![0019-1.png](ch19%20%E6%8E%A5%E5%8F%A3%20d0c5e5fb1a054e34a7404085ea76be7d/0019-1.png)

实现了`Namer`接口的类型的变量可以赋值给`ai`(即`receiver`的值)，方法表指针`（method table ptr)`就指向了当前的方法实现。当另一个实现了`Namer`接口的类型的变量被赋给`ai`时，`receiver`的值和方法表指针也会相应改变。

```go
func main(){
    var b bytes.Buffer
    fmt.Fprint(&b, "hello world")
    var w io.Writer
    w = &b
    fmt.Println(w)
}
```

这里例子中，因为bytes.Buffer实现了接口`io.Writer`，所以可以通过`w=&b`对`w`进行赋值。这个赋值的操作会把定义类型的值存入接口类型的值。

接口的值被赋值后，接口值内部的布局。接口的值是一个两个字长度的数据结构，第一个字包含**一个指向内部表结构的指针，这个内部表里存储的有实体类型的信息以及相关联的方法集**；第二个字包含的是**一个指向存储的实体类型值的指针**。

所以接口的值结构其实是两个指针，这也可以说明接口其实是一个引用类型。

[ch39 interface底层实现](ch39%20interface%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0%20526943e33e8d4c15b07eead6a45cc538.md)

## **特性**

- 多个类型可以实现一个接口
- 一个类型可以实现多个接口
- 实现某个接口的类型除了实现接口方法外，可以有其他的办法
- 即使接口在类型之后才定义，二者处于不同的包中，被单独编译：只要类型实现了接口中的方法，它就实现了此接口。

## **接口嵌套组合**

一个接口可以包含一个或多个其他的接口，这相当于直接将这些内嵌接口的方法列举在外层接口中一样。

比如`File`中包含了`ReadWriter`和`Locker`的所有方法，还额外有一个`Close()`方法

```
type ReadWrite interface {
    Read(b Buffer) bool
    Write(b Buffer) bool
}

type Locker interface {
    Lock()
    UnLock()
}

type File interface {
    ReadWrite
    Locker
    Close()
}
```

## **类型断言**

一个接口类型的变量`varI`中可以包含任何类型的值，必须有一种方式来检测它的**动态**类型，即运行时在变量中存储的值的实际类型。

在执行过程中动态类型可能会有所不同，通常可以使用**类型断言**来测试在某个时刻`varI`是否包含类型`T`的值。

`varI`必须是一个接口变量，否则编译器会报错

```go
if v, ok := varI.(T); ok {
    Process(v)
    return
}
```

如果转换合法，`v`是`varI`转换到类型`T`的值，`ok`会是`true`；否则`v`是类型`T`的零值，`ok`是false。

```go
package main

import "fmt"

type Shaper interface {
    Area()
}

type Circle struct {
    radius float32
}

func main() {
    var area Shaper
    area = &Circle{radius:3}

    if t, ok := area.(*Circle); ok {
        fmt.Printf("The type of areaIntf is: %T\n", t)
    }
}

func (c *Circle) Area(){
    fmt.Println("circle's area")
}
```

### **type-switch结构**

- 接口变量的类型也可以使用一种特殊形式的`switch`来检测
- 此外，在处理来自于外部的、类型未知的数据时，比如解析诸如JSNO或XML编码的数据时，类型测试和转换会非常有用

```go
switch x.(type) {
    case bool:
    // do something
    case int:
    // do something
    default:
    //do something
}
```

## 常见接口

### **Sort接口**

```go
type Interface interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
}
```

### **读写接口**

读和写是软件中很普遍的行为，提起它们就会立即想到读写文件、缓存（比如字节或字符串切片）、标准输入输出、标准错误以及网络连接、管道等，或者读写我们的自定义类型。为了让代码尽可能通用，GO采取了一致的方式来读写数据。

`io`包提供了用于读和写的接口`io.Reader`和`io.Writer`

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

只要类型实现了读写接口，提供 `Read()` 和 `Write` 方法，就可以从它读取数据，或向它写入数据。一个对象要是可读的，它必须实现 `io.Reader` 接口，这个接口只有一个签名是 `Read(p []byte) (n int, err error)` 的方法，它从调用它的对象上读取数据，并把读到的数据放入参数中的字节切片中，然后返回读取的字节数和一个 `error` 对象，如果没有错误发生返回 `nil`，如果已经到达输入的尾端，会返回 `io.EOF("EOF")`，如果读取的过程中发生了错误，就会返回具体的错误信息。类似地，一个对象要是可写的，它必须实现 `io.Writer` 接口，这个接口也只有一个签名是 `Write(p []byte) (n int, err error)` 的方法，它将指定字节切片中的数据写入调用它的对象里，然后返回实际写入的字节数和一个 `error` 对象（如果没有错误发生就是 `nil`）。

`io` 包里的 `Readers` 和 `Writers` 都是不带缓冲的，`bufio` 包里提供了对应的带缓冲的操作，在读写 `UTF-8` 编码的文本文件时它们尤其有用。

[ch28 进阶输入输出](ch28%20%E8%BF%9B%E9%98%B6%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA%20784fbd0a42d743aabb1b928aff653039.md)

## **空接口**

**空接口**不包含任何办法，它对实现不做任何要求

```
type Any interface {}

```

任何其他类型都实现了空接口，`any`或`Any`是空接口一个很好的别名或缩写

可以给空接口类型的变量`var val interface {}` 赋予任何类型的值

空接口是一种接口，它是一种指针类型的数据类型，虽然不严谨，但它确实保存了两个指针，**一个是对象的类型(或iTable)，一个是对象的值。**所以赋值过程是让空接口any保存各个数据对象的类型和对象的值。

```go
var test interface{}
test = "string"
```

### **利用空接口构建通用类型的数组**

让我们给空接口定义一个别名类型`Element:type Element interface{}`

然后定义一个容器类型的结构体`Vector`，它包含一个`Element`类型元素的切片

```go
type Element interface{}

type Vector struct {
    a  []Element
}

func (p *Vector) At(i int) Element {
    return p.a[i]
}

func (p *Vector) Set(i int, e Element) {
    p.a[i] = e
}
```

### **复制数据切片至空接口切片**

假设有一个`myType`类型的数据切片，若想将切片中的数据赋值到一个空接口切片中，类似

```go
var dataSlice []myType = FuncReturnSlice()
var interfaceSlice []interface{} = dataSlice
```

可惜不能这么做，编译时会出错：

`cannot use dataSlice (type []myType) as type []interface { } in assignment`

原因是它们俩在内存中的布局是不一样的（参考 [Go wiki](https://github.com/golang/go/wiki/InterfaceSlice)）。

必须使用`for-range`语句来一个一个显示地赋值：

```go
var dataSlice []myType = FuncReturnSlice()
var interfaceSlice []interface{} = make([]interface{}, len(dataSlice))
for i, d := range dataSlice {
    interfaceSlice[i] = d
}
```

### **通用类型的节点数据结构**

```go
package main

import "fmt"

type Node struct {
    le *Node
    data interface{}
    ri *Node
}

func NewNode(left, right *Node) *Node {
    return &Node{left, nil, right}
}

func (n *Node) SetDate(data interface{}) {
    n.data = data
}

func main() {
    root := NewNode(nil, nil)
    root.SetData("root node")
    a := NewNode(nil, nil)
    a.SetData("left node")
    b := NewNode(nil, nil)
    b.SetData("right node")
    root.le = a
    root.ri = b
    fmt.Printf("%v\n", root)
}
```

## **Go的动态类型**

Go语言没有类：数据（结构体或更一般的类型）和方法是一种松耦合的正交关系

Go中的接口跟Java、C#类似：都是必须提供一个指定方法集的实现。但是更加灵活通用：任何提供了接口方法实现代码的类型都隐式地实现了该接口。

和其他语言相比，Go是唯一结合了`接口值`，`静态类型检查（该类型是否实现了某个接口）`，`运行时动态转换的语言`，并且不需要显示地声明类型是否满足某个接口。 该特性允许我们在不改变已有代码地情况下定义和使用新接口。

类似于Python和Ruby这类动态语言中的`动态类型duck typing`

这意味这对象可以根据提供的方法被处理，而忽略它们的实际类型：它们能做什么比它们是什么更重要。

- 例1
  
    ```go
    package main
    
    import "fmt"
    
    type IDuck interface {
        Quack()
        Walk()
    }
    
    type Bird struct {
        // ...
    }
    
    func main() {
        b := new(Bird)
        DuckDance(b)
    }
    
    func (b *Bird) Quack() {
        fmt.Println("I am quacking!")
    }
    
    func (b *Bird) Walk()  {
        fmt.Println("I am walking!")
    }
    
    func DuckDance(duck IDuck) {
        for i := 1; i <= 3; i++ {
            duck.Quack()
            duck.Walk()
        }
    }
    ```
    
    如果对 `cat` 调用函数 `DuckDance()`，Go 会提示编译错误，但是 Python 和 Ruby 会以运行时错误结束。
    
- 例2
  
    ```go
    pakcage main
    
    import "fmt"
    
    var ai AbsInterface
    var si SqrInterface
    var empty interface{}
    
    func main() {
        pp := new(Point)
    
        // 任意对象包括pp都实现了空接口，所以可以赋值给空接口empty
        empty = pp
        fmt.Printf("%T\n", empty)
    
        ai = empty.(AbsInterface)
        // ai = empty
        fmt.Printf("%T\n", ai)
    
        si = empty.(SqrInterface)
        fmt.Printf("%T\n", si)
    
        si = ai.(SqrInterface)
        fmt.Printf("%T\n", si)
    }
    
    type Point int
    
    type AbsInterface interface {
        Abs() float32
    }
    
    type SqrInterface interface {
        Sqr() float32
    }
    
    func (p *Point) Abs() float32 {
        return 10.0
    }
    
    func (p *Point) Sqr() float32 {
        return 10.0
    }
    
    ```
    
- 例3
  
    ```go
    package main
    
    import "fmt"
    
    func main() {
        mi := MyInt(5)
        var x Adder = &mi
        x.add()
        // x.print()
        // x.print undefined (type Adder has no field or method print)
        x.(Printer).print()
    }
    
    type Printer interface {
        print()
    }
    
    type Adder interface {
        add()
    }
    
    type MyInt int
    
    func (i *MyInt) add() {
        fmt.Printf("%d\n", *i+1)
    }
    
    func (i *MyInt) print() {
        fmt.Printf("%v\n", *i)
    }
    ```
    
    `x` 转换为 `Printer`类型是完全动态的：只要 `x` 的底层类型（动态类型）定义了 `print` 方法这个调用就可以正常运行（译注：若 `x` 的底层类型未定义 `print` 方法，此处类型断言会导致 `panic`，最佳实践应该为 `if mpi, ok := x.(myPrintInterface); ok { mpi.print() }`）。
    
    针对上述代码的个人理解：这里的话，如果一个对象实现了很多很多接口，然后将它赋值给一个接口变量，那么该对象实现的其他接口会暂时被隐藏起来。
    

Go 提供了动态语言的优点，却没有其他动态语言在运行时可能发生错误的缺点。对于动态语言非常重要的单元测试来说，这样既可以减少单元测试的部分需求，又可以发挥相当大的作用。

## **接口的提取**

`提取接口`是非常有用的设计模式，可以减少需要的类型和方法数量，而且不需要像传统的基于类的面向对象语言那样维护整个的类层次结构。

有用的接口可以在开发的过程中被归纳出来。添加新接口非常容易，因为已有的类型不用变动（仅仅需要实现新接口的方法）。已有的函数可以扩展为使用接口类型的约束性参数：通常只有函数签名需要改变。对比基于类的 OO 类型的语言在这种情况下则需要适应整个类层次结构的变化。

因此，开发者不用提前设计出所有的接口；`整个设计可以持续演进，而不用废弃之前的决定`。类型要实现某个接口，它本身不用改变，只需要在这个类型上实现新的方法。

## **接口的继承**

[ch18 自定义类型与结构体](ch18%20%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B%E4%B8%8E%E7%BB%93%E6%9E%84%E4%BD%93%2076cc67414a6f417592c955fab74ebf63.md)

当一个类型包含（内嵌）另一个类型（实现了一个或多个接口）的指针时，这个类型就可以使用（另一个类型）所有的接口方法。

- 例1
  
  
    ```go
    type Task struct {
        Command string
        *log.Logger
    }
    ```
    
    这个类型的工厂方法像这样：
    
    ```go
    func NewTask(command string, logger *log.Logger) *Task {
    		return &Task{command, logger}
    }
    ```
    
    当 `log.Logger` 实现了 `Log()` 方法后，Task 的实例 task 就可以调用该方法：
    
    ```go
    task.Log()
    ```
    
    类型可以通过继承多个接口来提供像 `多重继承` 一样的特性：
    
    ```
    type ReaderWriter struct {
        *io.Reader
        *io.Writer
    }
    ```
    
- 例2
  
    ```go
    type intferface1 interface {
        read()
    }
    
    type intferface2 interface {
        write()
    }
    
    type myStruct1 struct {
        *myStruct2
    }
    
    type myStruct2 struct {
    
    }
    
    func (t *myStruct1) read() {
        fmt.Println("reading")
    }
    
    func (t *myStruct2) write() {
        fmt.Println(("writing"))
    }
    
    func main() {
        test := new(myStruct1)
        test.write()
        test.read()
    }
    ```
