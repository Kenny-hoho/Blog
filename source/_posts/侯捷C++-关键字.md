---
title: 侯捷C++-——关键字
date: 2023-12-17
index_img: "/img/bg/cpp.jpg"
tags: [C++]
categories: 
   -[C++]
---

<!-- more -->

# inline（内联函数）

直接定义在类中的函数不用使用 inline关键字也会成为 inline函数，相当于自动添加 inlne关键字，但是是否这个函数真的编程内联函数是完全取决于编译器的。也就是说，我们对一个函数声明他为 inline 只是**建议**编译器将其视作 inline函数，最终还是要由编译器决定（**太复杂的函数编译器没有能力将其变成内联函数**）。

# const关键字

## 简介

Cherno把const称为一个假的关键字，因为它并不会对产生的代码造成什么实质上的影响，它只是一种 承诺 ，承诺这是一个不会被改变的常量，用来方便编程。下面是const的几种用法：

## 修饰函数（只能修饰成员函数）

**如果没有修改参数或者不应该修改参数时，总是把这个方法标记为const！**

```C++
class A{
public:
    A(int x) : value(x) {}
    int GetValue() const {return x;}
private:
    int value;
};
{
    const A a(6); 
    a.GetValue(); // 正确，如果GetValue没有设置为const，编译器会报错，因为GetValue可能会修改常量a
}
```
![](/article_img/2023-12-11-14-37-20.png)
如上面的例子，const修饰函数时，可以确保我们不会修改一个常量。当我们忘记写const时，编译会报错，相当于**强迫**我们思考我们的函数中有没有修改常量，如果设置了const还在函数中进行了对非局部变量的修改，也会报错。

![](/article_img/2023-12-20-11-06-31.png)
如图中右侧的例子，在字符串的实现中，定义了两个操作符重载，一个是const，一个不是，这是因为字符串实现时采用了共享的设计模式（4个字符串对象内容一样会指向同一片内存），共享就导致当要对内容更改时，需要 **COW（Copy On Write）** 先复制再更改，防止影响其他共享对象；那么对于图中的 **[]取** 操作符就有两种情况，一种取出后更改（a[2] = 'a'）要考虑 COW，一种不改只读取不考虑 COW，这两个函数实现会不同，对于**常量字符串**（不能对其进行更改）我们当然希望不要 COW 造成额外的开销，因此就对不考虑 COW 的函数加**const**，让常量字符串**只会**调用该函数。**同时，当const和non-cont同时存在时，非常量对象只能调用non-const版本。**
也可以看出，const也是函数签名的一部分（返回值不是函数签名的一部分），不会造成同名函数编译不过的情况。 
![](/article_img/2023-12-20-11-21-44.png) | ![](/article_img/2023-12-20-11-22-15.png)
---|---

# static关键字

## 类内使用static

![](/article_img/2023-12-17-16-35-22.png)

使用static修饰的变量只有一份（**所有类的实例共享**），使用static修饰的函数称为静态函数，**静态函数没有this指针**，故静态函数不能操作任何非静态变量。对于一般的函数，其函数体当然也只有一份，要操作不同的类实例中的变量，自然就要一个this指针告诉函数体操作的对象是哪一个。

```C++
class Account{
public:
    static double m_rate;
    static void set_rate(const double x){m_rate = x;}
};
double Account::m_rate = 8.0;
int main(){
    Account::set_rate(5.0); // 通过class name调用

    Account a;
    a.set_rate(7.0); // 通过对象调用
}
```

调用static函数的方式有：
1. 通过对象调用
2. 通过类名调用

static应用之一单例模式：
```C++
class A{
public:
    static A& getInstance(){return a;}; // 使用静态函数让外界调用仅有的对象
    setup(){...}
private:
    A();
    A(const A& rhs); // 将构造函数放在private防止外界访问
    static A a; // 将自己声明为static，一开始就会有一个对象
    ...
};
{
    A::getInstance().setup();
}
```
可以对上面实现的单例做进一步优化：
```C++
class A{
public:
    static A& getInstance(); // 使用静态函数让外界调用仅有的对象
    setup(){...}
private:
    A();
    A(const A& rhs); // 将构造函数放在private防止外界访问
    ...
};
A& A::getInstance(){
    static A a;  // 只有第一次调用时才会创建唯一的对象，不调用不会创建
    return a;
}
```

## 类外使用static

类外的static修饰的符号在link阶段是**局部**的，也就是说它只对定义它的 **编译单元（.obj）** 可见。

当链接器（Linker）工作时，他不会找static修饰的变量，如果不是static修饰的全局变量，在link阶段就会随着链接而变成一个“**整个项目的全局变量**”，因为每一个使用到这个全局变量的编译单元都会去其他编译单元中寻找这个全局变量，如果不同的编译单元中定义了命名相同的全局变量，就会在链接阶段报重复定义的错。

同时static修饰的静态变量在离开作用域时，变量生命周期不会结束，**其生命随程序结束而结束**。

# 模板

## 类模板

```C++
template<typename T>
class complex{
public:
    complex(T r=0, T i=0): a(r), b(i) {}
private:
    T a, b;
};
{
    complex<double>c1(2.5, 2.1);
    complex<int>c2(1, 2); // 绑定T为int
}
```

## 函数模板

```C++
template<class T>
inline 
const T& min(const T& a, const T& b){
    return b<a ? b : a;
}
stone r1(2,3), r2(3,3), r3;
r3 = min(r1, r2);  // 不需要告知类型，编译器会自动进行参数推导
```

# namespace

![](/article_img/2023-12-18-10-40-37.png)
