---
title: C++（二）—— 继承与派生
date: 2020-09-08 21:55:32
tags:
- C/C++
categories:
- 语言
---

C++ 中的继承是类与类之间的关系，继承（Inheritance）可以理解为一个类从另一个类获取成员变量和成员函数的过程。例如类B继承于类A，那么B就拥有A的成员变量和成员函数。派生（Derive）和继承是一个概念，只是站的角度不同。继承是儿子接收父亲的产业，派生是父亲把产业传承给儿子。被继承的类称为父类或基类，继承的类称为子类或派生类。“子类”和“父类”通常放在一起称呼，“基类”和“派生类”通常放在一起称呼。
<!-- more -->

## 0 继承的典型场景
以下是两种典型的使用继承的场景：
* 当你创建的新类与现有的类相似，只是多出若干成员变量或成员函数时，可以使用继承，这样不但会减少代码量，而且新类会拥有基类的所有功能。
* 当你需要创建多个类，它们拥有很多相似的成员变量或成员函数时，也可以使用继承。可以将这些类的共同成员提取出来，定义为基类，然后从基类继承，既可以节省代码，也方便后续修改成员。

如下，我们可以定义People类，并由People类派生出Student类。代码如下：
``` c++
// people.h
#ifndef _PEOPLE_H_
#define _PEOPLE_H_

#include <string>
using namespace std;

class People {
private:
    string name;
    int age;

public:
    void setname(string name);
    void setage(int age);
    string getname() const;
    int getage() const;
};

#endif


// people.cpp
#include "people.h"

void People::setname(string name) { this->name = name; }

void People::setage(int age) { this->age = age; }

string People::getname() const { return name; }

int People::getage() const { return age; }


// student.h
#ifndef _STUDENT_H_
#define _STUDENT_H_

#include "people.h"

class Student : public People {
private:
    float score;

public:
    void setscore(float score);
    float getscore() const;
};

#endif


// student.cpp
#include "student.h"

void Student::setscore(float score) { this->score = score; }

float Student::getscore() const { return score; }


// main.cpp
#include "student.h"
#include <iostream>

int main(){
    Student stu;
    stu.setname("Tom");
    stu.setage(16);
    stu.setscore(95.5f);
    cout<<stu.getname()<<"'s age is "<<stu.getage()<<", score is "<<stu.getscore()<<endl;
    return 0;
}
```

执行结果如下：
``` bash
$ g++ -o main main.cpp student.cpp people.cpp && ./main 
Tom's age is 16, score is 95.5
```

本例中，People 是基类，Student 是派生类。Student 类继承了 People 类的成员，同时还新增了自己的成员变量 score 和成员函数 setscore()、getscore()。这些继承过来的成员，可以通过子类对象访问，就像自己的一样。

注意第41行，在student.h中，这就是声明派生类的语法。class 后面的“Student”是新声明的派生类，冒号后面的“People”是已经存在的基类。在“People”之前有一关键字 public，用来表示是公有继承。

## 1 三种继承方式
继承的方式总结如下：
``` c++
class 派生类名:［继承方式］ 基类名{
    派生类新增加的成员
};
```
继承方式限定了基类成员在派生类中的访问权限，包括 public（公有的）、private（私有的）和 protected（受保护的）。此项是可选项，如果不写，默认为 private（成员变量和成员函数默认也是 private）。

protected 成员和 private 成员类似，也不能通过对象访问。但是当存在继承关系时，protected 和 private 就不一样了：基类中的 protected 成员可以在派生类中使用，而基类中的 private 成员不能在派生类中使用。不同的继承方式会影响基类成员在派生类中的访问权限。

**1）public继承方式**
* 基类中所有 public 成员在派生类中为 public 属性；
* 基类中所有 protected 成员在派生类中为 protected 属性；
* 基类中所有 private 成员在派生类中不能使用。

**2）protected继承方式**
* 基类中的所有 public 成员在派生类中为 protected 属性；
* 基类中的所有 protected 成员在派生类中为 protected 属性；
* 基类中的所有 private 成员在派生类中不能使用。

**3）private继承方式**
* 基类中的所有 public 成员在派生类中均为 private 属性；
* 基类中的所有 protected 成员在派生类中均为 private 属性；
* 基类中的所有 private 成员在派生类中不能使用。

可以总结出：
* 基类成员在派生类中的访问权限不得高于继承方式中指定的权限。也就是说，继承方式中的 public、protected、private 是用来指明基类成员在派生类中的最高访问权限的。
* 不管继承方式如何，基类中的 private 成员在派生类中始终不能使用（不能在派生类的成员函数中访问或调用）。
* 如果希望基类的成员能够被派生类继承并且毫无障碍地使用，那么这些成员只能声明为 public 或 protected；只有那些不希望在派生类中使用的成员才声明为 private。
* 如果希望基类的成员既不向外暴露（不能通过对象访问），还能在派生类中使用，那么只能声明为 protected。
* 由于 private 和 protected 继承方式会改变基类成员在派生类中的访问权限，导致继承关系复杂，所以实际开发中我们一般使用 public。


## 2 继承时的名字遮蔽
如果派生类中的成员（包括成员变量和成员函数）和基类中的成员重名，那么就会遮蔽从基类继承过来的成员。所谓遮蔽，就是在派生类中使用该成员（包括在定义派生类时使用，也包括通过派生类对象访问该成员）时，实际上使用的是派生类新增的成员，而不是从基类继承来的。

基类成员和派生类成员的名字一样时会造成遮蔽，这句话对于成员变量很好理解，对于成员函数要引起注意，不管函数的参数如何，只要名字一样就会造成遮蔽。换句话说，基类成员函数和派生类成员函数不会构成重载，如果派生类有同名函数，那么就会遮蔽基类中的所有同名函数，不管它们的参数是否一样。

## 3 基类和派生类的构造函数/析构函数
### 3.1 构造函数
派生类可以继承基类的普通成员函数，但是不能继承基类的构造函数。在设计派生类时，对继承过来的成员变量的初始化工作也要由派生类的构造函数完成，但是大部分基类都有 private 属性的成员变量，它们在派生类中无法访问，更不能使用派生类的构造函数来初始化。解决这个问题的思路是：在派生类的构造函数中调用基类的构造函数。

如下，添加People、Student的构造函数，并修改main.cpp。
``` c++
// people.h
...
public:
    People(string, int);
...

// people.cpp
...
People::People(string name, int age) {
    this->name = name;
    this->age = age;
}
...

// student.h
...
public:
    Student(string, int, float);
...

// student.cpp
...
Student::Student(string name, int age, float score) : People(name, age), score(score) {}
...

// main.cpp
#include "student.h"
#include <iostream>

int main(){
    Student stu("Tom", 16, 95.5);
    cout<<stu.getname()<<"'s age is "<<stu.getage()<<", score is "<<stu.getscore()<<endl;
    return 0;
}
```

执行结果如下：
``` bash
$ g++ -o main main.cpp student.cpp people.cpp && ./main 
Tom's age is 16, score is 95.5
```

如第23行所示，用初始化参数列表的方式初始化派生类的构造函数，这是因为基类构造函数不会被继承，不能当做普通的成员函数来调用。换句话说，只能将基类构造函数的调用放在函数头部，不能放在函数体中。

事实上，通过派生类创建对象时必须要调用基类的构造函数，这是语法规定。换句话说，定义派生类构造函数时最好指明基类构造函数；如果不指明，就调用基类的默认构造函数（不带参数的构造函数）；如果没有默认构造函数，那么编译失败。

从上面的分析中可以看出，基类构造函数总是被优先调用，这说明创建派生类对象时，会先调用基类构造函数，再调用派生类构造函数，多级继承也是这样的顺序，最先调用的总是最高基类的构造函数。

### 3.2 析构函数
和构造函数类似，析构函数也不能被继承。与构造函数不同的是，在派生类的析构函数中不用显式地调用基类的析构函数，因为每个类只有一个析构函数，编译器知道如何选择，无需程序员干涉。

另外析构函数的执行顺序和构造函数的执行顺序也刚好相反：
* 创建派生类对象时，构造函数的执行顺序和继承顺序相同，即先执行基类构造函数，再执行派生类构造函数。
* 而销毁派生类对象时，析构函数的执行顺序和继承顺序相反，即先执行派生类析构函数，再执行基类析构函数。


## 虚基类和虚继承

