# 第四章 运算符重载


### 4.1 友元

友元可以是一个**全局函数**或**另一个类的成员函数**，甚至是**一个类，**分别称为**友元函数和友元类**。

友元类的所有成员函数都是友元函数，可以访问被访问类的任何成员。

友元的特性：**不可传递性、单向性、不可继承**

**友元声明以关键字friend开始，只能出现在被访问类的定义中**。具体格式如下：

```cpp
// friend  <函数值类型>  <函数名>(<参数表>);
// friend  class <类名>
```

示例代码1：全局函数作为友元函数

```cpp
#include <iostream>
using namespace std;

class A {
private:
    float x, y;
public:
    A(float a, float b) {  
        x = a; 
        y = b;
    }
    float Sum() {  
        return x + y;
    }
    friend float Sum(A& a) {
        return a.x + a.y;
    }
    friend float Diff(A& a);
};

float Diff(A& a) {
    return a.x - a.y;
}

int main() {  
    A t1(4, 5), t2(10, 20);
    cout << t1.Sum() <<endl;
    cout << Sum(t2) << endl;
    cout << Diff(t2) << endl;
    return 0;
}
```

示例代码2：另一个类的成员函数作为友元函数

遇到了一个问题类A和类B互相包含的问题，具体见[C++中两个类互相包含的问题 - 马谦的博客 (dyxmq.cn)](https://www.dyxmq.cn/program/code/c-cpp/cpp-header-file-contain-each-other.html)

```cpp
#include <iostream>
using namespace std;

class B;

class A {
public:
    void show1(const B& b);
    // void show2(const B& b);
};

// void A::show2(const B& b) {
//     cout << b.x << " " << b.y << endl;
//     // x, y 不可访问
// } 

class B {
    friend void A::show1(const B& b);
private:
    int x, y;
public:
    B() { x = 0; y = 0;}
    B(int x, int y) { this->x = x; this->y = y;}
   
};

void A::show1(const B& b) {
    cout << b.x << " " << b.y << endl;
}

int main() {
    A a;
    B b(10, 20);
    a.show1(b);
}
```

示例代码3：友元类

```cpp
// B类所有成员函数，均可通过对象名访问A类的数据成员，对于B类而言，A类是透明的
#include <iostream>
using namespace std;

const double PI = 3.1415926;
class  A {
private:
		float r;
		float h;
public:	
    A(float a,float b) {
        r = a;
        h = b;
    }
		float Getr() {
        return r;
	  }
		float Geth() {
        return h;
		}
		friend class B;//定义B类为A类的友元
};

class B {   
private:
    int number;
public:	
    B(int n = 1) {
        number = n;
    }
		void Show(const A &a) { 
        cout << PI *a.r *a.r *a.h * number << endl;
    }
    //求A类的某个对象*n的体积
};

int  main() {	
    A a1(25, 40), a2(10, 40);
		B b1(2);
		b1.Show(a1);
    b1.Show(a2);
    return 0;
}
```

{{< admonition info "友元函数">}}

友元函数近似于普通的函数，它不带有this指针，因此必须**将对象名或对象的引用**作为友元函数的参数，这样才能访问到对象的成员。

{{< /admonition>}}

### 4.2 运算符重载的本质

C++提供运算符重载机制，使得系统预定义的运算符**能够在完成用户自定义数据类型的计算。**

运算符重载本质上是一种**特殊的函数重载。**

运算符重载实现方式：

- 类的成员函数
- 友元函数
- ~~普通函数，既不是成员函数，也不是友元函数（不推荐）~~

### 4.3 无法重载的运算符及运算符重载原则

**运算符重载原则**

- 重载的功能应与原功能类似
- 不能改变原运算符的操作数个数
- 运算符的优先级不能改变
- 运算符的结合性不能改变
- 要保持原运算符的语法结构

**不能重载的运算符**

- 作用域运算符 `::`
- 成员访问运算符 `.`
- 成员指针访问运算符 `.*`
- 条件运算符 `?:`
- 长度运算符 `sizeof`

### 4.4 运算符重载注意事项

- 用成员函数实现运算符的重载时，运算符的左操作数为当前对象，并且要用到隐含的this指针。
- 运算符重载函数不能定义为静态的成员函数，因为静态的成员函数中没有this指针。
- **后置的运算符，如`a++,a—`，重载不能用this指针返回**

### 4.5 双目运算符的重载

**4.5.1 成员函数**

{{< admonition info "双目运算符以成员函数重载">}}

运算符+的重载，a3 = a1 + a2; 等价于 a3 = a1.operator+(a2);

左操作数调用右操作数，然后返回运算结果

{{< /admonition>}}

  ```cpp
  // <类名> operator <运算符>(<参数表>) {
  //     函数体	
  // }
  
  // A operator+ (A &a);
  
  //重载二元运算符举例
  #include <iostream>
  using namespace std;
  
  class A {
      int i;
      public:
      A(int a = 0) { 
          i = a;
      }
      void Show(void) {
          cout <<"i = " << i <<endl;
      }
      void Add1(A &a, A &b) {	
          i = a.i + b.i;
      }
      void Add2(A &a) {
          this->i += a.i;
      }
      A operator +(A &a) { 
          A t;
          t.i = i + a.i;
          return t;
      }
  };
  
  int main() {
      A a1(10), a2(20), a3;
      a1.Show();
      a2.Show();
      a3 = a1 + a2;	       
      a3.Add1(a1, a2);		
      a3.Show ();
      a1.Add2(a2);
      a1.Show();
      return 0;
  }
  ```

**4.5.2 友元函数**

示例代码：运算符+的重载

```cpp
// friend <函数值类型> operator <运算符>(<参数表>) {
//		<函数体>；
// }
#include <iostream>
using namespace std;

class A {
    friend A operator +(A& a, A& b);
    private:
    int i;
    public:
    A(int a = 0) { 
        i = a;
    }
    void Show(void) {
        cout << "i = " << i <<endl;
    }
};

A operator +(A& a, A& b) {
    A t;
    t.i = a.i + b.i;
    return t;
}

int main() {
    A a1(10), a2(20), a3;
    a1.Show();
    a2.Show();
    a3 = a1 + a2;	       	
    a3.Show();
    return 0;
}
```

### 4.6 单目运算符的重载

**4.6.1 类的成员函数**

{{< admonition info "单目运算符成员函数重载">}}

++为前置运算符时，它的运算符重载函数的一般格式为：\<type> operator ++( )

++为后置运算符时，它的运算符重载函数的一般格式为：\<type>  operator ++(int)

{{< /admonition>}}

示例代码：a++和++a的重载

```cpp
#include <iostream>
using namespace std;

class A {
private:
    float x, y;
public:
    A(float a = 0, float b = 0) {
        x = a; y = b;
    }
    A operator ++() {
        A t;
        t.x = ++x;
        t.y = ++y;
        return t;
    }
    A operator ++(int) {
        A t;
        t.x = x++;
        t.y = y++;
        return t;
    }
    void show() {
        cout << x << " " << y << endl;
    }
};

int main() {
    A a(2, 3), c(5.1, 9.3);
    A b = ++a;
    A d = c++;
    b.show();
    d.show();
    c.show();
    return 0;
}
```

**4.6.2 友元函数**

{{< admonition info "友元函数重载单目运算符">}}

++为前置运算符时，它的运算符重载函数的一般格式为：A operator ++(A &a)

++为后置运算符时，它的运算符重载函数的一般格式为：A operator ++(A &a, int）

{{< /admonition>}}

示例代码：a++和++a的重载

```cpp
#include <iostream>
using namespace std;

class A {
private:
    int i;
public:
    A(int a = 0) {
        i=a;
    }
    void Show(void) {
        cout << "i = " << i <<endl;	
    }
    friend A operator++(A &a) { 
        a.i++;
        return a;
    }
    friend A operator++(A &a, int n) {
        A t;
        t.i = a.i;
        a.i++;
        return t;
    }
};

int main() {	
    A a1(10), a2, a3(20), a4;
    a2 = ++a1;
    a4 = a3++;
    a1.Show();
    a2.Show();
    a3.Show();
    a4.Show();
    return 0;
}
```

### 4.7 利用友元函数重载cin/cout

在C++中允许用户重载运算符“<<”和“>>”，实现对象的输入和输出。

重载这两个运算符时，**在对象所在的类中**，将重载这两个运算符的函数说明为该类的友元函数。

{{< admonition info "重载cin和cout" >}}

friend  istream &operator >> (istream&, ClassName&);

friend  ostream &operator << (ostream&, ClassName&);

cin >> a; 等价于operator >> (cin,a);

返回值类型为类istream的引用，目的是为了cin中可以连续使用运算符 >>

{{< /admonition>}}

```cpp
#include <iostream>
using namespace std;

class incount{
private:
    int c1,c2;
public:	
    incount(int a = 0, int b = 0) {	
        c1=a;
        c2=b;
    }
		void show()	{
        cout << "c1 = " <<c1 << '\t' << "c2 = " << c2 << endl;
    }
		friend istream &operator>>(istream&, incount&);
		friend ostream &operator<<(ostream&, incount&);
};

istream &operator>>(istream &is, incount &cc) {
    is >> cc.c1 >> cc.c2;	
    return is;
}
ostream &operator<<(ostream &os, incount &cc) {
    os << "c1 = " << cc.c1 << '\t' << "c2 = " << cc.c2 << endl;
    return os;
}

int main() {
		incount x1,x2;
		cout << x1 << x2;
		cin >> x1 >> x2;
		cout << x1 << x2;
    return 0;
}
```

### 4.8 两种运算符重载方式的选择

一般来说，单目运算符最好被重载为成员函数；对双目运算符最好被重载为友元函数。

### 4.9 类型转换运算符重载

C++引入一种特殊的成员函数——类型转换函数。

类型转换函数实际上就是一个类型间转换运算符重载函数，**它只能被重载为成员函数。**

{{< admonition info "类型转换重载">}}

<类1>::operator <类2> () {	
        return 类1实例;
}

{{< /admonition>}}

```cpp
#include <iostream>
using namespace std;

class IntComplex {
public:
    int Real, Imag;
};

class FloatComplex {
public:
    double Real, Imag;
    FloatComplex(double real = 0, double imag = 0) {	
        Real = real;
        Imag = imag;
    }
    operator IntComplex(); // 成员函数，定义类转换 FloatComplex－>double
    operator double();	   // 成员函数，定义类转换 FloatComplex－>IntComplex
};
FloatComplex::operator double() { 
    return Real * Real + Imag * Imag;
}
FloatComplex::operator IntComplex() {
    IntComplex ic;
    ic.Real = Real;
    ic.Imag = Imag;
    return ic;
}

int main() {	
    FloatComplex c1(3.2, 4.1);
    double d = 2.5 + c1;   // 隐式调用类型转换函数FloatComplex->double
	cout << d << endl;
    IntComplex c2 = c1;    // 隐式调用类型转换函数FloatComplex->IntComplex
    cout << c2.Real << " " << c2.Imag << endl;
    return 0;
}
```


