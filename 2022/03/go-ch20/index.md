# Ch20 接口


# ch20 map

[ch38 map底层实现](ch38%20map%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0%20fc416a402c864cfa896a986082fc798e.md)

写在前面，map并不是并发安全的，如果需要在goroutine中使用，需要使用`sync.map`，更多的信息请参考

[ch25 并发编程](ch25%20%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%20f0cdfb2499f74bc48b4fc5ece4619ee1.md)

## 定义与初始化

先定义后初始化

```go
// 定义map
// var mp map[KeyType]ValueType
var mp map[string]int

// 在未初始化时,mp无法使用，否则会宕机
mp = make(map[string]int)
// 或者用如下方式
mp = map[string]int{}
```

定义并初始化

```go
mp1 := make(map[string]int)
mp2 := map[string]int{}
mp3 := map[string]int{
		"apple":2,
		"banana":3,
}
```

## 根据key查询value

如果key不存在时，会返回value对应类型的默认值，但不能区分返回的”零值”是真实数据还是默认值，所以常需要按照如下的方式进行查询

```go
func main() {
		scores := make(map[string][]int)
		name := "Lily"
		n := 5
		if val, has := scores[name]; !has {
				// 不存在Lily的成绩记录，所以首先创建一个[]int
				scores[name] = make([]int, n)
		}
		// 如果直接执行下面这两行会因为scores[name]对应的是一个nil指针，而触发宕机错误
		scores[name][0] = 90
		scores[name][1] = 94
		... 
}
```

## 遍历map

```go
for k, v := range mp {
		fmt.Println(k, v)
}

for _, v := range mp {
		fmt.Println(v)
}

for k := range mp {
		fmt.Println(k)
}

```

```go
package main

import "fmt"

func main() {
		mp := map[string]int{"Lily": 90, "Tom": 94}
		// 注意for range val是一个拷贝
		for _, val := range mp {
				val = 20
				fmt.Println(val)
		}
		for key, val := range mp {
				fmt.Println(key, val)
		}
}
```

注：遍历输出的元素与填充顺序无关，不能期望map在遍历时返回某种期望顺序的结果。

## 删除键值对

使用delete内建函数从map中删除一组键值对，delete()函数的格式为`delete(mp，key)`

```go
package main

import "fmt"

func main() {
		scene := make(map[string]int)
		scene["route"] = 66
		scene["brazil"] = 4
		scene["china"] = 960

		delete(scene, "brazil")
		for k, v := range scene {
				fmt.Println(k, v)
		}
}
```

## 清空map中的元素

Go语言并没有为map提供任何清空所有元素的函数、方法。

清空map的唯一办法就是重新make一个新的map。垃圾回收的效率比写一个清空函数的效率高。

## sync.Map

Go中的map在并发时，只读是线程安全的，同时读写线程不安全。

```go
package main

func main() {
		m := make(map[int]int)
		go func() {
				for {
						m[1] = 1
				}
		}()

		go func() {
				for {
						_ = m[1]
				}
		}()
		select {}
}
// 运行时输出: fatal error: concurrent map read and map write
```

Go在1.9版本中提供了一种效率较高的并发安全的sync.Map

> [https://golangforall.com/en/post/sync-map.html](https://golangforall.com/en/post/sync-map.html)
> 
- 无需初始化，直接声明即可
- sync.Map不能使用map的方式进行取值和设置等操作，而是使用sync.Map的方法进行调用。**Store表示存储**，**Load表示获取**，**Delete表示删除**。
- 使用**Range**配合一个回调函数进行遍历操作，通过回调函数返回内部遍历出来的值。Range参数中的回调函数的返回值功能是：需要继续迭代遍历时，返回true，终止迭代遍历时，返回false。

```go
package main

import (
		"fmt"
		"sync"
)

func main() {
		var scene sync.Map

		scene.Store("greece", 97)
		scene.Store("london", 100)
		scene.Store("egypt", 200)
		fmt.Println(scene.Load("london"))

		scene.Delete("london")

		scene.Range(func(key interface{}, value interface{}) bool {
				fmt.Println("iterate:", key, value)
				return true
		})
}
```
