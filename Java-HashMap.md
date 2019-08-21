---
title: Java-HashMap
date: 2018-01-29 13:55:25
categories: 
- Java
tags:
- Java
---

## 摘要（未完）

查看了一下HashMap源代码，一些基本知识点

参考资料里面其实写的很好，建议看参考链接

<!--more-->

## 概述

它根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。 

 HashMap 大概具有以下特点：

- 底层实现是 链表数组，JDK 8 后又加了 红黑树
- 实现了 Map 全部的方法
- key 用 Set 存放，所以想做到 **key 不允许重复**，key 对应的类需要重写 hashCode 和 equals 方法
- 允许空键和空值（但空键只有一个，且放在第一位）
- 元素是无序的，而且顺序会不定时改变
- 插入、获取的时间复杂度基本是 O(1)（前提是有适当的哈希函数，让元素分布在均匀的位置）
- 遍历整个 Map 需要的时间与 桶(数组) 的长度成正比（因此初始化时 HashMap 的容量不宜太大）
- 两个关键因子：初始容量、加载因子

除了不允许 null 并且同步，Hashtable 几乎和他一样。

## HashMap内部结构

JDK1.7 中hashmap 是通过 桶（数组）加链表的数据结构来实现的。当发生hash碰撞的时候，以链表的形式进行存储。

JDK1.8 中hashmap 增加了在原有桶（数组） + 链表的基础上增加了黑红树的使用。当一个hash碰撞的数量超过指定次数(TREEIFY_THRESHOLD)的时候，链表将转化为红黑树结构。

### 负载因子

默认为0.75。一般不建议修改，

- 加载因子太大的话发生冲突的可能就会大，查找的效率反而变低，如果内存空间紧张而对时间效率要求不高可增加。
- 太小的话频繁 rehash，导致性能降低，如果内存空间很多而又对时间效率要求很高可

### 容量

#### 默认

桶数默认为16，非常规设计（一般采用素数），主要是为了在取模和扩容时做优化，同时为了减少冲突。

#### 设定

在构造hashmap时，如果你传入的是一个自己设定的数字，它会调用`tableSizeFor`帮你转换成一个比传入容量参数值大的最小的**2的n次方**（所以不管你传入什么数字最终还是2的幂）。

这里tableSizeFor如果你看过源代码的话，会发现用的是挺晦涩的位运算（强烈吐槽），下面一大堆的运算你可以理解为把这个数字的所有位都变为1，再加上1，就能得到比传入容量参数值大的最小的**2的n次方**。

```java
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

### MAXIMUM_CAPACIT

上面的`MAXIMUM_CAPACIT`的值是`1 << 30`

这个设定应该是考虑到int是4个字节，每个字节8位，就是32位，然后有一位是表示正负的，整数最大是(2^31)-1，但是选择`小于等于2^30次`而不是`小于2^31`，网上翻了一圈没找到答案，我姑且觉得就是其实无论是2^30也好，2^31也好，其实只是表示一个大概的峰值，实际项目中基本不会出现这种需求，所以具体数字是什么无所谓。

### Hash

HashMap 中通过将`传入键的 hashCode` 和`此值按位右移 16 位补零后的值`进行按位异或，得到这个键的哈希值。

```java
(h = key.hashCode()) ^ (h >>> 16);
```

这里是由于，由于哈希表的容量都是 **2 的 N 次方**（最大2^30），在当前，元素的 hashCode() 在很多时候下低位是相同的，这将导致冲突（碰撞），因此 1.8 以后做了个移位操作：将元素的 hashCode() 和自己右移 16 位后的结果求异或。

由于 int 只有 32 位（4*8），无符号右移 16 位相当于把高位的一半移到低位：

![ddd](http://img.blog.csdn.net/20160408155045341)

### 扩容





## 参考资料

[Java 8系列之重新认识HashMap](https://tech.meituan.com/java-hashmap.html)
