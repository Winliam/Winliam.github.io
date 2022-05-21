---
title: C++类继承机制中的作用域嵌套问题
categories:
  - - 工作技能
    - C/C++笔记
date: 2021-11-08 17:58:00
tags:
  - 继承
  - 动态绑定
  - 虚函数
  - 作用域
---
## 前言
OOP的三个支撑概念的其中之二，继承和动态绑定，都会带来作用域模糊的问题，这里来梳理以下。

## 一般规则
给定一个类的实例/类的实例的指针/类的实例的引用`p`，现在要通过`p`来访问它的对象`mem`。`mem`可以是方法也可以是成员，不影响下面的逻辑展开。

整个调用过程，可以用下面的流程图表示：

<div align=center><img title="" src="/img/article/cpp_herit.jpg" width="80%" height="80%" align=center></div>

<br>注意几个要点：
1. 名称的搜索只会向父类递归，不会向子类递归。
2. 名称查找成功后，直接进行类型检查，不成功直接报错，而不是继续递归查找。

## 补充案例
### 案例一
考虑一个三重继承A->B->C，当A和C之间的动态绑定发生时，如C没有override这个被A指针调用的虚函数，但是B override了，那么最终被调用的将是B的版本。也就是说动态绑定的优先级是动态类型，然后动态类型的直接基类，然后直接基类的直接基类，依次类推。

### 案例二
考虑在一对虚函数中，子类的虚函数函数体中调用了另外一个成员函数。如果这个成员函数涉及到动态绑定，那么按照案例一中的优先级，寻找最合适的虚函数版本即可。如果不涉及动态绑定，那么应该按照“子类作用域嵌套在父类作用域内”的原则取选取最合适的函数版本。

```
#include <iostream>

using namespace std;

class A {
private:
    int a = 1;
public:
    void f1(int x){
        std::cout << "this is class A, f1 " << x << std::endl;
    }
    virtual void f2(){
        std::cout << "this is class A, f2" << std::endl;
    }
};
class B : public A{
private:
    int b = 2;
public:
    void f1(int x){
        std::cout << "this is class B, f1 " << x << std::endl;
    }
    void f2(){
        std::cout << "this is class B, f2" << std::endl;
        f1(10);
    }
};
class C : public B{
private:
    int c = 3;
public:
    void f1(int x){
        std::cout << "this is class C, f1 " << x << std::endl;
    }
    void f2(){
        std::cout << "this is class C, f2" << std::endl;
        f1(10);
    }
};

int main() {
    A* pa = new C;
    pa->f2();
    delete pa;
    return 0;
}
```
