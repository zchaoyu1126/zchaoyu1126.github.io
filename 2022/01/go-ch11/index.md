# Ch11 数组


- 数组是同一种数据类型元素的集合
- 数组是切片、Map等数据结构的基础
- 在Go语言中，数组从声明时就已经确定数组中元素的类型和个数，在程序运行期间不可变
- 数组的最大长度是2GB

## 1 数组定义

```go
var identifier [len]type
```

例如，

```go
var arr1 [3]int
var arr2 [5]int
```

数组的长度必须是常量，并且长度是数组类型的一部分，一旦确定，不能改变。

此外，`[3]int`和`[5]int`是不同的类型。

## 2 数组初始化

```go
var arr1 [3]int                                        //[0,0,0]
var arr2 [3]int{1,2}                                   //[1,2,0]
var arr3 [3]string{"beijing", "shanghai", "shenzhen"}  //[北京,上海,深圳]
var arr4 = [...]int{1,2}                               //[1,2]
var arr5 = [...]int{1:1, 3:5}                          //[0,1,0.5]
var arr6 = [3][2]string{
    {"北京", "上海"},
    {"广州", "深圳"},
    {"成都", "重庆"},
}

var arr7 = [...][2]string{
    {"北京", "上海"},
    {"广州", "深圳"},
    {"成都", "重庆"},
}
```

- 当`[]`为空，而不是`...`时，此时的类型是slice
- 多维数组中，只有第一层可以使用`...`让编译器来推导数组长度

## 3 数组遍历

```go
for i := 0; i < len(arr1); i++ {
    fmt.Println(arr1[i])
}

for idx, value := range arrr1 {
    fmt.Println(idx, value)
}

for i := 0; i < len(arr6); i++ {
    for j := 0; j < len(arr6[i]); j++ {
        fmt.Println(arr6[i][j])
    }
    fmt.Println()
}

for _, rows := range arr6 {
    for _, city := range rows {
        fmt.Println(city)
    }
    fmt.Println()
}
```

## 4 指针数组和数组指针

指针数组和数组本身差不多，只不过元素类型是指针。

```go
package main

import "fmt"

func main() {
    array := [5]*int{1:new(int), 3:new(int)}
    // 至此只能给下标位1,3的解引用，对nil解引用会使得系统崩溃
		// 不能对空指针解引用
    for idx, p := range array {
        if p != nil {
            fmt.Println(idx, *p)
        } else {
            fmt.Println("nil")
        }
    }

    *array[1] = 1
    array[0] = new(int)
    *array[0] = 2

    fmt.Println("after......")
    for idx, p := range array {
        if p != nil {
            fmt.Println(idx, *p)
        } else {
            fmt.Println("nil")
        }
    }
}
```

数组是值类型，传参时会复制整个数组，改变副本的值而不改变本身的值。将数组作为参数传递给函数时，如果需要对数组进行修改，并将结果保留，可以使用以下两种方式

关于切片和关于函数将在[ch12 切片](http://zchaoyu1126.github.io/go-ch12.html)和[ch13 函数](http://zchaoyu1126.github.io/go-ch13.html)说明

- 使用指针

  注意这里不能用`for-range结构遍历修改数组`,具体原因参考[ch8 控制结构](http://zchaoyu1126.github.io/go-ch8.html)

```go
package main

import (
    "fmt"
)

func main() {
    array := [3]float64{7.0, 8.5, 9.1}
    fmt.Println(array)
    modifyArr(&array)
    fmt.Println(array)
}

func modifyArr(arr *[3]float64) {
    for i := 0; i < 3; i++ {
        if (*arr)[i] > 5.0 {
            (*arr)[i] = 5.0
        }
        //if arr[i] > 5.0 {
        //  arr[i] = 5.0
        //}
        //语法糖，与上面等价，隐藏了间接引用
    }
}
```

- 使用切片


```go
package main

import (
    "fmt"
)

func main() {
    array := [3]float64{7.0, 8.5, 9.1}
    fmt.Println(array)
    modifyArr(array[:])
    fmt.Println(array)
}

func modifyArr(arr []float64) {
    for i := 0; i < 3; i++ {
        if arr[i] > 5.0 {
            arr[i] = 5.0
        }
    }
}
```
