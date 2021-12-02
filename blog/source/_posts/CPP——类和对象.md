---
title: C++（一）—— 类和对象
date: 2020-08-16 19:21:20
tags:
- C/C++
categories:
- 语言
---
C和C++最大的区别就是C语言是面向过程的语言，而C++是面向对象的语言。C++最初叫做“带类的C”，顾名思义，理论上C++可以认为是C语言的超集。

类和对象是C++的重要特性，它们使得C++成为面向对象的编程语言，可以用来开发中大型项目。C++中的类（class）可以看作C语言中结构体（struct）的升级版，不同的是，类的成员不但可以是变量，还可以是函数。通过类定义出来的变量被称作对象。

本文的内容大多学习于《C++ Primer》和[C++入门教程](http://c.biancheng.net/cplus/)，大家可以移步学习。在此，我也是仿照该教程敲代码学习一遍，做一做学习笔记。
<!-- more -->

## 0 类的定义
类是创建对象的，一个类可以创建多个对象，每个对象都是类类型的一个变量；创建对象的过程也叫类的实例化。与结构体一样，类只是一种复杂数据类型的声明，不占用内存空间。而对象是类这种数据类型的一个变量，或者说是通过类这种数据类型创建出来的一份实实在在的数据，所以占用内存空间。

和许多教程一样，我们定义一个学生（Student）类，这里我们直接采取多文件编程，一般类的声明放在头文件中，将函数的实现放在源文件中，首先新建一个名为student.h的头文件，如下：
``` c++
#ifndef _STUDENT_H_
#define _STUDENT_H_

#include <string>

class Student {
private:
    std::string name;
    int age;
    float score;

public:
    void setname(std::string name);
    void setage(int age);
    void setscore(float score);
    void show();
};

#endif
```

然后新建一个名为student.cpp的源文件，并在其中实现类的成员函数：
``` c++
#include "student.h"
#include <iostream>

using namespace std;

void Student::setname(string name) {
    this->name = name;
}

void Student::setage(int age) {
    this->age = age;
}

void Student::setscore(float score) {
    this->score = score;
}

void Student::show() {
    cout<<name<<"'s age is "<<age<<", grade is "<<score<<endl;
}
```

在主函数中进行调用，如下：
``` c++
#include "student.h"
using namespace std;

int main() {
    Student stu;
    stu.setname("Tom Cat");
    stu.setage(16);
    stu.setscore(99.5);
    stu.show();

    return 0;
}
```

最后，我们输入以下指令进行编译，得到可执行文件并执行：
``` bash
$ g++ -o main main.cpp student.cpp && ./main
Tom Cat's age is 16, grade is 99.5
```

### 0.1 类的成员属性
在C++中定义了**访问说明符**加强类的封装性：
* 定义在**public**说明符之后的成员在整个程序内可以被访问，public成员定义类的**接口**；
* 定义在**private**说明符之后的成员可以被类的成员函数访问，但是不能被使用该类的代码访问，private**封装（隐藏）**了类的实现细节。

值得注意的是，C++中struct也可以拥有成员函数，它和class唯一的区别就是默认访问权限不同：在第一个访问说明符之前定义的成员，即没有显式定义访问权限的成员，class关键字默认为private，而struct关键字默认为public。相比之下，C++中的struct和C中的struct区别更大。

在一个类体中，private 和 public 可以分别出现多次。每个部分的有效范围到出现另一个访问限定符或类体结束时（最后一个右花括号）为止。但是为了使程序清晰，应该养成这样的习惯，使每一种成员访问限定符在类定义体中只出现一次。

Java、C# 程序员注意，C++ 中的 public、private、protected 只能修饰类的成员，不能修饰类，C++中的类没有共有私有之分。

### 0.2 成员函数的定义
::被称为域解析符（也称作用域运算符或作用域限定符），用来连接类名和函数名，指明当前函数属于哪个类。

成员函数必须先在类体中作原型声明，然后在类外定义，也就是说类体的位置应在函数定义之前。在类体中定义的成员函数会自动成为内联函数。

在这里需要说明一下，内联函数在编译的时候会将函数体展开在函数调用的地方，当然，编译器可能会忽略这个请求。如果我们想要在类体外定义内联函数，除了inline关键字外，还需要将其放在类体定义同一文件中（原因是编译器只针对单一源文件进行编译，如果不在统一文件中（预处理：头文件会自动展开到源文件），源文件去替换函数时将找不到函数体），所以不如直接在类体中定义，这样相当简单，还不需要关键字声明。

在类体外定义成员函数需要使用域解析符声明函数所属的类。

### 0.3 this指针
观察student.cpp可以看到，在三个set函数的实现时，我使用了this指针，但是最后show函数的实现却没有用到。

this 只能用在类的内部，通过 this 可以访问类的所有成员。在三个set函数中成员函数的参数和成员变量重名，只能通过 this 区分。在show函数中，用不用this形式都可以，毕竟没有重名的困扰。

this 虽然用在类的内部，但是只有在对象被创建以后才会给 this 赋值，并且这个赋值的过程是编译器自动完成的，不需要用户干预，用户也不能显式地给 this 赋值。而且对于不同的对象，this 的值也不一样。

几点注意：
* this 是 const 指针，它的值是不能被修改的，一切企图修改该指针的操作，如赋值、递增、递减等都是不允许的；
* this 只能在成员函数内部使用，用在其他地方没有意义，也是非法的；
* 只有当对象被创建后 this 才有意义，因此不能在 static 成员函数中使用（后续会讲到 static 成员）。

this 实际上是成员函数的一个形参，在调用成员函数时将对象的地址作为实参传递给 this。不过 this 这个形参是隐式的，它并不出现在代码中，而是在编译阶段由编译器默默地将它添加到参数列表中。

this 作为隐式形参，本质上是成员函数的局部变量，所以只能用在成员函数的内部，并且只有在通过对象调用成员函数时才给 this 赋值。

成员函数最终被编译成与对象无关的普通函数，除了成员变量，会丢失所有信息，所以编译时要在成员函数中添加一个额外的参数，把当前对象的首地址传入，以此来关联成员函数和成员变量。这个额外的参数，实际上就是 this，它是成员函数和成员变量关联的桥梁。

## 1 构造函数和析构函数
### 1.1 构造函数
类通过一个或者几个特殊的成员函数来控制其对象的初始化过程，这些函数叫做构造函数（Constructor）。构造函数的任务是初始化类对象的数据成员，无论何时只要类的对象被创建，就会执行构造函数。

在前面的例子中，我们通过成员函数 setname()、setage()、setscore() 分别为成员变量 name、age、score 赋值，这样做虽然有效，但显得有点麻烦。有了构造函数，我们就可以简化这项工作，在创建对象的同时为成员变量赋值。修改student.h文件如下：
``` c++
#ifndef _STUDENT_H_
#define _STUDENT_H_

#include <string>

class Student {
private:
    std::string name;
    int age;
    float score;

public:
    Student(std::string name, int age, float score);
    void show();
};

#endif
```

修改student.cpp如下：
``` c++
#include "student.h"
#include <iostream>

using namespace std;

Student::Student(string name, int age, float score) {
    this->name = name;
    this->age = age;
    this->score = score;
}

void Student::show() {
    cout<<name<<"'s age is "<<age<<", grade is "<<score<<endl;
}
```
对应地，修改main.cpp如下：
``` c++
#include "student.h"
using namespace std;

int main() {
    Student stu("Tom Cat", 16, 99.5);
    stu.show();

    return 0;
}
```
再次执行以下指令，得到的结果相同。
``` bash
$ g++ -o main main.cpp student.cpp && ./main
Tom Cat's age is 16, grade is 99.5
```

构造函数必须是 public 属性的，否则创建对象时无法调用。当然，设置为 private、protected 属性也不会报错，但是没有意义。

构造函数没有返回值，因为没有变量来接收返回值，即使有也毫无用处，这意味着：
* 不管是声明还是定义，函数名前面都不能出现返回值类型，即使是 void 也不允许；
* 函数体中不能有 return 语句。

和普通成员函数一样，构造函数是允许重载的。一个类可以有多个重载的构造函数，创建对象时根据传递的实参来判断调用哪一个构造函数。

构造函数的调用是强制性的，一旦在类中定义了构造函数，那么创建对象时就一定要调用，不调用是错误的。如果有多个重载的构造函数，那么创建对象时提供的实参必须和其中的一个构造函数匹配；反过来说，创建对象时只有一个构造函数会被调用。

#### 1.1.1 默认构造函数
如果用户自己没有定义构造函数，那么编译器会自动生成一个**默认构造函数**，只是这个构造函数的函数体是空的，也没有形参，也不执行任何操作，其按照如下的规则初始化类的成员变量：
* 如果存在类内的初始值，用它来初始化成员变量；
* 否则，默认初始化。

一个类必须有构造函数，要么用户自己定义，要么编译器自动生成。一旦用户自己定义了构造函数，不管有几个，也不管形参如何，编译器都不再自动生成。示例中，Student 类已经有了一个构造函数，也就是我们自己定义的，编译器不会再额外添加构造函数Student()。为了避免出现定义无参数的对象时出错，我们一般手动定义一个类似默认构造函数，如下：

student.h添加第三行```Student();```:
``` c++
...
public:
    Student();
    Student(string name, int age, float score);
...
```

student.cpp添加:
``` c++
Student::Student() { name = ""; age = 0; score = 0.0; }
```
这个时候最好保留set函数，因为总要设置成员变量值的嘛。

我们还可以用如下形式构造默认构造函数，``` = default ```表示要求编译器来生成构造函数，其既可以出现在类体内（内联函数），也可以出现在类的外部（普通函数）。
``` c++
...
public:
    Student() = default;
    Student(string name, int age, float score);
...
```


#### 1.1.2 构造函数初始化列表
初始化列表使得代码更加简洁。修改构造函数如下：
``` c++
Student::Student(string name, int age, float score) : name(name), age(age), score(score) {}
```
值得注意的是：this指针属于对象，初始化列表在构造函数之前执行，在对象还没有构造完成前，使用this指针，编译器无法识别，所以构造函数初始化列表时不能使用this指针。

初始化 const 成员变量的唯一方法就是使用初始化列表，这个会在后面讲到。


#### 1.1.3 拷贝构造函数
如果一个构造函数的第一个参数是自身类类型的引用，且任何额外的参数都有默认值，则此构造函数是拷贝构造函数，拷贝构造函数也叫复制构造函数。例如我们在student.cpp中添加如下代码：
``` c++
Student(const Student &stu);
```

可以看出，拷贝构造函数只有一个参数，它的类型是当前类的引用，而且一般都是 const 引用。

**1）为什么必须是当前类的引用呢？**

如果拷贝构造函数的参数不是当前类的引用，而是当前类的对象，那么在调用拷贝构造函数时，会将另外一个对象直接传递给形参，这本身就是一次拷贝，会再次调用拷贝构造函数，然后又将一个对象直接传递给了形参，将继续调用拷贝构造函数……这个过程会一直持续下去，没有尽头，陷入死循环。

**2) 为什么是 const 引用呢？**

拷贝构造函数的目的是用其它对象的数据来初始化当前对象，并没有期望更改其它对象的数据，添加 const 限制后，这个含义更加明确了。

另外一个原因是，添加 const 限制后，可以将 const 对象和非 const 对象传递给形参了，因为非 const 类型可以转换为 const 类型。如果没有 const 限制，就不能将 const 对象传递给形参，因为 const 类型不能转换为非 const 类型，这就意味着，不能使用 const 对象来初始化当前对象了。

如果我们没有为一个类定义拷贝构造函数，那么编译器会为我们定义一个，这个默认的拷贝构造函数很简单，就是使用“老对象”的成员变量对“新对象”的成员变量进行一一赋值，和上面 Student 类的拷贝构造函数非常类似。对于简单的类，默认拷贝构造函数一般是够用的，我们也没有必要再显式地定义一个功能类似的拷贝构造函数。但是当类持有其它资源时，如动态分配的内存、打开的文件、指向其他数据的指针、网络连接等，默认拷贝构造函数就不能拷贝这些资源，我们必须显式地定义拷贝构造函数，以完整地拷贝对象的所有数据。

### 1.2 析构函数
创建对象时系统会自动调用构造函数进行初始化工作，同样，销毁对象时系统也会自动调用一个函数来进行清理工作，例如释放分配的内存、关闭打开的文件等，这个函数就是析构函数。

析构函数（Destructor）也是一种特殊的成员函数，没有返回值，不需要程序员显式调用（程序员也没法显式调用），而是在销毁对象时自动执行。构造函数的名字和类名相同，而析构函数的名字是在类名前面加一个\~符号。

注意：析构函数没有参数，不能被重载，因此一个类只能有一个析构函数。如果用户没有定义，编译器会自动生成一个默认的析构函数。

C++ 中的 new 和 delete 分别用来分配和释放内存，它们与C语言中 malloc()、free() 最大的一个不同之处在于：用 new 分配内存时会调用构造函数，用 delete 释放内存时会调用析构函数。构造函数和析构函数对于类来说是不可或缺的，所以在C++中我们非常鼓励使用 new 和 delete。

析构函数在对象被销毁时调用，而对象的销毁时机与它所在的内存区域有关：
* 在所有函数之外创建的对象是全局对象，它和全局变量类似，位于内存分区中的全局数据区，程序在结束执行时会调用这些对象的析构函数；
* 在函数内部创建的对象是局部对象，它和局部变量类似，位于栈区，函数执行结束时会调用这些对象的析构函数；
* new 创建的对象位于堆区，通过 delete 删除时才会调用析构函数；如果没有 delete，析构函数就不会被执行。

如下，添加析构函数如下，可以观察构造函数和析构函数的调用时机，修改student.cpp：
``` c++
Student::Student(string name, int age, float score) : name(name), age(age), score(score) {
    cout<<name<<" Constructor!"<<endl;
}

Student::~Student() { cout<<name<<" Destructor!"<<endl; }
```
修改main.cpp如下：
``` c++
#include <iostream>
using namespace std;

Student g_stu("Global", 15, 98);

void func() {
    Student stu("Local", 17, 93);
}

int main() {
    Student stu("Tom Cat", 16, 99.5);

    Student *p_stu = new Student("New", 18, 96);

    func();

    cout<<"main END!"<<endl;
    return 0;
}
```

最后执行结果如下，可以看出，全局对象Global最先创建，最后退出；局部变量Local在退出func函数时及调用析构函数；通过new申请内存的New对象，因为没有调用delete，所以没有调用析构函数；Cat在main函数结束后调用析构函数。
``` bash
$ g++ -o main main.cpp student.cpp && ./main
Global Constructor!
Tom Cat Constructor!
New Constructor!
Local Constructor!
Local Destructor!
main END!
Tom Cat Destructor!
Global Destructor!
```

析构函数体自身并不直接销毁成员，成员是在析构函数体之后隐含的析构阶段中销毁的。

特别地，当我们在构造函数中使用了new申请了堆内存，在析构函数中需要使用delete释放内存。

## 2 静态成员
### 2.1 静态成员变量
有时候类需要一些成员与类本身直接相关，而不是与类的各个对象保持关联。有时候我们希望在多个对象之间共享数据，对象 a 改变了某份数据后对象 b 可以检测到。共享数据的典型使用场景是计数，以前面的 Student 类为例，如果我们想知道班级中共有多少名学生，就可以设置一份共享的变量，每次创建对象时让该变量加1。我们可以使用静态成员变量来实现多个对象共享数据的目标。静态成员变量是一种特殊的成员变量，它被关键字static修饰。如下，在student.h中添加static成员变量total。
``` c++
public:
    static int total;
```
static 成员变量属于类，不属于某个具体的对象，即使创建多个对象，也只为 total 分配一份内存，所有对象使用的都是这份内存中的数据。当某个对象修改了 total，也会影响到其他对象。static 成员变量必须在类声明的外部初始化，在student.cpp中进行外部初始化，静态成员变量在初始化时不能再加 static，但必须要有数据类型。被 private、protected、public 修饰的静态成员变量都可以用这种方式初始化。
```c++
int Student::total = 0;
```
注意：
* static 成员变量的内存既不是在声明类时分配，也不是在创建对象时分配，而是在（类外）初始化时分配。反过来说，没有在类外初始化的 static 成员变量不能使用;
* static 成员变量不占用对象的内存，而是在所有对象之外开辟内存，即使不创建对象也可以访问。具体来说，static 成员变量和普通的 static 变量类似，都在内存分区中的全局数据区分配内存，如下，在main.cpp中添加如下代码。
```c++
#include "student.h"
#include <iostream>
using namespace std;

int main() {
    /* Access the static memeber with class */
    Student::total = 10;
    cout<<"Member total address is "<<&Student::total<<endl;

    /* Access the static member with object */
    Student stu("Tom Cat", 16, 99.5);
    cout<<"Total is "<<stu.total<<", address is "<<&stu.total<<endl;
    stu.total = 20;
    cout<<"Total is "<<stu.total<<", address is "<<&stu.total<<endl;

    return 0;
}
```
这时候我们执行如下指令可以证明静态成员变量是同一块地址。
``` bash
$ g++ -o main main.cpp student.cpp && ./main
Member total address is 0x6021e4
Total is 10, address is 0x6021e4
Total is 20, address is 0x6021e4
Tom Cat Destructor!
```

但是对于total这一静态成员，我们应该将其设置为私有（private）属性，并在每次定义对象的时候+1，所以我们修改如下：
``` c++
// student.h
private:
	...
	static int total;
```
``` c++
// student.cpp
int Student::total = 0;

Student::Student(string name, int age, float score) : name(name), age(age), score(score) {
    total++;
}

void Student::show() {
    cout<<name<<"'s age is "<<age<<", grade is "<<score<<", total student num is "<<total<<endl;
}
```
``` c++
// main.cpp
int main() {
    Student stu1("Tom Cat", 16, 99.5);
    stu1.show();

    Student stu2("Jerry Mouse", 16, 99.5);
    stu2.show();

    Student stu3("Goff Dog", 16, 99.5);
    stu3.show();

    return 0;
}
```
执行结果如下：
```bash
$ g++ -o main main.cpp student.cpp && ./main
Tom Cat's age is 16, grade is 99.5, total student num is 1
Jerry Mouse's age is 16, grade is 99.5, total student num is 2
Goff Dog's age is 16, grade is 99.5, total student num is 3
```

注意：
*  一个类中可以有一个或多个静态成员变量，所有的对象都共享这些静态成员变量，都可以引用它；
* static 成员变量和普通 static 变量一样，都在内存分区中的全局数据区分配内存，到程序结束时才释放。这就意味着，static 成员变量不随对象的创建而分配内存，也不随对象的销毁而释放内存。而普通成员变量在对象创建时分配内存，在对象销毁时释放内存；
* 静态成员变量既可以通过对象名访问，也可以通过类名访问，但要遵循 private、protected 和 public 关键字的访问权限限制，**譬如当total是private属性时，就不能通过对象名直接进行访问了**。当通过对象名访问时，对于不同的对象，访问的是同一份内存。

### 2.2 静态成员函数
在类中，static 除了可以声明静态成员变量，还可以声明静态成员函数。**普通成员函数可以访问所有成员（包括成员变量和成员函数），静态成员函数只能访问静态成员。**

编译器在编译一个普通成员函数时，会隐式地增加一个形参 this，并把当前对象的地址赋值给 this，所以普通成员函数只能在创建对象后通过对象来调用，因为它需要当前对象的地址。而静态成员函数可以通过类来直接调用，编译器不会为它增加形参 this，它不需要当前对象的地址，所以不管有没有创建对象，都可以调用静态成员函数。

普通成员变量占用对象的内存，静态成员函数没有 this 指针，不知道指向哪个对象，无法访问对象的成员变量，也就是说静态成员函数不能访问普通成员变量，只能访问静态成员变量。

**普通成员函数必须通过对象才能调用**，而静态成员函数没有 this 指针，无法在函数体内部访问某个对象，所以不能调用普通成员函数，只能调用静态成员函数。

静态成员函数与普通成员函数的根本区别在于：**普通成员函数有 this 指针，可以访问类中的任意成员；而静态成员函数没有 this 指针，只能访问静态成员（包括静态成员变量和静态成员函数）**。

例如，我们增加返回并显示总人数的函数接口，在三个文件中添加的代码如下：
```c++
// student.h
public:
	static int getAndShowTotal();

// student.cpp
int Student::getAndShowTotal() {
    cout<<"Total is "<<total<<endl;
    return total;
}

// main.cpp
int main() {
    Student stu1("Tom Cat", 16, 99.5);
    stu1.show();

    Student stu2("Jerry Mouse", 16, 99.5);
    stu2.show();
    stu1.getAndShowTotal();//通过对象访问

    Student stu3("Goff Dog", 16, 99.5);
    stu3.show();
    Student::getAndShowTotal();//通过类访问

    return 0;
}
```
执行如下：
``` bash
$ g++ -o main main.cpp student.cpp && ./main
Tom Cat's age is 16, grade is 99.51
Jerry Mouse's age is 16, grade is 99.52
Total is 2
Goff Dog's age is 16, grade is 99.53
Total is 3
```
和静态成员变量类似，静态成员函数在声明时要加 static，在定义时不能加 static。

## 3 const成员和对象
### 3.1 const成员变量
const 成员变量的用法和普通 const 变量的用法相似，只需要在声明时加上 const 关键字。初始化 const 成员变量只有一种方法，就是通过构造函数的初始化列表，这点在前面已经讲到了。

### 3.2 const成员函数
const 成员函数也称为常成员函数。常成员函数需要在声明和定义的时候在函数头部的结尾加上 const 关键字，这里，const的作用是修改隐式this指针的类型。常成员函数可以使用类中的所有成员变量，但是不能修改它们的值，这种措施主要还是为了保护数据而设置的。我们通常将 get 函数设置为常成员函数。

如下修改：
``` c++
// student .h
public:
    string getname() const;
    int getage() const;
    float getscore() const;

// student.cpp
string Student::getname() const { return name; }
int Student::getage() const { return age; }
float Student::getscore() const { return score; }
```
这三个函数的功能都很简单，仅仅是为了获取成员变量的值，没有任何修改成员变量的企图，所以我们加了 const 限制，这是一种保险的做法，同时也使得语义更加明显。

注意：
* 需要强调的是，必须在成员函数的声明和定义处同时加上 const 关键字。如果只在一个地方加 const 会导致声明和定义处的函数原型冲突；
* 静态成员函数不能修饰成const成员函数，因为const的作用是修改隐式this指针的类型，而静态成员函数连this指针都没有，所以不能修饰成const成员函数。

### 3.3 const对象
在 C++ 中，const 也可以用来修饰对象，称为常对象。**一旦将对象定义为常对象之后，就只能调用类的 const 成员（包括 const 成员变量和 const 成员函数）了**。

如下，在main函数中定义常对象，访问其非const成员是错误的做法。
```c++
int main() {
    const Student stu("Tom Cat", 16, 99.5);
//  stu.show(); //error
    cout<<stu.getname()<<"'s age is "<<stu.getage()<<"'s score is "<<stu.getscore()<<endl;

    const Student *p_stu = new Student("Jerry Mouse", 17, 98);
//  p_stu->show(); //error
    cout<<p_stu->getname()<<"'s age is "<<p_stu->getage()<<"'s score is "<<p_stu->getscore()<<endl;

    return 0;
}
```
执行后：
``` bash
$ g++ -o main main.cpp student.cpp && ./main
Tom Cat's age is 16's score is 99.5
Jerry Mouse's age is 17's score is 98
```


## 4 友元（friend）
类可以允许其他类或者函数访问它的非公有成员，方法是令其他类或者函数称为它的友元（friend）。
### 4.1 友元函数
在当前类以外定义的、不属于当前类的函数也可以在类中声明，但要在前面加 friend 关键字，这样就构成了友元函数。友元函数可以是不属于任何类的非成员函数，也可以是其他类的成员函数。

友元函数可以访问当前类中的所有成员，包括 public、protected、private 属性的。

#### 4.1.1 将非成员函数声明为友元函数
如下，将show函数为友元函数，分别修改源码如下：
``` c++
// student.h
friend void show(Student *);

// student.cpp
void show(Student *p_stu) {
    cout<<p_stu->name<<"'s age is "<<p_stu->age<<", grade is "<<p_stu->score<<endl;
}

// main.cpp
int main() {
    Student stu("Tom Cat", 16, 99.5);
    show(&stu); //error

    Student *p_stu = new Student("Jerry Mouse", 17, 98);
    show(p_stu); //error

    return 0;
}
```
运行结果为：
``` bash
$ g++ -o main main.cpp student.cpp && ./main
Tom Cat's age is 16, grade is 99.5
Jerry Mouse's age is 17, grade is 98
```
show() 是一个全局范围内的非成员函数，它不属于任何类。注意，友元函数不同于类的成员函数，在友元函数中不能直接访问类的成员，必须要借助对象，所以show函数必须通过Student传参参能访问类的成员。成员函数在调用时会隐式地增加 this 指针，指向调用它的对象，从而使用该对象的成员；而 show() 是非成员函数，没有 this 指针，编译器不知道使用哪个对象的成员，要想明确这一点，就必须通过参数传递对象（可以直接传递对象，也可以传递对象指针或对象引用），并在访问成员时指明对象。

#### 4.1.2 将其他类的成员函数声明为友元函数
如下，我们声明一个Address类，用作表征人的籍贯属性。声明Student类的成员函数show作为Address类的友元函数。首先我们新建address.cpp和address.h文件，分别如下：
``` c++
// address.h
#ifndef _ADDRESS_H_
#define _ADDRESS_H_

#include <string>
#include "student.h"
using namespace std;

class Address {
private:
    string province;
    string city;
    string district;

public:
    Address(string province, string city, string district);
    friend void Student::show(Address *);
};

#endif
```
``` c++
// address.cpp
#include "address.h"

Address::Address(string province, string city, string district) {
    this->province = province;
    this->city = city;
    this->district = district;
}
```
修改原来三个文件如下：
``` c++
// student.h
class Address;

class Student {
private:
	...

public:
	...
    void show(Address *);
    ...
};

// student.cpp
#include "address.h"
...

void Student::show(Address *addr) {
    cout<<name<<"'s age is "<<age<<", grade is "<<score;
    cout<<", address is "<<addr->province<<" "<<addr->city<<" "<<addr->district<<endl;
}

// main.cpp
#include "student.h"
#include "address.h"
#include <iostream>
using namespace std;

int main() {
    Student stu("Tom Cat", 16, 99.5);
    Address addr("Beijing", "", "Haidian");
    stu.show(&addr);

    Student *p_stu = new Student("Jerry Mouse", 17, 98);
    Address *p_addr = new Address("Jiangsu", "Nanjing", "Yuhuatai");
    p_stu->show(p_addr);

    return 0;
}
```
执行结果如下：
``` bash
$ g++ -o main main.cpp student.cpp address.cpp && ./main 
Tom Cat's age is 16, grade is 99.5, address is Beijing  Haidian
Jerry Mouse's age is 17, grade is 98, address is Jiangsu Nanjing Yuhuatai
```

这里需要注意以下几点：
* 是在**被友元**的类中声明其它类的成员函数，类似于在自己家里声明谁是朋友，这样谁才能见到我家的私有成员，譬如闺女；
* 在student.h中需要对Address类进行提前声明，在Student类中使用到了Address类；但是却不能直接包含address.h头文件，这是因为头文件中也包含了student.h，在预处理展开头文件时，不会重复展开student.h，所以导致在解析该头文件时，对Address类内用到的Student成员无法解析；
* 在address.h中需要包含student.h的头文件，因为其不仅用到了Student类，还用到了其成员，必须知道其声明格式。

一般情况下，类必须在正式声明之后才能使用；但是某些情况下（如上例所示），只要做好提前声明，也可以先使用。建对象时要为对象分配内存，在正式声明类之前，编译器无法确定应该为对象分配多大的内存。编译器只有在“见到”类的正式声明后（其实是见到成员变量），才能确定应该为对象预留多大的内存。在对一个类作了提前声明后，可以用该类的名字去定义指向该类型对象的指针变量（本例就定义了 Address 类的指针变量）或引用变量，因为指针变量和引用变量本身的大小是固定的，与它所指向的数据的大小无关。

### 4.2 友元类
不仅可以将一个函数声明为一个类的友元，还可以将整个类声明为另一个类的友元，这就是友元类。友元类中的所有成员函数都是另一个类的友元函数。

如下，我们将Student类声明为Address类的友元类，修改address.h如下：
``` c++
class Address {
...
public:
    friend class Student;
};
```
最后执行也能得到正确结果：
``` bash
$ g++ -o main main.cpp student.cpp address.cpp && ./main 
Tom Cat's age is 16, grade is 99.5, address is Beijing  Haidian
Jerry Mouse's age is 17, grade is 98, address is Jiangsu Nanjing Yuhuatai
```

关于友元之间的关系，有以下准则：
* 友元的关系是单向的而不是双向的。如果声明了类 B 是类 A 的友元类，不等于类 A 是类 B 的友元类，类 A 中的成员函数不能访问类 B 中的 private 成员；
* 友元的关系不能传递。如果类 B 是类 A 的友元类，类 C 是类 B 的友元类，不等于类 C 是类 A 的友元类。

## 5 小结
全部来自于[C++入门教程——C++类和对象的总结](http://c.biancheng.net/view/221.html)：

类的成员有成员变量和成员函数两种。

成员函数之间可以互相调用，成员函数内部可以访问成员变量。

私有成员只能在类的成员函数内部访问。默认情况下，class 类的成员是私有的，struct 类的成员是公有的。

可以用“对象名.成员名”、“引用名.成员名”、“对象指针->成员名”的方法访问对象的成员变量或调用成员函数。成员函数被调用时，可以用上述三种方法指定函数是作用在哪个对象上的。

对象所占用的存储空间的大小等于各成员变量所占用的存储空间的大小之和（如果不考虑成员变量对齐问题的话）。

定义类时，如果一个构造函数都不写，则编译器自动生成默认（无参）构造函数和复制构造函数。如果编写了构造函数，则编译器不自动生成默认构造函数。一个类不一定会有默认构造函数，但一定会有复制构造函数。

任何生成对象的语句都要说明对象是用哪个构造函数初始化的。即便定义对象数组，也要对数组中的每个元素如何初始化进行说明。如果不说明，则编译器认为对象是用默认构造函数或参数全部可以省略的构造函数初始化。在这种情况下，如果类没有默认构造函数或参数全部可以省略的构造函数，则编译出错。

对象在消亡时会调用析构函数。

每个对象有各自的一份普通成员变量，但是静态成员变量只有一份，被所有对象所共享。静态成员函数不具体作用于某个对象。即便对象不存在，也可以访问类的静态成员。静态成员函数内部不能访问非静态成员变量，也不能调用非静态成员函数。

常量对象上面不能执行非常量成员函数，只能执行常量成员函数。

包含成员对象的类叫封闭类。任何能够生成封闭类对象的语句，都要说明对象中包含的成员对象是如何初始化的。如果不说明，则编译器认为成员对象是用默认构造函数或参数全部可以省略的构造函数初始化。

在封闭类的构造函数的初始化列表中可以说明成员对象如何初始化。封闭类对象生成时，先执行成员对象的构造函数，再执行自身的构造函数；封闭类对象消亡时，先执行自身的析构函数，再执行成员对象的析构函数。

const 成员和引用成员必须在构造函数的初始化列表中初始化，此后值不可修改。

友元分为友元函数和友元类。友元关系不能传递。

成员函数中出现的 this 指针，就是指向成员函数所作用的对象的指针。因此，静态成员函数内部不能出现 this 指针。成员函数实际上的参数个数比表面上看到的多一个，多出来的参数就是 this 指针。