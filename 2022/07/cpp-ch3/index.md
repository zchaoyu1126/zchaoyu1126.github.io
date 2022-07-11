# 第三章 使用类和对象


### 3.1 构造函数和析构函数

调用时机：对象被创建和消亡时分别执行构造函数和析构函数

调用顺序：构造函数与声明对象的顺序相同，析构函数相反

**构造函数**

- 如果在类中没有显示定义构造函数，编译系统会自动生成一个**默认的无参构造函数。**
- 默认的无参构造函数**仅用于创建对象**，**并不对所产生对象的成员变量进行初始化。**
- 显示定义了带参数的构造函数后，不产生默认的构造函数，此时声明类对象 A a1;会提示语句错误。
- 构造函数可以没有参数，也可以有参数，也可以由带默认值的参数，允许重载

{{< admonition info "二义性问题">}}

注意带默认参数的构造函数与重载时可能造成的二义性问题

{{< /admonition>}}

**成员变量的初始化**方法

- 成员变量在构造函数体内被初始化

- 构造函数中的参数初始化列表

**析构函数**

- 每个类都必须有一个析构函数，如果没有显示定义，编译系统会自动生成一个**默认形式的析构函数**
- 析构函数没有参数，因此不能重载

```cpp
#include <iostream>
using namespace std;

class TestObject {
public:
    string str;
    int num;
    int* pointer;
    TestObject(string, int);
    TestObject(string);
    ~TestObject();
};

TestObject::TestObject(string str, int num)  {
    cout << "Constructor begin" << endl;
    this->str = str;
    this->num = num;
    pointer = new int(0);
    cout << "Constructor end" << endl;
}

TestObject::TestObject(string str) : str(str) {
    cout << "Constructor begin" << endl;
    this->num = num;
    pointer = new int(0);
    cout << "Constructor end" << endl;
}

TestObject::~TestObject() {
    cout << "Destructor begin" << endl;
    delete(pointer);
    cout << "Destructor end" << endl;
}

int main() {
    TestObject to1 = TestObject("hi", 20);
    TestObject to2 = TestObject("haihihai");
    return 0;
}
```

### 3.2 复制构造函数

生成一个对象的副本有两种途径：**对象的赋值和复制**

{{< admonition info "赋值和复制的不同">}}


对象的赋值是对一个已经存在的对象赋值，因此必须先定义被赋值的对象，才能进行赋值。

对象的复制是从无到有地建立一个新对象，并使他与一个已有的对象完全相同。

{{< /admonition>}}

```cpp
class Dog {
public: 
    string name;
    int age;
}

int main() {
    Dog dog1, dog2;
    dog2 = dog1;     // 赋值
    Dog dog3 = dog1; // 复制 
    return 0;
}
```

复制构造函数是一种特殊的构造函数，它的功能用**一个已知的对象**来**初始化**一个被定义的同类对象，其形式如下

```cpp
<类名>::<类名>(const <类名>& <对象名>) {
	<函数体>
}
```

{{< admonition info "默认复制构造函数">}}

把已知对象的每个成员变量的值都复制到新创建的对象中，而不做其他处理，这属于浅拷贝

**当含动态分配内存的时候，注意这么写的话会有问题，应实现深拷贝。**

{{< /admonition>}}

**复制构造函数被调用的情况**：

- 对象作为形参
- 程序中需要新建立一个对象，并用另一同类对象对它初始化
- **函数返回值是类的对象，在函数调用完毕后将返回值带回函数调用处**

{{<admonition info "第三种情况与编译器有关">}}

返回值为类的对象时，不一定调用复制构造函数，具体与编译器有关


当函数的返回值为类类型时，return 语句会返回一个对象，不过为了防止局部对象被销毁，也为了防止通过返回值修改原来的局部对象，编译器并不会直接返回这个对象，而是根据这个对象先创建出一个临时对象（匿名对象），再将这个临时对象返回。而创建临时对象的过程，就是以拷贝的方式进行的，会调用拷贝构造函数。

{{< /admonition>}}

示例代码1：拷贝构造函数（深拷贝）

```cpp
// 本例中完成了深拷贝
#include <iostream>
using namespace std;

class TestObject {
public:
    string str;
    int num;
    int* pointer;
    TestObject(string, int);
    TestObject(string);
    TestObject(const TestObject& object);
    ~TestObject();
};

TestObject::TestObject(string str, int num)  {
    cout << "Constructor begin" << endl;
    this->str = str;
    this->num = num;
    pointer = new int(0);
    cout << "Constructor end" << endl;
}

TestObject::TestObject(string str) : str(str) {
    cout << "Constructor begin" << endl;
    this->num = num;
    pointer = new int(0);
    cout << "Constructor end" << endl;
}

TestObject::TestObject(const TestObject& object) {
    cout << "Copy Constructor begin" << endl;
    this->str = object.str;
    this->num = object.num;
    this->pointer = new int(*object.pointer);
    cout << "Copy Constructor end" << endl;
}

TestObject::~TestObject() {
    cout << "Destructor begin" << endl;
    delete(pointer);
    cout << "Destructor end" << endl;
}

int main() {
    TestObject to1 = TestObject("hi", 20);
    TestObject to2 = to1;
    cout << *to1.pointer << " " << *to2.pointer << endl;
    (*to1.pointer) = 20;
    cout << *to1.pointer << " " << *to2.pointer << endl;
    return 0;
}
```

示例代码2：复制构造函数被调用的时机

```cpp
#include <iostream>
using namespace std;

class A {
public:
    A() {
        cout << "Constructor" << endl;
    }
    ~A() {
        cout << "Destructor" << endl;
    }
    A(const A& a) {
        cout << "Copy Constructor" << endl;
    }
};

void testFunc1(A a) {
    cout << "testFunc execute" << endl;
}

A testFunc2() {
    A a;
    return a;
}

int main() {
    A a1;
    // A a2 = a1;          // 拷贝构造函数 输出正确
    // testFunc1(a1);      // 拷贝构造函数 输出正确
    A a2 = testFunc2();   // 拷贝构造函数 输出不正确 
		// 这里可能是将函数内的局部对象返回给了a2，并做了一定处理，使其不会被销毁。
    return 0;
}
```

### 3.3 静态成员

静态成员是C++提供的解决**同一个类的不同对象之间**数据和函数共享问题的机制。

类的静态成员分为**静态成员变量**和**静态成员函数。**

{{< admonition info "静态成员不占内存">}}

静态成员是类所有对象共享的成员，而不是某个对象的成员，它在对象中不占内存，是属于整个类的成员

{{< /admonition>}}

**类的静态成员变量**

{{< admonition info "静态数据变量的初始化">}}

静态数据变量必须在类体外进行初始化！

静态数据变量的初始化方式：**<数据类型> <类名> :: <静态成员变量名> = <值>**

{{< /admonition>}}

实例属性：一个类的所有对象具有相同的属性（**非静态成员变量**）

类属性：某个属性为整个类所有，不属于任何一个具体对象（**静态成员变量**）

**静态成员变量不随对象的建立而分配空间，也不随对象的撤销而释放。它是在程序编译时分配空间，到程序结束时才释放空间。**

**成员变量可以不通过对象访问**，在类外公有静态成员变量访问方式有如下两种

- <对象名>.<静态成员变量>
- <类名>::<静态成员变量>

**类的静态成员函数**

- 用static 关键字修饰，是类的一部分，各个对象所公有
- 静态成员函数的作用**能够为了处理静态成员变量**
- 静态成员函数**没有this指针**
- 类内声明静态成员函数时，需要加上static关键字，但类外实现时，不必加上

{{< admonition info "静态成员函数">}}

1、静态成员函数可以**直接访问该类的静态成员，但不能直接访问类中的非静态成员

2、如果静态成员函数中要使用到**非静态成员**，必须通过**参数传递的方式得到对象名**，然后可以通过对象名来访问**非静态成员**

{{< /admonition>}}

```cpp
#include <iostream>
using namespace std;

class Tc{
private:
    int a;
    static int b;
public:
    Tc(int num):a(num){}
    static void display(Tc c);
};

int Tc::b=2;

void Tc::display(Tc c){
    cout << "a=" << c.a << ", b=" << b << endl;
}

int main(){
    Tc A(2),B(4);
    Tc::display(A);
    Tc::display(B);
    return 0;
}
```

### 3.4 对象指针与动态对象

每一个对象创建之后，会在内存中分配一定内存，故可以通过对象的起始地址来访问一个对象，即**对象指针。**

```cpp
//对象指针的定义
<类名>* <对象指针名> = &<对象名>
<对象指针名> = &<对象名>

//访问方式
<对象指针名> -> <公有成员>
<*(对象指针名)>.<公有成员>

//定义与删除动态对象
<类名>* <对象指针名> = new <类名>;
<类名>* <对象指针名> = new <类名>(<初值列表>);
delete <对象指针名>

//可参考基本类型！！！基本一致
```

### 3.5 this指针

**除静态成员函数**以外，每个成员函数都有一个**this指针参数**。this指针的值是当前被调用的成员函数所在的对象的起始地址。

`this-><成员名>` 等价于 `（*this).<成员名>`。

this指针是一个**const指针**，成员函数不能对其进行赋值。

当一个对象调用成员函数时，编译系统先将对象的地址赋给this指针，然后调用成员函数存取成员变量，则会隐含地使用this指针。

### 3.6 指向对象中成员的指针

**3.6.1 指向非静态成员变量的指针**

数据类型名 *指针变量名=&对象名.成员变量名

**3.6.2 指向非静态函数成员的指针**

{{< admonition info "函数指针">}}

编译系统将函数的入口地址赋值给某函数指针时，必须在一下3方面都要匹配：①函数参数的类型和参数个数②函数的返回值类型③所属的类

{{< /admonition>}}

重温一下指向**普通函数**的指针变量的定义方法

```cpp
// 数据类型名（指针变量名）(参数列表)   
int add(int a, int b) {
		return a + b;
}
int main() {
		int (*p)(int, int) = &add;
		cout << (*p)(2,3) << endl;
		return 0;
}
```

如果要指向类中的成员函数，则需要加上作用域运算符

```cpp
// 数据类型名（类名::指针变量名）(参数列表) = &类名::成员函数名
```

**3.6.3 指向静态成员变量的指针**

数据类型名 *指针变量名=&类名::成员变量名

**3.6.4 指向静态函数成员的指针**

数据类型名（*指针变量名）(参数列表) = &类名::成员函数名

```cpp
#include <iostream>
using namespace std;

class TestObject {
public:
    int a;
    static int b;
    int doubleA();
    static int doubleB();
    TestObject();
    TestObject(int);
};

int TestObject::b = 1;

TestObject::TestObject() {
    this->a = 1;
}

TestObject::TestObject(int x) {
    this->a = x;
}

int TestObject::doubleA() {
    return this->a * 2;
}

int TestObject::doubleB() {
    return 2*b;
}

int main() {
    int* p1 = &TestObject::b;
    cout << (*p1) << endl;
    int (*fp1)() = &TestObject::doubleB;
    cout << (*fp1)() << endl;

    TestObject tc = TestObject(5);
    int* p2 = &tc.a;
    int (TestObject::*fp2)() = &TestObject::doubleA;
    cout << (*p2) << endl;
    cout << (tc.*fp2)() << endl;
}
```

### 3.7 对象的引用

为了避免通过值传递（复制构造函数完成的事宜）来传递对象，而是通过引用来传递对象。

**参数传递的是引用，没有构造函数和析构函数被调用**，节约了系统资源，提高了运行效率。

{{< admonition info "引用作为参数">}}

注意值传递不能改变原对象的值，但传递引用则可以改变，若不想改变原对象的值，则需要用常引用(const 类型& 引用名)，具体见1.7节。


{{< /admonition>}}

### 3.8 类的常成员

**非静态常成员变量**

- 初始化位置：构造函数的初始化列表中
- 任何函数中都不能对其赋值
- 在类的不同实例中该值均不能改变，**但不同实例中该值可以不同**

**静态常成员变量**

- 静态常成员变量在类外初始化

**常成员函数：**一个访问权限仅为Read-only的函数

- 常成员函数的声明 <数据类型> <函数名> (<参数表>) **const**
- const是函数类型的一部分，在声明和定义时都必须加上，但调用时不必加const
- 常成员函数可访问来**本类中所有的成员变量**，但**不能修改**
- 常成员函数不能调用该类中的**非const成员函数**，因为这会破坏常成员函数的权限**（只读）**
- **const关键字可以用于区分重载函数**

{{< admonition info "const关键字区分重载">}}

const关键字虽然可以用于区分重载函数


int testFunc(int) const

int testFunc(int)

问：在调用时怎么知道使用的是上述哪个函数？答：常对象常引用，调用时只能调用常函数。

{{< /admonition>}}

```cpp
#include <iostream>
using namespace std;

class A {
public:
    A(int, int);
    void print();
    int plusConst() const;
    // static int plusStaticConst() const; // 静态成员函数上不允许使用类型限定符
    static int plusStaticConst();
private:
    const int a, b; // 非静态常数据成员
    static const int x, y; // 静态常数据成员
};

const int A::x = 10;
const int A::y = 20;

A::A(int a, int b):a(a), b(b) {}

void A::print() {
    cout << a << ":" << b << endl;
    cout << x << ":" << y << endl;
}

int A::plusConst() const {
    return a+b;
}

int A::plusStaticConst() {
    return x+y;
}

int main() {
    A a1(100, 200), a2(-520, 340);
    a1.print();
    a2.print();
    cout << a1.plusConst() << endl;
    cout << a1.plusStaticConst() << endl;
    cout << a2.plusConst() << endl;
    cout << a2.plusStaticConst() << endl;
    return 0;
}
```

### 3.9 常对象

常对象是其**成员变量值**在对象的整个生存期间内**不能被改变**的对象

```cpp
const <类名> <对象名> (<初始化值>);
<类名> const <对象名> (<初始化值>);
```

常对象的所有成员变量在初始化之后，均不能改变，因为常对象是只读的。

常对象**只能调用常成员函数**，即**常对象的成员变量只允许被常成员函数访问。**

常对象中的成员函数不是常成员函数，除非有const关键词修饰。

如果要修改常对象中某个成员变量的值，可以将成员变量声明为**mutable**，这样就可用**常成员函数**来修改它的值。

```cpp
#include <iostream>
using namespace std;
class Test {
public:
    int a, b;
    mutable int c;
public:
    Test(int x, int y, int z) {
        a = x; b = y; c = z;
    }

    void SetA(int x) {
        a = x;
    }
    int GetA() {
        return a;
    }

    // 常函数只能用于访问对象， 所以SetB编译失败
    // void SetB(int x) const {
    //     b = x;
    // }
    int GetB() const {
        return b;
    }

    // c是mutable的
    void SetC(int x) const {
        c = x;
    }
    int GetC() const {
        return c;
    }
};

int main() {
    Test t1 = Test(10, 20, 30);
    t1.SetA(100);
    cout << t1.GetA() << endl;
    t1.SetC(100);
    cout << t1.GetC() << endl;
    
    const Test t2 = Test(1, 2, 3);
    // t2.SetA(15); // t2是常对象，只能调用常函数
    t2.SetC(15);    // 由于c是mutable的所以打破了常函数中无法写的限制
    cout << t2.GetC() << endl; 
}
```

### 3.10 const与指针

{{< admonition info "const修饰的对象!">}}

通过const的修饰对象来区分是指向对象的常指针还是指向常对象的指针

{{< /admonition>}}

**指向对象的常指针**

对象自身可以改变，但指针的值无法改变

```cpp
// **<类名> *const 指针变量名 = 对象地址**
Time t1(21,53, 24), t2(22,20, 31);
Time *const ptr1 = &t1;  // 指向对象的常指针
// ptr1 = &t2; // 不允许更改指针的值
```

**指向常对象的指针**

指针本身的值可以改变，对象的值视情况而定

```cpp
// **const <类名> *<指针变量名> = 对象地址**
const Time t3(20, 12, 45);
const Time *ptr2 = &t3; // 指针常对象的指针
// Time *ptr3 = &t3;    // 不能用普通的指针指向常对象
```

不能用普通的指针指向常对象，必须使用指向常对象的指针。

指向常对象的指针还可以指向非const型的对象，此时不能通过指针改变该对象的值。通过隐式类型转换将非const型的对象，转换为const型，此时只能调用常成员函数以及读取成员变量，而不能做修改。

```cpp
#include <iostream>
using namespace std;

class Time {
public:
    int hour, minute, second;
    Time();
    Time(int, int, int);
    void Output();
    void Output() const;
};

Time::Time() {hour=0; minute=0; second=0;}
Time::Time(int h, int m, int s):hour(h), minute(m), second(s) {}
void Time::Output() { cout << hour << " " << minute << " " << second << endl; }
void Time::Output() const { cout << hour << " " << minute << " " << second << endl; }

int main() {
    Time t1(21,53, 24), t2(22,20, 31);
    Time *const ptr1 = &t1;  // 指向对象的常指针
    ptr1->Output();
    // ptr1 = &t2; // 不允许更改指针的值
    const Time t3(20, 12, 45);
    const Time *ptr2 = &t3; // 指针常对象的指针
    Time *ptr3 = &t3;       // 不能用普通的指针指向常对象
    ptr2->Output();         // 常对象只能调用常函数，所以调用的是带有const标志的Output函数
}
```

### 3.11 对象作为成员

**初始化方式：**

- 直接在类的定义时，对成员对象进行初始化操作

{{< admonition info "顺序问题">}}

构造函数的调用顺序：先对象成员的构造函数，再本类的构造函数

{{< /admonition>}}

```cpp
#include <iostream>
using namespace std;

class A {
public:
    A() { cout << "Constructor A" << endl; }
    A(int x) { cout << "Constructor A" << " " << x << endl; }
};

class B {
public:
    B() { cout << "Constructor B" << endl; };
private:
    // A a;
    // A a = A(20);    
    int x = 15;
    A a = A(x);
};

int main() {
    B b;
}
```

- 通过传递引用，将外部的对象赋值给类中的成员对象

```cpp
#include <iostream>
using namespace std;

class A {
public:
    A() { cout << "Constructor A" << endl; }
    A(int x) { cout << "Constructor A" << " " << x << endl; }
};

class B {
public:
    B() { cout << "Constructor B" << endl; }
    B(const A& a) { cout << "Constructor B" << endl; this->a = a; }
private:
    A a;
};

int main() {
    A a(20);
    B b(a);
}
// Constructor A 20
// Constructor A
// Constructor B
```

### 3.12 对象数组

栈空间分配：<类名> <数组名> [<下标表达式>]；

堆空间分配：<类名> *<指针变量名> = new <类名>[<下标表达式>] ;

上述均调用的**无参构造函数**，构造函数的调用次数等于数组中元素个数

如果构造函数只有一个参数，在定义数组时可以直接在等号后面的大括号内提供实参，例如

```cpp
Student stus[3]={60,78,80};
```

如果构造函数有多个参数或者有默认的参数或有多个构造函数时，在定义对象数组时在大括号中分别写出构造函数并指定实参。

```cpp
Point points2[3] = {Point(2,3), Point(), Point(4,5)};
```

```cpp
#include <iostream>
using namespace std;
class Point {
public:
    int x, y;
    Point();
    Point(int x, int y);
    void Output();
};

Point::Point() {
    x = 0;
    y = 0;
}

Point::Point(int x, int y) {
    this->x = x;
    this->y = y;
}

void Point::Output() {
    cout << x << " " << y << endl;
}

int main() {
    Point points1[5];
    Point points2[3] = {Point(2, 3), Point(), Point(4, 5)};
    for(auto p : points2) {
        p.Output();
    }
    // 返回的值是一个数组首地址
    Point *points3 = new Point[3]{Point(3, 4), Point(), Point(6, 9)};
    // points3[0].Output();
    // points3[1].Output();
    // points3[2].Output();
    for(int i = 0; i < 3; i++) {
        (points3+i)->Output();
    }
    delete[] points3;
    return 0;
}
```

### 3.13 类模板

类模板是对一组仅有成员数据类型不同的类的抽象，类模板又称参数化的类

类模板是类的抽象，类是类模板的实例，由类模板经实例化而生成的具体类称之为**模板类**

```cpp
template <class T1,class T2……>
class <类模板名>
{
    类成员的声明；
};

//举例如下
template <class T>
class Compareable{
public:
    Compareable(T a, T b){ x=a; y=b; }
    T Max(){
        return (x > y) ? x : y;
    }
private:
    T x,y;
};
```

**定义模板类的对象**

```cpp
// 类模板名 <实际的类型> <对象名> [(实际参数表)]
Compareable <int> cmp(3, 7); 
// cmp为类模板Compareable经由**int类型**实例化生成实例之后的模板类的实例
```

**类模板中成员函数的具体实现**

- 可以在类内，也可以在类外，但在类外时，必须按照以下格式
- **每一个成员函数前都需要加上template <class 类型参数>**

```cpp
template <class 类型参数>
<返回值类型> <类模板名><类型参数>::<函数名>（<参数表>）{
    <函数体>
}

//eg
template <class T>
class Compareable{
public:
    Compareable(T a, T b){ x=a; y=b; }
    T Max();
private:
    T x,y;
};

template<class T>
T Compareable<T>::max() {
		return (x > y) ? x : y;
}
```

 **使用默认参数的类模板**

```cpp
template <class T=int>
class Array{
		…
}; //使用默认参数的类模板

Array<int> intob;
Array<double> doubleob;
Array<> defaultob;
```

