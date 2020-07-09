---
title: Swift | 关于Array & Dictionary
date: 2020-07-08 20:28:12
categories:
    - iOS
tags: 
    - swift
mathjax: true
---

## 传值还是传引用
在swift中，类的实例 & 函数本身都是引用类型。除了这两个之外，其他都是值类型。这里的其他主要指的是结构体 & 枚举，数组和字符串都是结构体类型的数据。

通常，我们说值类型的传递更安全，其安全的本质其实就是通过copy。这样一想的话就很昂贵，但是swift有两点保证了这不会发生
- let & var 一旦let了，这个值不会变，自然也不用再重复一遍
- copy on write 只有在使用时才会

## Array
### 底层
Array底层是类似于vector的数据结构。主要体现在内存的扩展上，成2倍的扩展。在数据量减小的时候并不会减少分配的空间。
Array可以转换为NSArray，这两者有些许不同。Array本身是结构体类型，NSArray是类类型。一个是引用、一个是值，但是OC内部通过`_ObjectiveCBridgeable`有一个桥接的功能，Array转换为NSArray。

### map之类的方法
函数式编程，在C++中不常用的一种方式。在swift中非常常见。这里大致列举下可能使用的方法。

```swift
let array = [1,2,3,4,5]
//map
let a = array.map { (num) in
    num * num
}
//filter
let b = array.filter { (num) in
    num%2 == 0
}
//allSatisf
let c = array.allSatisfy { (num) in
    num%2==0
}
//reduce
let d = array.reduce(0,+)
let d1 = array.reduce(into: 0){(total,num) in
    total = total + num
}
//forEach
array.forEach { (num) in
    print(num)
}
```

这类方法背后的思想在于，将一些模版化的东西抽出来，开发者只需要去关注真正实现功能的代码即可。比方说一个map操作，自己写的话至少先要一个循环、然后是局部变量的声明等，map解除了这些东西的束缚，开发者只要去对这些数据做处理即可。

这也是一个编程思路，swift提供了对原生API的扩展，当要使用复杂的、重复的代码的时候，考虑能不能对原生的数据结构进行扩展，让代码更加优雅。

但是要注意的是，map filter这些方法是产生一个新的数组，因此在大数组的时候，除非明确的知道要使用所有的元素，再使用这些方法，否则的话看一看是否有别的操作方式。

还要多讨论的一个方法是`forEach`，他和for循环没有本质上的区别，有些情况下使用`forEach`会更加方便。要注意的是，如果循环中要加入return语句，forEach并不会真正的return，而仅仅是退出闭包，因此往往一个循环还是会走完。

### 数组切片
ArraySlice和Array满足的是同一个协议，因此实现的方法基本是相同的。产生数组切片也十分简单
```swift
let array = [1,2,3,4,5]
let slice = array[1...]
```
slice和原数组指向的内存是同一块，操作的时候尽量还是把slice传给新的Array进行处理。

### 其他
尽管Array是支持索引操作的，但是swift仿佛在暗示我们尽量不要这样去做。使用索引的场景往往会意味着循环，我们可以通过别的方式去处理，比如`dropFirst(k: Int)` `dropLast(k: Int)`，或者map + filter来进行一定的过滤。除非你真的确认索引不会出错，进行不要使用这种方式。