# Ch12 切片


# ch12 切片

切片是一种数据结构，围绕动态数组的概念设计的，可以按需自动改变大小，使用这种结构，可以更方便地管理和使用数据集合。

## **内部实现**

切片是对数组一个连续片段的引用，该数组称之为相关数组，通常都是匿名的，所以说切片是一个引用类型。由于底层的内存是连续分配的，所以效率非常高。此外，切片还拥有可以通过索引获得数据，可以迭代以及垃圾回收的好处。

切片对象非常小，是一个只有三字段的数据结构，包括指针，切片长度，切片容量。

```go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```

![0012-1.png](ch12%20%E5%88%87%E7%89%87%20f180658defee43da8e84d4b2b815cf76/0012-1.png)

## **声明和初始化**

### **使用make函数创建**

使用内置的make函数，第一个参数用于指定切片长度，第二参数用于指定切片容量。如果不指定容量，那么容量默认与长度相同。

```go
slice1 := make([]int, 5)
slice2 ：= make([]int, 5, 10)
```

在`slice2`中只能访问5个元素，因为切片长度为5，剩下5个元素，需要切片扩充后才能访问。

### **使用字面量创建**

```go
slice3 := []int{1, 2, 3, 4, 5}
slice4 := []int{4:1}
```

### **基于原数组或切片创建**

```go
slice5 := []int{1, 2, 3, 4, 5}
slice6 := slice5[:]
slice7 := slice5[0:]
slice8 := slice5[:5]
```

- `[i:j]`是从`i`开始，但不包括`j`
- `i`省略则默认0，`j`省略则默认为长度，`i,j` 都应该在范围内， `i>=0, j <=size`
- 新切片和旧切片共享同一个底层数组，一荣俱荣，一损俱损

```go
func testSlice() {
    var originArr [5]int
    slice1 := originArr[:]
    slice2 := originArr[:2]
    originArr[2] = 4
    fmt.Println(slice1)
    fmt.Println(slice2)
    slice2[1] = 5
    fmt.Println(slice1)
    fmt.Println(slice2)
    slice3 := []int{1, 2, 3, 4, 5}[:]
    slice4 := []int{1, 2, 3, 4, 5}[:]
    slice4[3] = 9
    fmt.Println(slice3)
    fmt.Println(slice4)
    //slice1,2共享底层数组，但slice3,4不共享底层数组
}
```

## **切片长度、容量**

- 切片长度：`len(slice)`
- 切片容量：`cap(slice)`
- 将切片s的长度拓展到容量上限：`s = s[:cap(s)]`
- `make([]int, length, capactiy)`, `s[start:end:capacityend]`可操作修改容量，具体地见如下代码

```
package main

import (
    "fmt"
)

func main() {
    // capacityend needs to be less than or equal to the basical capacity
    slice1 := make([]int, 5, 10)
    slice2 := slice1[:6]
    slice3 := slice1[:6:8]   // 8 <= cap(slice1)
    slice4 := slice1[4:5:10] // 10 <= cap(slice1)
    slice5 := slice2[1:]
    slice6 := slice3[2:]
    slice7 := slice3[2::6]   // 6 <= cap(slice3)
    fmt.Printf("capacity of slice1 is %d\n", cap(slice1)) //10
    fmt.Printf("capacity of slice2 is %d\n", cap(slice2)) //10
    fmt.Printf("capacity of slice3 is %d\n", cap(slice3)) //8 = 8-0
    fmt.Printf("capacity of slice4 is %d\n", cap(slice4)) //6 = 10-4
    fmt.Printf("capacity of slice5 is %d\n", cap(slice5)) //9 = cap(slice2) - 1
    fmt.Printf("capacity of slice6 is %d\n", cap(slice6)) //6 = cap(slice3) - 2
    fmt.Printf("capacity of slice7 is %d\n", cap(slice7)) //4 = 6- 2
}
```

## **特殊切片**

nil切片和空切片长度和容量都是0，但nil切片指向底层数组的指针是nil，而空切片对应的是一个地址。

```go
package main

import (
    "fmt"
)

func main() {
    var nilSlice []int
    var empytSlice = []int{}
    fmt.Println(len(nilSlice), cap(nilSlice))
    fmt.Println(len(empytSlice), cap(empytSlice))
}
// 0 0
// 0 0
```

## **扩充切片**

### **在容量范围内，修改切片长度**

切片只能访问到其长度内的元素，访问超过元素外的元素，会导致运行时异常。

```go
package main

import "fmt"

func main() {
    slice1 := make([]int, 0, 10)
    fmt.Printf("slice1:%d %d\n", len(slice1), cap(slice1))
    slice2 := slice1[5:7]
    fmt.Printf("slice2:%d %d\n", len(slice2), cap(slice2))
    slice3 := slice2[:len(slice2)+1]
    fmt.Printf("slice3:%d %d\n", len(slice3), cap(slice3))
    slice4 := slice2[:cap(slice2)]
    fmt.Printf("slice2:%d %d\n", len(slice4), cap(slice4))
}

// slice1:0 10
// slice2:2 5
// slice3:3 5
// slice2:5 5
```

> 注意：不同向前移动，只能向下标增大的方向获取底层数组剩下的元素
> 

### **增加切片的容量**

```go
package main

import (
    "fmt"
)

func main() {
    nums1 := make([]int, 5)
    fmt.Println(cap(nums1), len(nums1))
    fmt.Printf("the address of nums in modify is %p\n", &nums1)
    nums1 = append(nums1, 6)
    fmt.Println(cap(nums1), len(nums1))
    fmt.Printf("the address of nums in modify is %p\n", &nums1)

    nums2 := make([]int, 5, 10)
    for i := range nums2 {
        nums2[i] = i + 1
    }
    fmt.Println(cap(nums2), len(nums2))
    fmt.Printf("the address of nums in modify is %p\n", &nums2)
    nums2 = append(nums2, 6)
    fmt.Println(cap(nums2), len(nums2))
    fmt.Printf("the address of nums in modify is %p\n", &nums2)
}

//5 5
//the address of nums in modify is 0xc000004078
//10 6
//the address of nums in modify is 0xc000004078
//10 5
//the address of nums in modify is 0xc000004090
//10 6
//the address of nums in modify is 0xc000004090
```

首先看这份代码，可以看到`nums1`长度为5，容量为5，没有位置去加入元素6，所以使用`append`函数后，扩充了容量。而对于`nums2`长度为5，容量为10，只需增加长度即可。此外，`nums1`和`nums2`的地址都没有变，说明它们还是原来那个切片，但`nums1`指向的底层数组已经发生改变，而`nums2`没有。

append函数会智能的增长底层数组的容量，目前的算法是：容量小于1000个时，总是成倍的增长，一旦容量超过1000个，增长因子设为1.25。若切片底层数组还有容量，则在原数组上操作；若切片底层数组没有足够的容量时，就会创建一个新底层数组，把原来数组的值复制到新底层数组里，再追加新值，**而且这样不会影响原来的底层数组**，实例如下：

```go
package main

import "fmt"

func main() {
    slice1 := make([]int, 5)
    slice2 := slice1
    slice1[0] = 1
    fmt.Println(slice1, slice2)
    slice2 = append(slice2, 6)
    fmt.Println(slice1, slice2)
    slice1[2] = 3
    fmt.Println(slice1, slice2)
}
// [1 0 0 0 0] [1 0 0 0 0]
// [1 0 0 0 0] [1 0 0 0 0 6]
// [1 0 3 0 0] [1 0 0 0 0 6]
```

因此，一般在创建新切片的时候，最好要让新切片的长度和容量一样，这样在追加操作的时候就会生成新的底层数组，和原有数组分离，就不会因为共用底层数组而引起奇怪问题。因为共用数组的时候修改内容，会影响多个切片。

或者直接使用copy函数进行深拷贝，这样就不会共享同一个底层数组。

## **append操作**

内置的append 也是一个可变参数的函数，所以可以同时追加好几个值，也可以通过`...`操作符，把一个切片追加到另一个切片里面。

```go
a = append(a, x, y, z)                    //末尾添加元素x y z
x, a = a[len(a)-1], a[:len(a)-1]          //去除切片a最末尾的元素x
a = append(a, b...)                       //将切片b中的所有元素加入a
a = append(a[:i], a[j:]...)               //删除切片a中i,i+1..j-1的元素
a = append(a[:i], append(b, a[i:]...)...) //索引i的位置插入切片b的所有元素
a = append(a, make([]T, j)...)            //为切片a拓展j个元素长度
```

## **切片的使用**

### **切片迭代**

```go
package main

import (
    "fmt"
)

func main() {
    cities := []string{"shanghai", "beijing", "shaanxi"}
    for _, city := range cities {
        fmt.Println(city)
    }

    for i := 0; i < len(cities); i++ {
        fmt.Println(cities[i])
    }
}
```

### **切片作为函数参数**

值传递与引用传递：[https://zhuanlan.zhihu.com/p/366908019](https://zhuanlan.zhihu.com/p/366908019)

切片作为参数，在函数间以传递的方式传递，占用内存非常小。在传递赋值切片时，底层数组不会被赋值，不受影响。

```go
package main

import (
    "fmt"
)

func main() {
    nums := []int{1, 2, 3, 4, 5}
    fmt.Printf("the address of nums in main is %p\n", &nums)
    modify(nums)
    fmt.Println(nums)
}

func modify(nums []int) {
    fmt.Printf("the address of nums in modify is %p\n", &nums)
    nums[0] = 10
}
// the address of nums in main is 0xc000004078
// the address of nums in modify is 0xc000004090
// [10 2 3 4 5]
```

可以发现这两个切片地址不一样，这是因为两个num保存的是切片的地址，对nums取地址得到的是nums的地址，而不是底层数组。在修改一个索引的值后，原切片的值也被修改了，说明其共用一个底层数组。

### **切片拷贝**

正如在切片作为函数参数中所说，切片作为参数时是值传递的。将切片赋值给另一个切片时，它们之间是共享底层数组的，实例如下：

```go
package main

import "fmt"

func main() {
    a := []int{1, 2, 3}
    b := a
    fmt.Println(a, b)
    b[0] = 22
    fmt.Println(a, b)
}
// [1 2 3 ] [1 2 3]
// [22 2 3] [22 2 3]
```

如果想要互不影响的话，可以使用`copy`函数

`func copy(dst, src []T) int` ：copy方法将类型为T的切片从源地址src拷贝到目标地址dst，覆盖dst的相关元素，并且返回拷贝的元素的个数，其中拷贝个数是src和dst长度的最小值。

```go
package main

import "fmt"

func main() {
    a := []int{1, 2, 3}
    b := make([]int, len(a))
    copy(b, a)
    a[0] = -2
    b[0] = 2
    fmt.Println(a, b)
}
```

## **垃圾回收**

切片的底层指向一个数组，但该数组的实际容量可能要大于切片所定义的容量。

只有在没有任何切片指向的时候，底层的数组内存才会被释放，这种特性有时会导致程序占用多余的内存。

示例，函数FindDigits将一个文件加载到内存，然后搜索正则表达式的第一个匹配项（这里是数字）并返回一个切片。

```go
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte{
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
```

这段代码可以顺利运行，但返回的`[]byte`指向的底层是整个文件的数据。只要该切片不被释放，垃圾回收器就不能释放整个文件所占用的内存。换句话说，这一点点有用的数据却占用了整个文件的内存。

修改后的代码如下：

```go
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte{
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    c := make([]byte, len(b))
    copy(c, b)
    return c
}
```

## **切片练习**

### **增加切片容量的函数**

```go
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    if n > cap(slice) {
        newSlice := make([]byte, (n + 1) * 2)
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:n]
    copy(slice[m:n], data)
    return slice
}
```

### **写一个函数InsertStringSlice将切片插入到另一个切片的指定位置**

```go
func main() {
    s3 := make([]byte, 5, 15)
    s3[0], s3[1], s3[2], s3[3], s3[4] = 'h', 'e', 'l', 'l', 'o's4 := []byte("test")
    ans, _ := insertStringSlice(s3, s4, 2)
    fmt.Println(string(ans))
    s1 := []byte("helloworld")
    s2 := []byte("testing")
    ans2, _ := insertStringSlice(s1, s2, 2)
    fmt.Println(string(ans2))
}

func insertStringSlice(dst, data []byte, pos int) ([]byte, bool) {
    if pos < 0 || pos > len(dst) {
        return nil, false
    }
    err := true
    n := len(dst) + len(data)
    if n <= cap(dst) {
        dst = dst[:n]
    } else {
        newSlice := make([]byte, n)
        copy(newSlice, dst)
        dst = newSlice
    }
    copy(dst[pos+len(data):], dst[pos:])
    copy(dst[pos:pos+len(data)], data)
    return dst, err
}
//hetestllo
//heftestinglloworld
```

### **写一个函数RemoveStringSlice将从start到end索引的元素从切片中移除**

```go
func main() {
    nums := []int{1, 2, 3, 4, 5, 6, 7}
    fmt.Println(removeStringSlice(nums, 3, 6))
}

func removeStringSlice(nums []int, start, end int) ([]int, bool) {
    if start < 0 || end >= len(nums) {
        return nil, false
    }
    copy(nums[start:], nums[end+1:])
    n := len(nums)
    nums = nums[:n-(end-start+1)]
    return nums, true
}
// [1, 2, 3] true
```
