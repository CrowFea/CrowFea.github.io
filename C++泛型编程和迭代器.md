---
title: C++泛型编程和迭代器
date: 2019-07-19 22:26:45
categories:
    - C++
    - 数据结构
tags: 
    - C++
mp3: http://domain.com/awesome.mp3
cover: http://domain.com/awesome.jpg
---

### 一个例子
首先是我碰到了这样一个问题，要对map容器进行遍历。但是问题在于不知道map内部到底有多少个元组，也没有具体的地址可以进行遍历。所以这里需要使用迭代器。
```c++
map<int,int> m;
m[1]=2;
for(map<int,int>::iterator iter=m.begin();m!=m.end();++iter){
    cout<<iter->first;  //1
    cout<<iter->second; //2
}
```
接下来详细看一下迭代器和其中的泛型编程思想。

### 泛型编程思想
泛型编程的目的是编写独立于数据类型的代码。我们知道有面对对象的编程思路，这是针对于数据的，将数据以对象的的形式进行管理组织。泛型编程是针对于算法，目标是对于多种多样的数据类型，有统一的方式对数据进行操作。在此之前常用的方法是模板，模板使泛型有了可能，在此基础上泛型编程针对于算法更进一步。

其中理解迭代器就能更好地理解泛型编程。

### 为什么使用迭代器
我们在这里看一个例子。

假设我要在一个double类型的数组里寻找一个特定的值，会这样写：
```c++
double find_ar(double *ar,int n,const double &val){
    for(int i=0;i<n;++i){
        if(ar[i]==val)  return &ar[i];
    }
    return 0;
}
```
那么如果现在要在链表里进行同样的操作，就需要这样写：
```c++
struct Node{
    int val;
    Node* next;
}

Node* find_II(Node* head,const double &target){
    Node *p;
    while(p!=NULL){
        if(p->val==target)  return p;
        p=p->next;
    }
    return NULL;
}
```
这里实际上在代码上会是两个不同逻辑的代码，但是广义上来说就是同一个思路：遍历所有的变量，找到目标返回。这个时候我们希望有这样一种东西，对于这些逻辑相同的操作有着同样的运算。


### STL的迭代器
STL遵循上面的方法。

首先对于每个容器，都定义了相应的迭代器类型。其中有可能是指针，也有可能是对象。不管实现方式是怎样的，迭代器都将提供所需的操作，如*或者++。
其次，每个容器类都要有一个超尾标记，当迭代器递增到超越容器后的最后一个值后，这个值会赋给迭代器。

每个容器类都有begin()和end()方法，分别指向第一个元素和超尾位置。每个容器类都有++操作。

实际上，作为一种编程风格，最好避免直接使用迭代器，而应尽可能的使用STL函数比如说for_each()，也可以使用C++ 11新增的基于范围的for循环：
```c++
for(auto=x :nums)   cout<<x<<endl;
```