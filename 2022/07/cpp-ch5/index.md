# 第五章 继承与派生


{{< admonition info "继承与派生">}}

继承，是两个类之间的关系。派生，是基类创建新类的过程。

继承的传递性将增加类之间的耦合度，应防止滥用继承。

{{< /admonition>}}

### 5.1 派生类中的基类成员的访问权限

```cpp
// 派生类的定义格式如下：
class <派生类名>:<继承方式> <基类名> {
    <派生类新定义成员>
};
```

类的继承方式有public(公有继承)、protected(保护继承)和private(私有继承)三种，**默认情况下为私有继承。**

派生类继承了**基类的全部成员变量，以及除了构造、析构函数之外的全部成员函数**，它们在派生类中的访问属性由继承方式控制。

{{< image src="img1.png" caption="访问权限" src_s="img1.png" src_l="img1.png" >}}

```cpp
#include <iostream>
using namespace std;

class Point {
public:
    void InitP(float xx = 0, float yy = 0) {
        X = xx;
        Y = yy;
    }
    void Move(float xOff, float yOff) {
        X += xOff;
        Y += yOff;
    }
    float GetX() {
        return X;
    }
    float GetY() {
        return Y;
    }
private:
    float X, Y;
};

class Rectangle: public Point {
public:
    void InitR(float x, float y, float w, float h) {
        InitP(x, y);
        W = w;
        H = h;
    }
    float GetH() {
        return H;
    }
    float GetW() {
        return W;
    }
private:
    float W, H;
};

int main() {
    Rectangle rect;
    rect.InitR(2, 3, 20, 10);
    rect.Move(3, 2);
    cout << "The date of rect(X,Y,W,H):" << endl;
    cout << rect.GetX() << "," << rect.GetY() << "," << rect.GetW() << "," << rect.GetH() << endl;
	return 0;
}
// The date of rect(X,Y,W,H):
// 5,5,20,10
```

### 5.2 派生类的构造与析构

构造函数：基类的构造函数→成员对象构造函数→派生类的构造函数。

析构函数：相反。

{{< admonition info "构造函数">}}

当基类中没有显式定义构造函数，或定义了无参构造函数时，派生类构造函数的初始化表可以省略对基类构造函数的调用，而采用隐含调用。

当基类的构造函数使用一个或多个参数时，派生类必须定义构造函数，提供将参数传递给基类构造函数的途径。这时，派生类构造函数的函数体可能为空，仅起到参数传递作用。

如果在基类中既定义了无参构造函数，又定义了带参构造函数，则在定义派生类构造函数时，既可以包含基类构造函数和参数，也可以不包含基类构造函数。

{{< /admonition>}}

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    Base() {
        cout << "Base Constructor1" << endl; 
        x = 0; y = 0; 
    }
    Base(int x, int y) { 
        cout << "Base Constructor2" << endl; 
        this->x = x; this->y = y; 
    }
    ~Base() { cout << "Deconstructor" << endl; }
    int x, y;
};

class B1:public Base{
public:
    B1() { z = 0; }
    int z;
};

class B2:public Base{
public:
    B2(int x, int y, int z) : Base(x,y) {
        this->z = z;
    }
    // 注释里这样做是不对的
    // B2(int x, int y, int z) {
    //     Base(x, y);
    //     this->z = z;
    // }
    int z;
};

int main() {
    B1 b1;
    B2 b2(1, 2, 3);
    return 0;
}
```

### 5.3 多继承

**5.3.1 构造函数调用顺序**

```cpp
<派生类名>(<总参数表>):<基类名1>(<参数表1>)，…，< 基类名n> (<参数表n>) {
    <派生类数据成员的初始化>
};
<总参数表>必须包含完成所有基类初始化所需的参数
```

- 先调用所有基类的构造函数，再调用对象成员类构造函数，最后调用派生类的构造函数。
- 处于同一层次的各基类构造函数的调用顺序取决于定义派生类时所指定的基类顺序，与派生类构造函数中所定义的成员初始化列表顺序无关。
- 如果有多个成员类对象，则构造函数的调用顺序是对象在类中被声明的顺序，而不是它们出现在成员初始化表中的顺序。
- 析构函数的调用顺序与构造函数的调用顺序相反

**5.4.2 多重继承引起的二义性问题**

- 两个基类有同名成员（编译出错）
- 两个基类和派生类三者都有同名成员（派生类定义的新成员会覆盖基类成员）
- 类C继承的类A、类B是从同一个基类派生的（会有两份拷贝）

### 5.4 虚基类

为避免对基类成员访问的二义性问题，可以将直接基类（如A、B）的共同基类（如N）设置为虚基类，这样共同基类（N）在内存中只有一个副本存在 。

{{< admonition info "虚拟类的理解">}}

从最底层开始到最顶上的基类，全部的继承路径中只有一个虚基类成员的副本

{{< /admonition>}}

**5.4.1 虚基类的定义格式**

```cpp
class <派生类名>:virtual <继承方式><共同基类名>;
```

```cpp
// 如果去掉virtual，会调用输出两次Constructor Base
#include <iostream>
using namespace std;

class Base {
public:
    Base() {
        cout << "Constructor Base" << endl;
    }
    int x;
};

class A:virtual public Base {
public:
    A() {
        cout << "Constructor A" << endl;
    }
};

class B:virtual public Base {
public:
    B() {
        cout << "Constructor B" << endl;
    }
};

class C:public A, B {
public:
    C() {
        cout << "Constructor C" << endl;
    }
};

int main() {
    C c;
    return 0;
}
// Constructor Base
// Constructor A
// Constructor B
// Constructor C
```

**5.4.2 虚基类的构造函数调用顺序**

- 先调用虚基类的构造函数，再调用非虚基类的构造函数。
- 若同一层次中包含多个虚基类，其调用顺序为定义时的顺序。
- 若虚基类由非虚基类派生而来，则仍按先调用基类构造函数，再调用派生类构造函数的顺序。

```cpp
#include <iostream>
using namespace std;

class Base1 {
public:
    Base1() {
        cout << "class Base1" << endl;
    }
};

class Base2 {
public:
    Base2() {
        cout << "class Base2" << endl;
    }
};

class Level1:public Base2, virtual public Base1 {
public:
    Level1() {
        cout << "class level1" << endl;
    }
};

class Level2:public Base2, virtual public Base1 {
public:
    Level2() {
        cout << "class level2" << endl;
    }
};

class TopLevel:public Level1, virtual public Level2 {
public:
    TopLevel() {
        cout << "class TopLevel" << endl;
    }
};

int main() {
    TopLevel tl;
    return 0;
}
// class Base1  toplevel->level2->base1
// class Base2  toplevel->level2->base2
// class level2 toplevel->level2
// class Base2  toplevel->level1->base2
// class level1 toplevel->level1
// class TopLevel toplevel
```

**5.4.3 虚基类的初始化**

如果在虚基类中定义了带参数的构造函数，则要在其**所有派生类（包括直接派生类或间接派生类）中**，通过构造函数的**初始化表**对虚基类进行初始化。

```cpp
#include <iostream>
using namespace std;

class A {
public:
    A(int i){}
};

class B:virtual public A {
public:
    B(int n):A(n){}
};

class C:virtual public A {
public:
    C(int n):A(n){}
};

class D:public B, public C {
public:
    D(int n):A(n),B(n),C(n){}
};

int main() {
    D d(5);
    return 0;
} 
```

{{< admonition question "为什么?">}}

不是很能理解为什么要这么做，每个直接或间接派生类，都要调用一次。

编译器会自动优化，B，C的构造函数中不会再次调用A类的构造函数。

{{< /admonition>}}


