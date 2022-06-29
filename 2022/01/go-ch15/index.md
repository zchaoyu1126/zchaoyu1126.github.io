# Ch15 匿名函数与闭包


# ch15 匿名函数与闭包

## **第一类值**

在Go语言中，函数被看作第一类值(first-class values)：**①函数拥有类型**，可以**②被赋值给其他变量**，**③作为参数传递给函数**，**④作为参数被返回**。

> 第一类值（一等公民）概念出自函数式编程，在此不做介绍
> 

```go
// 函数的类型
package main

import "fmt"

func main() {
    fmt.Printf("%T\n", addInt)
    fmt.Printf("%T\n", addFloat64)
}

func addInt(x, y int) int {
    return x + y
}

func addFloat64(x, y float64) float64 {
    return x + y
}
// func(int, int) int
// func(float64, float64) float64
```

```go
package main

import "fmt"

func main() {
    // 作为返回值，赋值给其他变量
    op1 := selector('+')
    op2 := selector('-')
    // 作为参数
    calculator(20, 30, op1)
    calculator(20, 30, op2)
}

func add(a, b int) int {
    return a + b
}
func sub(a, b int) int {
    return a - b
}
func selector(op byte) func(int, int) int {
    switch op {
        case '+':
        return add
        case '-':
        return sub
        default:
        return nil
    }
}

func calculator(x, y int, f func(int, int) int) {
    res := f(x, y)
    fmt.Println(res)
}
```

## **匿名函数**

匿名函数与之前提到的函数区别在于没有函数名，只有函数体。

例如，

```go
func (x, y int) int {
    return x + y
}
```

匿名函数不能单独存在，一般有如下的使用方式

```go
func main() {
    //1.加上括号，直接调用
    fun () {
        fmt.Println("hi")
    }()
    //2.赋值给变量、作为参数或返回值等
    //2.1函数字面量复制给变量
    //:=之后是一个寻找`nums`数组中第一个大于等于`target`的函数字面量
    //将其赋值给`binarySearchFirst`，编译器会自动推导`binarySearchFirst`的类型。
    binarySearchFirst := func(nums []int, target int) int {
        low, high := 0, len(nums)-1
        for low <= high {
            mid := (low + high) / 2
            if nums[mid] >= target {
                high = mid - 1
            } else {
                low = mid + 1
            }
        }
        return low
    }
}
```

一个返回值为另一个函数的函数可以被称之为工厂函数，这在需要创建一系列相似的函数时非常有用。书写一个工厂函数而不是针对每种情况都书写一个函数。

> 此代码涉及了闭包的概念，见下一小节
> 

```go
//2.2作为返回值
func MakeAddSuffix(suffix string) func(string) string {
    return func(name string) string {
        if !strings.HasSuffix(name, suffix) {
            return name + suffix
        }
        return name
    }
}

func main() {
    addBmp := MakeAddSuffix(".bmp")
    addJpeg := MakeAddSuffix(".jpeg")
    addBmp("file") //file.bmp
    addJpeg("file") //file.jpeg
}
```

```go
//2.3作为参数
package main

import "fmt"

func main() {
    calculate(20, 30, func(a, b int) {
        fmt.Println(a + b)
    })
}

func calculate(x, y int, f func(int, int)) {
    f(x, y)
}
```

观察可得，**匿名函数**与**函数**的用途非常相似，如何将二者联系起来？

在**ch6常量与变量**中曾提及有名常量和无名常量，无名常量例如`2`，有名常量即通过const 定义的有名常量`const num int = 2`。因此，使用如下形式声明函数时，可以理解为一个匿名函数绑定一个唯一的名字。这个过程就类似于声明一个有名常量，为无名常量`2`绑定一个名称，之后就可以使用名称来代指`2`。

> 这里的描述为个人理解，具体还有待商榷，对结构体中的方法是否适用？要验证其正确性，需要去了解函数与结构体中的方法底层实现的差别。未完待续
> 

常量有时称为字面量（literal），基于二者的相似性，匿名函数也可被称为函数字面量

```go
func functionName(param1 type1, param2 type2) (type3) {
    ...
    return res
}
```

## **闭包**

拥有函数名的函数只能在包级声明，不能在函数内部声明，但可以引入函数字面量（匿名函数）来解决此需求。

例如，在函数中想要对一棵树进行遍历，但又不想在函数外另外再声明一个函数，可以使用匿名函数的方式，将匿名函数赋值给`traverse`，之后便可以向正常调用函数一样，使用`traverse`函数。

**leetcode 563** 求解一棵二叉树的坡度

- 一棵树的坡度定义为，所有节点的坡度之和。
- 树中一个节点`Node`的坡度定义为，`Sum(Node.Left)`与`Sum(Node.Right)`差的绝对值。
- `Sum`函数是以该节点为根的树中所有节点的值之和。

在函数内部引入了匿名函数辅助解决一些模块化的问题时，免不了需要进行数据的交互，例如在上述问题中需要考虑如何记录各个节点的坡度之和。问题的难点在于`traverse`函数如何在递归遍历的过程中，将计算得到的各节点坡度信息传递给`FindTilt`函数。

使用`C++`解决的方法可以是，①设置一个成员变量（或者全局变量），令`dfs`函数在递归遍历的过程中修改`ans`值，②修改dfs函数的定义，额外传入一个int类型的引用，用于记录结果。

- C++解决办法
  
    ```cpp
    // 成员变量
    class Solution {
    public:
        int ans = 0;
    
        int findTilt(TreeNode* root) {
            dfs(root, ans);
            return ans;
        }
    
        int dfs(TreeNode* node, &int ans) {
            if (node == nullptr) {
                return 0;
            }
            int sumLeft = dfs(node->left);
            int sumRight = dfs(node->right);
            ans += abs(sumLeft - sumRight);
            return sumLeft + sumRight + node->val;
        }
    };
    ```
    
    ```cpp
    class Solution {
    public:
        int findTilt(TreeNode* root) {
            int ans = 0;
            dfs(root, ans);
            return ans;
        }
    
        int dfs(TreeNode* node, int& ans) {
            if (node == nullptr) {
                return 0;
            }
            int sumLeft = dfs(node->left, ans);
            int sumRight = dfs(node->right, ans);
            ans += abs(sumLeft - sumRight);
            return sumLeft + sumRight + node->val;
        }
    };
    ```
    
- Go通过增加返回值，或者使用指针
  
    在`leetcode563v1`中令遍历函数`traverse`每次都返回两个值，`sum`和`lValue+rValue+root.Val`分别用于自底向上遍历树时，对坡度与节点值之和进行累记。
    
    ```go
    // leetcode563v1
    func FindTilt(root *utils.TreeNode) int {
        var traverse func(root *utils.TreeNode) (int, int)
        traverse = func(root *utils.TreeNode) (int, int) {
            sum := 0
            if root == nil {
                return 0, 0
            }
            sumL, lValue := traverse(root.Left)
            sumR, rValue := traverse(root.Right)
            sum = abs(rValue - lValue) + sumL + sumR
            return sum, lValue + rValue + root.Val
        }
        res, _ := traverse(root)
        return res
    }
    ```
    
    此外，也可以使用传入`int`指针的方式解决。
    
    ```go
    // leetcode563v2
    func FindTilt(root *utils.TreeNode) int {
        var traverse func(root *utils.TreeNode, sum *int) int
        sum := 0
        traverse = func(root *utils.TreeNode, sum *int) int {
            if root == nil {
                return 0
            }
            lValue := traverse(root.Left, sum)
            rValue := traverse(root.Right, sum)
            (*sum) += abs_int(rValue - lValue)
            return lValue + rValue + root.Val
        }
        traverse(root, &sum)
        return sum
    }
    ```
    

Go语言提供了一个非常优秀的特性，**匿名函数可以捕获外部环境中的自由变量**，并且**具有记忆效应**。

一个匿名函数与外部环境相组合，便形成了**闭包（Closure）**。简单地，**闭包可以理解为一个引用了外部变量的匿名函数**。

> 闭包应该也是一个函数式编程中的概念，在此不做过多解释
> 

### **捕获外部变量**

可以注意到`Accumulator`函数返回的是一个类型为`func() int`的函数值， 所以使用`return`返回一个类型符合的匿名函数。但`startValue`声明的位置在匿名函数外，而匿名函数中的逻辑却尝试对`startValue`进行了修改， 并且能够成功编译运行。

```go
package main

import "fmt"

func Accumulator() func() int {
    startValue := 1
    fmt.Printf("The address of startValue is %x\n", &startValue)
    return func() int {
        fmt.Printf("In annoymous function, address is %x\n", &startValue)
        startValue++
        return startValue
    }
}

func main() {
    accumulator := Accumulator()
    fmt.Println(accumulator())
    accumulator()
    accumulator()
    accumulator()
    fmt.Println(accumulator())

    newAccumulator := Accumulator()
    fmt.Println(newAccumulator())
}
```

该代码能够成功运行的原因在于匿名函数可以捕获了外部环境中变量的特性。捕获之后，便可以将其视为一般的变量进行操作与修改，并且**所有的修改操作均会反应在外部环境中**。

比如，匿名函数捕获了`sum`变量，然后将匿名函数本身的函数值赋值给`traverse`。在递归遍历的过程中，不断对`sum`进行修改，但在匿名函数外访问`sum`时，值不再是原来的`0`。

```go
// leetcode563v3
func FindTilt(root *utils.TreeNode) int {
    var traverse func(root *utils.TreeNode) int
    sum := 0
    traverse = func(root *utils.TreeNode) int {
        if root == nil {
            return 0
        }
        lValue := traverse(root.Left)
        rValue := traverse(root.Right)
        sum += abs(rValue - lValue)
        return lValue + rValue + root.Val
    }
    traverse(root)
    return sum
}
```

### **记忆效应**

被引用的自由变量和匿名函数一同存在，即使已经离开了自由变量的环境也不会被释放或者删除，在闭包中可以继续使用这个自由变量。

被捕获到闭包中的变量让闭包本身拥有了记忆效应，闭包中的逻辑可以修改闭包捕获的变量，变量会跟随闭包生命周期一直存在，闭包本身就如同变量一样拥有了记忆效应。

```go
package main

import "fmt"

func generator() func() int {
    value := 0
    return func() int {
        value++
        fmt.Println(value)
        return value
    }
}

func main() {
    f := generator()
    f()
    f()
    f()
    f = generator()
    f()
}
//1
//2
//3
//1
```

这里`generator`函数返回的是函数`f`捕获了`value`变量，并且`f`在`main`函数中执行，已经离开了原来的环境，但是还能正常运行，且随着`f`的生命周期一直存在。

### 记忆效应的 **底层原理**

> 变量逃逸分析
> 
> 
> [https://blog.csdn.net/weixin_42117918/article/details/90409744](https://blog.csdn.net/weixin_42117918/article/details/90409744)
> 

简而言之，Go编译器检测到局部变量仍有指针指向访问，原本应该分配在**栈**内的局部变量放在**堆**中，这种情况称之为变量逃逸。

[ch33 垃圾回收与变量逃逸分析](ch33%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E4%B8%8E%E5%8F%98%E9%87%8F%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90%20860c699cfcf840f090738a8815ecfe8a.md)

`go run -gcflags "-m -l" main.go`
使用 `go run` 运行程序时，`-gcflags` 参数是编译参数。其中 `-m` 表示进行内存分配分析，`-l`表示避免程序内联，也就是避免进行程序优化。

```go
package main

import "fmt"

func Accumulator() func() int {
    startValue := 1
    fmt.Printf("The address of startValue is %x\n", &startValue)
    return func() int {
        fmt.Printf("In annoymous function, address is %x\n", &startValue)
        startValue++
        return startValue
    }
}

func main() {
    accumulator := Accumulator()
    fmt.Println(accumulator())
    accumulator()
    accumulator()
    accumulator()
    fmt.Println(accumulator())

    newAccumulator := Accumulator()
    fmt.Println(newAccumulator())
}
```

```
go run -gcflags "-m -l" main.go
# command-line-arguments
.\main.go:6:2: moved to heap: startValue
.\main.go:7:12: ... argument does not escape
.\main.go:8:9: func literal escapes to heap
.\main.go:9:13: ... argument does not escape
.\main.go:17:13: ... argument does not escape
.\main.go:17:25: accumulator() escapes to heap
.\main.go:21:13: ... argument does not escape
.\main.go:21:25: accumulator() escapes to heap
.\main.go:24:13: ... argument does not escape
.\main.go:24:28: newAccumulator() escapes to heap
The address of startValue is c0000aa058
In annoymous function, address is c0000aa058
2
In annoymous function, address is c0000aa058
In annoymous function, address is c0000aa058
In annoymous function, address is c0000aa058
In annoymous function, address is c0000aa058
6
The address of startValue is c0000aa088
In annoymous function, address is c0000aa088
2
```
