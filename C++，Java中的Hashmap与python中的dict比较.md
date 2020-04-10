---
title: C++，Java中的Hashmap与python中的dict比较
date: 2019-04-18 14:32:45
categories:
    - C++
tags: 
    - C++
mp3: http://domain.com/awesome.mp3
cover: http://domain.com/awesome.jpg
---

### 前言
之前在做leetcode的两数之和，用三种不同的语言进行了研究。思路都是用key-value形式的数据结构：C++、Java是Hashmap，在Python里面是字典。这篇文章我们就分析一下这几种不同语言下的数据结构。

### C++ 中的 map & unoerdered_map
我们这里讨论的结构来源于STL标准模板库。

先说STL：它是于1994年正式成为ANSI/ISO C++的一部份，STL不是面向对象编程，而是一种不同的编程模式——泛式编程。这使得STL在功能和方法方面都很有趣。我们简单的将它的作用理解如下：

STL 将“**在数据上执行的操作”**与“要执行操作的数据分开”，分别以如下概念指代：

- 容器：包含、放置数据的地方。
- 迭代器：在容器中指出一个位置、或成对使用以划定一个区域，用来限定操作所涉及到的数据范围。
- 算法：要执行的操作。

>1994年1月6日，Koenig寄封電子郵件給Stepanov，表示如果Stepanov願意將标准模板库的說明文件撰寫齊全，在1月25日前提出，便可能成為標準C++的一部份。Stepanov回信道：“Andy, are you crazy?” 。 Koenig便說：“Well, yes I am crazy,but why not try it?”。

在STL的容器内部有11种通用容器，其中包括7种序列容器和4种关联容器。关联容器全部是key-value形式的数据结构：map、set、multimap、multiset。

- set：值和键类型相同，且只能有唯一的值 [1,2,3]
- multiset:值和键类型相同，值可以存在相同的 [1,1,2,2,3]
- map:值和键类型可以不同，值只能与唯一的键关联 [[1,2],[2,3],[3,4]]
- multimap:值和键类型可以不同，值可与多个键关联 [[1,2],[1,3],[3,4]]

set, multiset, map, multimap内部采用的就是一种非常高效的平衡检索二叉树：红黑树(Red-Black Tree，也称AVL树)。RB树的统计性能要好于一般的平衡二叉树。使用红黑树操作的复杂度在O(logN),但搜索的过程是保持动态且有序的。这就意味着使用红黑树的数据结构可以在顺序上进行更多的操作，并且计算的复杂度随key值的变化波动不大，可以稳定在O(logN)。

在c++ 11中添加了一个新的序列容器：unordered_map。使用的是散列进行哈希，时间复杂度为O(1)。但取而代之的是空间消耗更大，同时不能提供一个稳定的输出，随着key值算法的时间波动较大。关于两者的性能看到了这篇文章写的很好 [Sam-Cen的博客]https://blog.csdn.net/blues1021/article/details/45054159。

unordered_map 查找效率快五倍，插入更快，节省一定内存。如果没有必要排序的话，尽量使用 hash_map(unordered_map 就是 boost 里面的 hash_map 实现)。

### Java中的HashMap & TreeMap
java里面的HashMap是根据map接口的一个实现，我们直接从JDK里来看HashMap的构造：
```java
public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        // Find a power of 2 >= initialCapacity
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;

        this.loadFactor = loadFactor;
        threshold = (int)Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        table = new Entry[capacity];
        useAltHashing = sun.misc.VM.isBooted() &&
                (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
        init();
}
```
其中table就是我们得到的散列表，在第 18 行
```java
table = new Entry[capacity];
```
在构造函数中，创建了一个 Entry 的数组，其大小为 capacity。再让我们看一下Entry的结构：
```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    final int hash;
   
}
```
Entry 是一个 static class，其中包含了 key 和 value，也就是键值对，另外还包含了一个 next 的 Entry 指针。也就是说，在java内部散列的过程，采用的是链地址法：在哈希表每个单元中设置链表。某个数据项的关键值仍然映射到哈希表的单元中，而数据项本身插入这个单元的链表中其他同样映射到这个位置的数据项只需要加入到链表中。时间复杂度平均能达到O(1)。

同样的，Java中也存在以红黑树为原型的key-value数据结构，称为TreeMap。其原理与特性与C++中的map类似，我们略去不表。只是将Java的两种数据结构的调用方法陈列如下：
```java
//初始化
HashMap<String,Integer> hashMap = new HashMap<>();
//添加。如果key不存在，则返回null
hashMap.put("key",value);
//添加。若有则不添加。
hashMap.putIfAbsent("key",value);
//删除元素
hashMap.remove("key");
hashMap.remove("key",value);
//获取元素
hashMap.get("key")
//元素遍历
Iterator iterator = hashMap.keySet().iterator();
while (iterator.hasNext()){
   String key = (String)iterator.next();
   System.out.println(key+"="+hashMap.get(key));
}
//或者：
Iterator iterator1 = hashMap.entrySet().iterator();
while (iterator1.hasNext()){
    Map.Entry entry = (Map.Entry) iterator1.next();
    String key = (String) entry.getKey();
    Integer value = (Integer) entry.getValue();
    System.out.println(key+"="+value);
}
//判断key或value是否存在
hashMap.containsKey("key");
hashMap.containsValue(value);
```

### Python中的字典
对于Python来说，字典就是数组形式的哈希表，数据结构为数组，采用了哈希的散列，解决冲突的办法为开放寻址法。

神奇的一点是，python在散列的过程中，会根据当前表的疏松程度进行空间的再申请。当使用量超过总槽数的2/3，就会申请不少于当前活动槽数4倍的新空间。统计的来说，最后得到的表所利用的空间达不到一开始的1/3。这在空间上的损耗是巨大的，但是以空间换时间，在时间上的复杂度为O(1)。

Python里面的字典方法：
```python
# 初始化
info = dict()
# 添加
>>> info['name'] = 'cold'
>>> info['blog'] = 'linuxzen.com'
>>> info
{'blog': 'linuxzen.com', 'name': 'cold'}
>>> info
{'blog': 'linuxzen.com', 'name': 'cold night'}
# 删除
del info['name']
# 遍历
>>> info = dict(name='cold', blog='linuxzen.com')
>>> for key, value in info.items():
...     print key, ':',  value
```

### 总结
总结看下来，基本上key-value形式的数据结构分为两类：

- 以红黑树为基础的数据结构：map、set、TreeMap
- 基于散列哈希的数据结构：HashMap、dictionary

前者的优势在于有序，能提供较为稳定的复杂度算法，并且可以在顺序上做文章。但后者更快，在大数据的情况下更实用。

最后值得注意的是，红黑树为基础的数据结构内部重载了小于的操作（因为红黑树的插入本身要先排序）；而哈希重载了等于的操作（哈希通过判断相等来确定要不要处理冲突）。在自定义时记得自己声明重载。