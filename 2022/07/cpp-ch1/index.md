# 第一章 概述


## **第一章 概述**

### 1.1 名字空间

C++标准库中的类和函数是在名字空间std中声明的。名字空间可以消除那些因重名而导致的命名冲突

{{< admonition info "💡允许使用没有名字的名字空间">}}

由于名字空间没有名字，因此无法在其他空间中引用。

无名名字空间中的变量不必声明，可以理解为一个全局变量，再次声明相同变量名后会被覆盖。

无名名字空间内的成员的作用域为**本文件**从**声明**到**文件结束。**

{{< /admonition>}}

```cpp
#include <iostream>
using namespace std;
namespace ns1 {
    int inflag = 1;
}
namespace ns2 {
    int inflag = 2;
}
namespace {
    int inflag = 0;
}
int main() {
    cout << ns1::inflag << endl;
    cout << ns2::inflag << endl2;
    cout << inflag << endl;
    cout << "****" << endl;

    ns1::inflag = 2;
    ns2::inflag = 3;
    inflag = 4;
    cout << ns1::inflag << endl;
    cout << ns2::inflag << endl2;
    cout << inflag << endl;
    cout << "****" << endl;

    int inflag = 1;
    cout << inflag << endl;
    return 0;
}
// 1
// 2
// 0
// ****
// 2
// 3
// 4
// ****
// 1
```

### 1.2 定义常量

格式：

```cpp
const int MAXLine = 1000
```

注意：**定义时必须初始化**，**且不可再次赋值**

{{< admonition info "💡宏定义与常量的区别">}}


const常量有数据类型，而宏常量没有数据类型

宏常量有时可能会导致某些错误，例如

#define x a+b   若在程序中写了x*4此类表达，则会出错

{{< /admonition>}}

### 1.3 函数重载

- 函数的参数个数不同
- 函数的参数类型不同
- 函数必须同名
- **const关键字可以用于区分重载函数**（见后）

### 1.4 函数模板

函数模板不是一个函数，编译系统不产生任何执行代码，当编译程序发现与函数模板中相匹配的函数调用时，生成一个重载函数。

该重载函数称为模板函数，是函数模板的一个实例，只处理一种唯一的数据类型。

```cpp
template <class T1, class T2, class T3>
T1 max(T2 a, T3 b){
    return (a>b)?(T1)a:(T2)b;
}
```

### 1.5 带默认参数的函数

- 实参与形参的结合顺序，从左到右，**故带默认参数的参数必须在最右边**

- 注意与重载函数共同使用时可能会出现二义性问题
- ~~函数声明中给形参指定默认值，又在函数定义中给形参指定默认值，有些编译器会报错，有些会通过（以第一次为准），一般在函数声明中赋值（老师上课所讲，了解即可）~~

### 1.6 引用

{{< admonition info "💡引用的核心">}}

引用的核心就是一个地址两个名字。

{{< /admonition>}}

- **引用时必须初始化**
- 初始化值不能是一个常数
- 一旦被声明就不能指向其他变量
- 引用是变量的另一个名字，可以对引用进行赋值，取址等操作
- 以下的声明是非法的

```cpp
void& test1 = max; // 不允许使用对void的引用
int& a[6] = ...    // 不允许使用引用的数组
int&* p = max;     // 不允许使用指向引用的指针
```

{{< admonition info "💡指向引用的指针和指向指针的引用">}}

指针需要指向内存空间，而引用只是一个别名而已，不是内存空间。

引用不是一个对象，所以就不会存在指向引用的指针，但指针是对象，所以存在指向指针的引用。

int&* p   指向引用的指针 不合法

int*& p   指向指针的引用 合法

{{< /admonition>}}

```cpp
#include <iostream>
using namespace std;

int main() {
    //引用的定义,可以是变量名也可以是另一个引用
    int max = 1;
    int* pmax = &max;
    int nums[6];
    int& refmax = max;
    int& reffmax = refmax;

    cout << max << endl;
    refmax++;
    cout << max << endl;
    reffmax++;
    cout << max << endl;
    //三者全部指向max,皆可对该内存单元进行操作

    // void& test1 = max; // 不允许使用对void的引用
    // int& a[6] = ...    // 不允许使用引用的数组
    // int&* p = max;     // 不允许使用指向引用的指针 
    int*& p = pmax;
    cout << (*p) << " " << p << endl;
}
```

### 1.7 const与常引用

const是用作定义常量，一旦某个对象定义为一个常量，那么就不能改变这个对象的值。当const与引用相结合使用时，称为常引用。

- 常引用不能修改值
- 常引用可以与常量，变量（或字面值、表达式）绑定
- 如下代码所示，普通引用不能与常量绑定

```cpp
const int a = 20;
// int& ref = a; // 将int & 类型的引用绑定到const int类型的初始值设定项时，限定符被丢弃
const int& ref = a;
```

{{< admonition info "💡常引用的作用">}}

用这种方式声明的引用，不能**通过引用**对目标变量的值进行修改，达到了引用的安全性。

一旦完成声明，该引用的值就不再改变，即使被引用的变量已经发生了变化。**

用途：**安全地进行函数传参**

{{< /admonition>}}

```cpp
#include <iostream>
using namespace std;

int main() {
    const int a = 20;
		// 将int& 类型的引用绑定到const int类型的初始值设定项时
		//限定符被丢弃
    // int& ref = a; 
    const int& ref1 = a;

    int x = 5, y = 4;
    const int& ref2 = (x+y);
    cout << ref2 << endl;
    y++;
    cout << ref2 << endl;
    // ref2++; // 表示式必须是可被修改的左值。
    return 0;
}
// 9 
// 9
// 10
// 9
```

### 1.8 函数与引用

- C++把变量的引用作为函数形参，即传送变量的别名，具体略
- 函数的返回值为某一变量的引用，因此可对其返回值进行赋值操作
  - **一般来说，函数不能作为左值**，但如果将**函数返回引用类型**则可对其进行赋值
  - 若要返回某个变量的引用，该变量必须存储在静态区，如全局变量或静态局部变量

注意：**先调用，再赋值**

```cpp
#include <iostream>
using namespace std;

int x = 20;
int& add() {
    cout << "in add before inc x: " << x << endl;
    x++;
    cout << "in add after inc x: " << x << endl;
    return x; 
}

int main() {
    cout << add() << endl;
    add() = -20;
    cout << add() << endl;
    return 0;
}
// in add before inc x: 20
// in add after inc x: 21
// 21
// in add before inc x: 21
// in add after inc x: 22
// in add before inc x: -20
// in add after inc x: -19
// -19
```



### 1.9 内联函数

编译时，将调用函数代码嵌入到主函数中，以**空间效率换取时间效率**

定义格式：

```cpp
inline <函数值类型> <函数名>(参数表){

}
```

- inline 关键字必须与函数定义体放在一起
- 内联函数必须充分简单，不含循环递归
- 一个好的编译器，会根据函数的函数体，自动取消不值得的内联
- 类内定义的成员函数会自动视为内联函数

### 1.10 字符串变量

字符串常量以**\0**结束，但将字符串常量存放到字符串变量中时，只存放字符串本身而不包括'\0'

```cpp
#include <iostream>
#include <string>
#include <cstring>
using namespace std;

int main() {
    string str1, str2, str3;
    str1 = "China";
    str2 = str1;
    str3 = str1 + str2;
    cout << strlen("China") << endl;
    cout << sizeof("China") << endl;
    cout << str1.size() << endl;
}
// 5
// 6
// 5
```

### 1.11 new/delete

new操作从**堆**中分配一块与<类型>相适应的存储空间，而用int,float之类的申请的内存在**栈**中

```cpp
<指针变量名> = new <类型>
<指针变量名> = new <类型>(<初值>)
<指针变量名> = new <类型>[<元素个数>]

delete <指针变量名>
delete[] <指针变量名>
```

```cpp
#include <iostream>
using namespace std;

int main() {
    int* p = new int;
    cout << p << " " << *p << endl;
    int* q = new int(0);
    cout << q << " " << *q << endl;
    int* num = new int[20];
    num[0] = 1; num[1] = 20;
    cout << num[0] << " " << num[1] << endl;
    delete(p);
    delete(q);
    delete(num);
    return 0;
}
// 0x1071780 17242864
// 0x10717a0 0
// 1 20
```
