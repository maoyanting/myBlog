---
title: Java-Java基础
date: 2018-10-08 21:47:46
categories: 
- Java
tags:
- Java
- 面试
---



## 摘要

本文是对一些java基础知识的整理，把之前印象笔记里面的全部慢慢搬到这个blog来

为了方便就按照《thinking in Java》的目录来编辑。

>这里面的内容均为面试题相关，可能的考点等

这本书里面有些翻译不是很好，建议和英文版对照。

提醒：文章较长，因为很早写的，罗列的知识点较多，在慢慢把知识点摘取出去以链接形式存在。

<!--more-->

***

## Chapter1-Introduction to Objects

### 特性

抽象、封装、继承、多态

面向对象设计方法主要特征：继承、封装、多态。

>延伸点：反射会破坏代码的封装性（mock等）

***

## Chapter2-Everything Is an Object

### Object方法

| return            | method                        | 解释                                                         |
| ----------------- | ----------------------------- | ------------------------------------------------------------ |
| protected  Object | clone()                       | 创建并返回此对象的一个副本。                                 |
| protected  void   | finalize()                    | 当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。 |
| void              | notify()                      | 唤醒在此对象监视器上等待的单个线程。                         |
| void              | notifyAll()                   | 唤醒在此对象监视器上等待的所有线程。                         |
| void              | wait()                        | 在其他线程调用此对象的 notify()方法或 notifyAll()方法前，导致当前线程等待。 |
| void              | wait(long timeout)            | 在其他线程调用此对象的 notify()方法或 notifyAll()方法，或者超过指定的时间量前，导致当前线程等待。 |
| void              | wait(long timeout, int nanos) | 在其他线程调用此对象的 notify()方法或 notifyAll()方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量前，导致当前线程等待。 |

方法属于对象的成员，静态方法属于类的成员

>Q:为什么 wait,  notify 和 notifyAll 这些方法不在 thread 类里面？
>
>一个很明显的原因是 JAVA 提供的锁是「对象级」 的而不是「线程级」的，每个对象都有锁，通过线程获得。如果线程需要等待某些锁那么调用对象中的 wait()方法就有意义了。
>
>如果 wait()方法定义在 Thread 类中，线程正在等待的是哪个锁就不明显了。
>
>简单的说，由于 wait，notify 和 notifyAll 都是锁级别的操作，所以把他们定义在 Object 类中因为锁属于对象。
>
>拓展：
>在 Java 中，每个对象都有两个池，锁（monitor）池和等待池
>
>1. 锁池：假设线程A已经拥有了某个对象（注意：不是类）的锁，而其它的线程想要调用这个对象的某个synchronized 方法（或者 synchronized 块），由于这些线程在进入对象的 synchronized 方法之前必须先获得该对象的锁的拥有权，但是该对象的锁目前正被线程A拥有，所以这些线程就进入了该对象的锁池中。
>
>2. 等待池：假设一个线程A调用了某个对象的 wait()方法，线程A就会释放该对象的锁（因为wait()方法必须出现在 synchronized 中，这样自然在执行 wait()方法之前线程A就已经拥有了该对象的锁），同时线程A就进入到了该对象的等待池中。
>
>   如果另外的一个线程调用了相同对象的 notifyAll()方法，那么处于该对象的等待池中的线程就会全部进入该对象的锁池中，准备争夺锁的拥有权。如果另外的一个线程调用了相同对象的 notify()方法，那么仅仅有一个处于该对象的等待池中的线程（随机）会进入该对象的锁池.

***

>Q:为什么 wait 和 notify 方法要在同步块中调用？
>
>1. 避免 IllegalMonitorStateException 
>2. 避免任何在 wait 和 notify 之间潜在的竞态条件
>
>参考资料：[并发编程之 wait notify 方法剖析](https://hacpai.com/article/1514818121670)                





### 基本数据类型

Integer，Float，Double等都继承自Number类 

| 名称            | 包装类型  | 初始化         | 其他                                                         |
| --------------- | --------- | -------------- | ------------------------------------------------------------ |
| boolean         | Boolean   | false          | 1位                                                          |
| byte（1字节）   | Byte      | (byte)0        | ASCII码：空格：32；数字0到9：48到57，后面为大小写字母； byte 类型用在大型数组中节约空间，主要代替整数，因为 占用的空间只有 int 类型的四分之一 |
| short（2字节）  | Short     | (short)0       | Short 数据类型也可以像 byte 那样节省空间。一个short变量是int型变量所占空间的二分之一； |
| char（2字节）   | Character | '\u0000'(null) | 单一的 16 位 Unicode 字符；                                  |
| int（4字节）    | Integer   | 0              |                                                              |
| float（4字节）  | Float     | 0.0f           | 精确到小数点后6位，在储存大型浮点数组的时候可节省内存空间；  |
| long（8字节）   | Long      | 0L             |                                                              |
| double（8字节） | Double    | 0.0d           | 精确到小数点后15位（默认的小数类型）                         |

BigInteger 任意精度的整数，不可变

BigDecimal 任何精度的定点数（商业计算），不可变，通常建议优先使用String构造方法

>参考：[Java BigDecimal详解](http://blog.csdn.net/jackiehff/article/details/8582449)
>
>延伸点：[为什么int整型(32位)的范围是-32768到32767？](https://my.oschina.net/lcniuren33/blog/63762)

parseInt()是把String变成int的基础数据类型； 

#### 类型转换

byte,short,char-> int -> long -> float -> double  不同类型运算结果类型向右边靠齐。

浮点数到整数的转换是通过舍弃小数得到，而不是四舍五入

***

#### 拆箱和装箱

**装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的**。

大多数情况下两者相同，java.io.file,java.util.Date,java.lang.string,包装类（Integer,Double等）等实现了自己的equals，比较规则为：如果两个对象的类型一致，并且内容一致，则返回true。



`valueOf() ` 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。 

在 Java 8 中，`Integer ` 缓存池的大小默认为 -128~127，所以除非用 `new Integer()` 显式的创建对象，否则都是同一个对象（-128～127）

| 比较符号 | 比较对象         | 数据转型       |
| -------- | ---------------- | -------------- |
| ==       | 同一个对象       | 有表达式会触发 |
| equals   | 同一个类型和内容 | 不触发         |



>一般会考你“==”和equals的区别，数据转型，拆箱装箱
>
>“==”和equals记住，前者相同，后者相等。前者会触发数据转型。
>
>new Integer()新建对象
>
>valueof装箱，xxxValue拆箱


### Math类

| 方法      | 功能               | 返回值    |
| ------- | ---------------- | ------ |
| ceil()  | 向上取整             | double |
| floor() | 向下取整             | double |
| round() | 原来的数字加上0.5后再向下取整 | int    |

### 变量名

三种元素构成：数字+字符+$+下划线。 

不能以数字开头+不能是关键字. 

### 类名

一个文件中可以有多个类，如果没有public类，则可以与任意类名相同，如果有public类则文件名必须与此类名相同，因为一个文件中只能有一个public类。如果文件中只有一个类，则文件名必须与类名相同

### 注释

| 名称              | 功能                    |
| --------------- | --------------------- |
| Override        | 标识当前的方法，是否覆盖了它的父类中的方法 |
| Deprecated      | 已经过期或者即将过期的方法，不推荐我们使用 |
| SuppressWarning | 屏蔽相应的警告               |

***

## Chapter3-Operators

| 运算符                  | 结合性  | 其他                      |
| -------------------- | ---- | ----------------------- |
| () [] .              | 从左到右 |                         |
| ! +(正)  -(负) ~ ++ -- | 从右向左 | ~n=-n-1                 |
| * / %                | 从左向右 | %取余操作，只适用于整型            |
| +(加) -(减)            | 同上   |                         |
| << >> >>>            | 同上   | 按位左（右）移运算符， 按位右移运补零运算符。 |
| < <= > >= instanceof | 同上   |                         |
| ==   !=              | 同上   |                         |
| &                    | 同上   | 相对应位都是1，则结果为1，否则为0      |
| ^                    | 同上   | 相对应位值相同，则结果为0，否则为1      |
| 或                    | 同上   | 相对应位都是0，则结果为0，否则为1      |
| &&                   | 同上   | 逻辑与（前面表达式为false，后面不计算）  |
| 逻辑或                  | 同上   | 逻辑或（前面表达式为true，后面不计算）   |



***

## Chapter4-Controlling Execution

### break continue return

1. break    **中断并跳出当前循环**。直接跳出当前的循环，从当前循环外面开始执行,忽略循环体中任何其他语句和循环条件测试。他只能跳出一层循环，如果你的循环是嵌套循环，那么你需要按照你嵌套的层次，逐步使用break来跳出.  


2. continue **终止当前的循环过程**，但他并不跳出循环,而是继续往下判断循环条件执行语句.他只能结束循环中的一次过程,但不能终止循环继续进行.     
3. return **导致当前的方法退出**，并返回那个值。

### switch

选择因子：整数类型（**long除外**），enum（jdk1.5后），String（jdk1.7后）

字符串的switch是通过equals()和hashCode()方法来实现的。

注意break

***

## Chapter5-Initialization & Cleanup

类的加载包括：加载，验证，准备，解析，初始化

构造方法内可以用this调用另一个构造方法

### 重载与重写

同名不同参数为**重载**，**与返回值类型无关**；（比如你调用这个方法的时候java怎么根据返回值来区分你实际调用的方法）与访问修饰符无关 

子类覆写父类的方法为**重写**

#### 方法重写应遵循“三同一小一大”原则：

1. ​    “三同”：即方法名相同，形参列表相同，返回值类型相同；    
2. ​    “一小”：子类方法声明抛出的异常比父类方法声明抛出的异常更小或者相等；    
3. ​    “一大”：子类方法的访问**修饰符**应比父类方法更大或相等。  

>题型：怎么算重载和重写，重写的修饰符范围
>
>final不能重写



***

## Chapter6-access control

### 访问控制符

| 名称        | 功能                                       |
| :-------- | :--------------------------------------- |
| 包访问权限     | 不提供任何权限修饰词情况下；默认包：处于同一个文件目录下             |
| public    | 都可用                                      |
| protected | 修饰方法和成员变量，处理继承的概念，保证了仅有**继承的类**才能访问这个成员。同时具有**包访问权限**。 |
| private   | 仅在被包含的类里面可用，类的变量通常声明为private             |

❤class编写顺序可按照：public👉protected👉private

类：❌（private & protected），⭕️（包访问权限 or public）



静态变量会默认赋初值，局部变量和final声明的变量必须手动赋初值





***

## Chapter7-ReusingClasses

### 组合

将对象置于新类中即可

想在新类中使用现有类的功能而非它的接口

### 继承

**构造函数不继承**，只能显式（super）或者隐式的调用，导出类的构造器会插入对基类构造器的调用

**方法（包括私有）均继承**，但是不能直接调用私有方法。

ps：也可以使用super访问父类被子类隐藏的变量或覆盖的方法。



### 代理

将一个成员对象置于构造的类中，同时在新类中暴露了该成员的所有方法。

例：代理类中创建原类的对象，代理类的方法中调用此对象的方法。（其实就是套个壳，间接调用原类的方法）

初始化：1、定义的时候初始化

​               2、构造器初始化

​               3、惰性初始化（在使用这些对象之前）

​               4、实例初始化

只有命令行调用的main方法会被调用



组合和继承（has a & is a ）：

组合通常适用于在新类中使用现有类的功能而非接口，即在新类中嵌入一个现有类的private对象。

继承通常用于，使用某个类，并开发一个它的特殊版本。是否需要从新类向基类进行向上转型

### final, finally, finalize 

**final**

修饰数据（数值不变）

修饰引用（引用不变但是对象自身可以改变）

❌数组中的数据无法被修饰为不可更改

空白final：被声明为final但是没有给定初始值的域，**允许但是必须在域的定义处或者每个构造器中对final赋值**

final 的方法**不能被修改**（继承类中不能覆盖），final类**不能继承**（里面的方法也是final的，数值不受影响）

**private final **修饰的方法，继承类中能覆盖（因为方法为private 的时候，它就不是接口的一部分，导出类中相同名称的方法并不是覆盖而是重新生成了一个新的方法）；



**finally**

一句话简介：确保代码一定会被执行

```java
try{
	do something;
	return;
}finally{
	do A;
}
//do A在代码return之前是一定会被执行的，不过还是有情况是不执行的
//详细见参考资料
```

参考资料：[Java finally语句到底是在return之前还是之后执行？](https://www.cnblogs.com/lanxuezaipiao/p/3440471.html)



确保正确清理：使用finally语句保证清理／执行类的所有特定清理动作，其顺序同生成顺序相反／不要使用finalize（）

**finalize()** 

一句话简介：对于覆盖了 finalize() 方法的对象，垃圾回收会特殊处理，回收之前先调用finalize()方法

具体请看：[Java将弃用finalize()方法？](http://www.infoq.com/cn/news/2017/03/Java-Finalize-Deprecated)

### static

用来修饰成员变量和成员方法，也可以形成静态static代码块。不能在方法中定义

static变量在第一次使用的时候初始化，只会有一份成员对象。  



### 初始化顺序

总结：（静态变量、静态初始化块）>（变量、初始化块）>构造器

***

## Chapter8-Polymorphism

### 向上转型

针对非静态方法是**编译看左，运行看右**，但是对于成员变量，都是看左，也就是父类



它通常只是为了方便输出，比如System.out.println(xx)，括号里面的“xx”如果不是String类型的话，就自动调用xx的toString()方法

Java中除了static和final方法以外，其他方法都是后期绑定（动态绑定）实现多态



动态绑定仅仅针对类的方法





***

## Chapter9-Interfaces

### 抽象类：abstract 

1、抽象的方法**没有body**

2、**含有抽象方法的类就是抽象类**

3、**不能创建实例**

ps：抽象类的继承类，如果没有为所有方法提供方法体，编译器会强制用abstract来修饰这个类。



#### jdk1.8更新

  JDK 1.8以前，抽象类的方法默认访问权限为protected 

  JDK 1.8时，抽象类的方法默认访问权限变为default 

***

### 接口：interface

1. **只有public、static、final变量和抽象方法**


2. 可以在此之前加public，implements关键字，**必须实现所有方法**


3. 先继承再实现
4. 不能创建实例

#### jdk1.8更新

在jdk1.8之前，interface之中可以定义常量和方法，**常量必须是public、static、final的，方法必须是public、abstract的**。由于这些修饰符都是默认的

JDK8及以后，允许我们在接口中定义static方法和default方法，**有方法体**

静态方法，只能通过接口名调用，不可以通过实现类的类名或者实现类的对象调用。

default方法，只能通过接口实现类的对象来调用。可以被覆写，但是不能再加default修饰符。

 JDK 1.9时，接口中的方法可以是private的 

>接口的修饰符，有无body，接口和抽象类的区别等





***

## Chapter10-Inner Classes

### 匿名内部类：

1、不能有**static**变量和字段

2、尾部有分号

3、传入的参数是final

4、**不含命名构造器**，实例初始化的实际效果就是构造器

5、只能实现一个接口，且不能同时继承和实现接口

局部内部类：在方法体里面创建

需要不止一个该内部类的时候使用。局部内部类含有构造器，匿名内部类只能用于实例初始化

### 内部类：

1、不能有static变量和字段

2、构造器必须连接到指向其外围对象的引用

3、外围类继承的时候，内部类并没有被继承。即使基类和导出类的内部类名称相同也是不同且独立的类。

常用：外部类有个方法指向内部类的引用（匿名类把内部类直接放到方法里面）

内部类对外部类引用：外部类.this（嵌套类没有）

创建内部类的对象：外部类对象.new  内部类名（）

### 嵌套类：

static的内部类

可以作为接口的一部分



静态内部类可以访问外围类的静态数据，包括私有数据，但不能访问非静态数据；

非静态内部类可以直接访问外围类的数据，包括私有数据



***

## Chapter11-Holding YourObjects

### Collection

| 方法                                       | 功能                                 |
| :--------------------------------------- | :--------------------------------- |
| Collections.synchronizedXxx()            | 将指定的集合包装成线程同步的集合                   |
| collection1.containsAll（Collection2）     | 合并两个collection                     |
| Collection.addAll(collection,11,12,13,14) | 接受一个collection对象，以及一个数组或者是用逗号分割的列表 |
| Collections.sort(list);                  | 根据元素的自然顺序 对指定列表按升序进行排序。            |
| Collections.shuffle(List<?>     list, Random rand); | 打乱                                 |

arrays.aslist()接受一个逗号分隔的元素列表（可变参数）或者数组，并转换为一个list对象

![ddd](https://uploadfiles.nowcoder.net/images/20160827/6316247_1472307925697_BD4D65F4FBC2294A775409EAC3802943)

### List

#### ArrayList() :

代表长度可以改变得数组。可以对元素进行随机的访问，插入与与删除元素的速度慢。

默认大小为10，不够再以1.5倍扩容新数组，拷贝数据到新数组中

#### LinkedList() 

在实现中采用链表数据结构。插入和删除速度快，随机访问速度慢。

| method                                                       | 解释               |
| ------------------------------------------------------------ | ------------------ |
| subList（List2）                                             | 创建片段           |
| indexOf（object）                                            | 获取索引编号       |
| containsAll（List2）                                         | 是否包含，无视顺序 |
| addFirst／add／addLast                                       | 添加单个元素       |
| remove／removeFirst／poll／移除并返回列表头  removeLast 移除并返回列表尾 |                    |
| getFirst／element／peek                                      |                    |
| peek／poll 列表空返回null                                    |                    |
|                                                              |                    |

>详细参考：[LinkedList基本用法](http://blog.csdn.net/i_lovefish/article/details/8042883)
>
>一个是数组，一个是链表，两者的读写属性基本相反。
>
>延伸点：[java中 CopyOnWriteArrayList 的使用](http://www.importnew.com/20840.html)



***

### Iterator

单向移动：

1. 使用方法iterator()要求容器返回一个Iterator。第一次调用Iterator的next()方法时，它返回序列的第一个元素。注意：iterator()方法是java.lang.Iterable接口,被Collection继承。
2. 使用next()获得序列中的下一个元素。
3. 使用hasNext()检查序列中是否还有元素。
4. 使用remove()将迭代器新返回的元素删除。

```java
    while(it.hasNext())
    {
        Pet p = it.next();
        print(p);
    }
```

>延伸点：可能会引起ConcurrentModificationException 异常



ListIterator：双向移动

| method          | 解释                                                         |
| --------------- | ------------------------------------------------------------ |
| nextIndex()     | 返回列表中ListIterator所需位置后面元素的索引                 |
| previousIndex() |                                                              |
| previous()      |                                                              |
| next（）        |                                                              |
| listIterator(n) | 一开始就指向列表索引为n处的ListIterator，空则指向开始处      |
| remove()        | 对迭代器使用hasNext()方法时，删除ListIterator指向位置后面的元素；当对迭代器使用hasPrevious()方法时，删除ListIterator指向位置前面的元素 |

>延伸：[JAVA中ListIterator和Iterator详解与辨析](http://blog.csdn.net/longshengguoji/article/details/41551491)



***

### Set：

不重复，测试归属性（查找是否存在set中）

和collection接口相同

***

### Map：

Map**没有继承**于Collection接口

Map集合中的键对象**不允许重复**，也就说，任意两个键对象通过equals()方法比较的结果都是false，但是可以将任意多个键独享映射到同一个值对象上。

​    

Object put(Object key, Object value):

Object remove(Object key)

void putAll(Map t):   将来自特定映像的所有元素添加给该映像

void clear(): 从映像中删除所有映射

Object get(Object key): 获得与关键字key相关的值

containsKey()和containsValue()测试Map中是否包含某个“键”或“值”。

| 名称                | 线程安全 | 功能                                       |
| :---------------- | :--: | :--------------------------------------- |
| HashMap           | 不安全  | 基于散列表的实现。插入和查询“键值对”的开销是固定的。可以通过构造器设置容量capacity和负载因子load factor，以调整容器的性能。 |
| LinkedHashMap     | 不安全  | 类似于HashMap，但是迭代遍历它时，取得“键值对”的顺序是其插入次序，或者是最近最少使用(LRU)的次序。只比HashMap慢一点。而在迭代访问时发而更快，因为它使用链表维护内部次序。 |
| TreeMap           | 不安全  | 基于红黑树数据结构的实现。查看“键”或“键值对”时，它们会被排序(次序由Comparabel或Comparator决定)。TreeMap的特点在于，你得到的结果是经过排序的。TreeMap是唯一的带有subMap()方法的Map，它可以返回一个子树。 |
| WeakHashMap       | 不安全  | 弱键(weak key)Map，Map中使用的对象也被允许释放: 这是为解决特殊问题设计的。如果没有map之外的引用指向某个“键”，则此“键”可以被垃圾收集器回收。 |
| IdentifyHashMap   | 不安全  | 使用==代替equals()对“键”作比较的hash map。专为解决特殊问题而设计。 |
| ConcurrentHashMap |  安全  | 将Hash表分为16segment，每次只需要的segment进行加锁      |

***

#### HashMap

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

##### HashMap内部结构

JDK1.7 中hashmap 是通过 桶（数组）加链表的数据结构来实现的。当发生hash碰撞的时候，以链表的形式进行存储。

JDK1.8 中hashmap 增加了在原有桶（数组） + 链表的基础上增加了黑红树的使用。当一个hash碰撞的数量超过指定次数(TREEIFY_THRESHOLD)的时候，链表将转化为红黑树结构。

###### 负载因子

默认为0.75。一般不建议修改，

- 加载因子太大的话发生冲突的可能就会大，查找的效率反而变低，如果内存空间紧张而对时间效率要求不高可增加。
- 太小的话频繁 rehash，导致性能降低，如果内存空间很多而又对时间效率要求很高可



###### 容量

桶数默认为16，非常规设计（一般采用素数），主要是为了在取模和扩容时做优化，同时为了减少冲突。

>关于桶数，在构造hashmap时，如果你传入的是一个自己设定的数字，它会调用tableSizeFor帮你转换成一个比传入容量参数值大的最小的**2的n次方**（所以不管你传入什么数字最终还是2的幂）。
>
>这里tableSizeFor如果你看过源代码的话，会发现用的是挺晦涩的位运算（强烈吐槽），那一大堆的运算你可以理解为把这个数字的所有位都变为1，再加上1，就能得到比传入容量参数值大的最小的**2的n次方**。
>
>默认值为16应该是

###### hash

HashMap 中通过将`传入键的 hashCode` 和`此值按位右移 16 位补零后的值`进行按位异或，得到这个键的哈希值。

```java
(h = key.hashCode()) ^ (h >>> 16);
```

这里是由于，由于哈希表的容量都是 **2 的 N 次方**（最大2^30），在当前，元素的 hashCode() 在很多时候下低位是相同的，这将导致冲突（碰撞），因此 1.8 以后做了个移位操作：将元素的 hashCode() 和自己右移 16 位后的结果求异或。

由于 int 只有 32 位（4*8），无符号右移 16 位相当于把高位的一半移到低位：





##### ConcurrentHashMap

使用segment来分段和管理锁，segment继承自ReentrantLock，因此ConcurrentHashMap使用ReentrantLock来保证线程安全。

JDK1.8之前采用分段锁，核心就是一句话：尽量降低同步锁的粒度。

JDK1.8之后使用CAS思想代替冗杂的分段锁实现。

不出意料，面试者答出CAS之后必定会被追问其思想以及应用，换做我自己的话会有如下思路作答：CAS采用乐观锁思想达到lock free，提一下sun.misc.Unsafe中的native方法，至于CAS的其他应用可以聊一聊Atomic原子类和一些无锁并发框架（如Amino），提到ABA问题加分。

##### Hashtable

都实现了Map接口，主要的区别有：

1. HashMap是非synchronized的，接受null而。
2. 另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。
3. Hashtable在单线程环境下比HashMap要慢。
4. HashMap不能保证随着时间的推移Map中的元素次序是不变的。

***

### Queue：

先进先出

***

### 线程安全

  喂（Vector）  S（Stack）  H（hashtable）  E（enumeration） 

***

## Chapter12-Error Handling with Exceptions

> 抛InterruptedException的代表方法有：
>
> - java.lang.Object 类的 wait 方法
> - java.lang.Thread 类的 sleep、join方法

![dd](http://img.my.csdn.net/uploads/201211/27/1354020417_5176.jpg)

1. 受检查的异常(checked exceptions)，其必须被  try{}catch语句块所捕获,或者在方法签名里通过throws子句声明.受检查的异常必须在编译时被捕捉处理,命名为 Checked  Exception 是因为Java编译器要进行检查，Java虚拟机也要进行检查,以确保这个规则得到遵守. 
2. 运行时异常(runtime exceptions)，需要程序员自己分析代码决定是否捕获和处理,比如  空指针,被0除... 
3. 而声明为Error的，则属于严重错误，如系统崩溃、虚拟机错误、动态链接失败等，这些错误无法恢复或者不可能捕捉，将导致应用程序中断，Error不需要捕捉。 


>[Java ConcurrentModificationException 异常分析与解决方案](http://blog.csdn.net/androiddevelop/article/details/21509345)



### 捕获异常

catch匹配到第一个符合的处理程序就结束

### 自定义异常

可以重写构造器，传入一个字符串

调用printStackTrace()可以打印出异常

### 异常的限制：                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           

构造器：派生类构造器不能捕获基类构造器抛出的异常／派生类构造器的异常说明必须包含基类构造器的异常说明。

如果基类构造器没有异常说明，派生类能有。

方法：派生类方法遵守基类方法的异常说明，但是也可以不抛出异常。

如果基类方法没有异常说明，派生类也不能有。

对象：派生类的对象只被要求捕获这个类所抛出的异常／基类的对象只被要求捕获基类所抛出的异常

异常匹配：抛出的异常可以被此异常的基类catch

***

### 使用finally

1. try、catch、finally语句中，在如果try语句有return语句，则返回的之后当前try中变量此时对应的值，此后对变量做任何的修改，都不影响try中return的返回值
2. 如果finally块中有return 语句，则返回try或catch中的返回语句忽略。
3. 如果finally块中抛出异常，则整个try、catch、finally块中抛出异常

**所以使用try、catch、finally语句块中需要注意的是**

1. 尽量在try或者catch中使用return语句。通过finally块中达到对try或者catch返回值修改是不可行的。
2.  finally块中避免使用return语句，因为finally块中如果使用return语句，会显示的消化掉try、catch块中的异常信息，屏蔽了错误的发生
3. finally块中避免再次抛出异常，否则整个包含try语句块的方法回抛出异常，并且会消化掉try、catch块中的异常



***

## Chapter13-Strings

String对象是不可变的（只读），内部使用 char 数组存储数据，该数组被声明为 final 。

| 方法                    | 功能                             |
| ----------------------- | -------------------------------- |
| substring               | 左闭右开，只有一个数值则右边到底 |
| int indexOf(String str) | 可用于判断子字符串是否存在       |

### StringBuilder（线程不安全）

Java 5提出，由于String的连接（+）其实是调用了StringBuilder.append，所以，建议在for循环内使用StringBuilder来进行字符串的拼接。

### StringBuffer（线程安全）

和StringBuilder类似

如果程序是在单线程下运行，或者是不必考虑到线程同步问题，我们应该优先使用StringBuilder类；如果要保证线程安全，自然是StringBuffer。

>优先StringBuilder，为了线程安全用StringBuffer



***

### 正则表达式

String正则表达式相关的method

| method                         | 解释                            |
| ------------------------------ | ------------------------------- |
| matches(String regex)          |                                 |
| split(String regex)            |                                 |
| split(String regex, int limit) | 限制分割次数，最终数组数为limit |
| replaceFirst(),replaceAll()    |                                 |

#### 贪婪匹配和非贪婪匹配

如：String str="abcaxc";  Patter p="ab*c"; 

**贪婪匹配**：正则表达式一般**趋向于最大长度匹配**，也就是所谓的贪婪匹配。如上面使用模式p匹配字符串str，结果就是匹配到：abcaxc(ab*c)。 

**非贪婪匹配**：就是匹配到结果就好，就少的匹配字符。如上面使用模式p匹配字符串str，结果就是匹配到：**abc**(ab*c)。 

***

#### Pattern和Matcher

```
Pattern p = Pattern.compile(regex);
Matcher m = p.matcher(String args);
```



| method      | 解释                       |
| ----------- | -------------------------- |
| matches()   | 是否匹配                   |
| lookingAt() | 该字符串的起始部分是否匹配 |
| find()      |                            |

### Compare to

返回参与比较的前后两个字符串的asc码的差值，

如果两个字符串首字母不同，则该方法返回首字母的asc码的差值；

如果参与比较的两个字符串如果首字符相同，则比较下一个字符，直到有不同的为止，返回该不同的字符的asc码差值，如果两个字符串不一样长，可以参与比较的字符又完全一样，则返回两个字符串的长度差值





***

## Chapter14-Type Information

RTTI：runtime type information

[Class对象]()

>延伸点：类的加载



所有的类都是在第一次使用时，动态加载到JVM中的。

构造器也是static

Class.forName(目标类全限定名的String)，返回class引用，如果没有加载就加载它

Object.getClass。返回该对象的实际类型的Class引用

Class.getInterfaces  

Class.getSuperclass

Class.newInstance：



***

## Chapter15-Generics

通常情况下，一个编译器处理泛型有两种方式：`Code specialization`和`Code sharing`。C++和C#是使用`Code specialization`的处理机制，而Java使用的是`Code sharing`的机制。

Code sharing方式为每个泛型类型创建唯一的字节码表示，并且将该泛型类型的实例都映射到这个唯一的字节码表示上。将多种泛型类形实例映射到唯一的字节码表示是通过类型擦除（`type erasue`）实现的。

也就是说，**对于Java虚拟机来说，他根本不认识`Map<String, String> map`这样的语法。需要在编译阶段通过类型擦除的方式进行解语法糖。**

类型擦除的主要过程如下： 

1. 将所有的泛型参数用其最左边界（最顶级的父类型）类型替换。 
2. 移除所有的类型参数。

虚拟机中没有泛型，只有普通类和普通方法，所有泛型类的类型参数在编译时都会被擦除，泛型类并没有自己独有的Class类对象。比如并不存在`List<String>.class`或是`List<Integer>.class`，而只有`List.class`。



### 泛型接口：

生成器（generator）专门负责创造对象的类；

一般一个生成器只定义一个方法：用以产生新的对象；

基本类型无法作为类型参数

### 泛型方法：将泛型参数列表置于返回值之前

泛型方法所在的类可以不是泛型类，如果static方法需要使用泛型能力，就必须使其成为泛型方法

使用泛型类，必须在创建对象的时候指定类型参数的值，泛型方法不用

类型推断避免重复的泛型参数列表，但是只对赋值有效：

e.g：`Map<Person,List<? extends Pet>> petPeople = New.Map()`；



### 反射

反射可以用于判断任意对象所属的类，获得Class对象，构造任意一个对象以及调用一个对象

详见：[java中的反射机制](https://www.jianshu.com/p/cb739ae78881)

```java
//获取类型
Class c1 = Class.forName("Employee"); 
Class c2 = Employee.class;  
Employee e = new Employee(); 
Class c3 = e.getClass();
//创建对象
Objecto = c1.newInstance(); //调用了Employee的无参数构造方法.  
//获取属性
Field[] fs = c.getDeclaredFields();  
getDeclaredMethods()//获取所有的方法
getReturnType() //获得方法的放回类型
getParameterTypes() //获得方法的传入参数类型
getDeclaredMethod("方法名",参数类型.class,……)//获得特定的方法
getDeclaredConstructors()//获取所有的构造方法
getDeclaredConstructor(参数类型.class,……) //获取特定的构造方法
getSuperclass()//获取某类的父类
getInterfaces()//获取某类实现的接口
```

使用案例如Hibernate，BeanUtils。

------

### 动态代理

> 在程序运行时，**运用反射机制**动态创建而成。 
>
> 动态代理类的字节码在程序**运行时**由Java反射机制动态生成，无需程序员手工编写它的源代码。
>
> java.lang.reflect 包中的Proxy类和InvocationHandler 接口提供了生成动态代理类的能力。 



代理类往往会在代理对象业务逻辑前后增加一些功能性的行为，如使用事务或者打印日志。

#### jdk动态代理

**必须实现InvocationHandler接口**，并在该类中定义代理行为。





#### cglib动态代理

依赖于字节码技术

cglib是针对类来实现代理的，他的原理是对指定的`目标类`生成一个`子类`，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对`final修饰`的类进行代理。



***

## Chapter16-Arrays

数组命名时名称与[]可以随意排列

**粗糙数组：**构成矩阵的每个向量都可以具有任意的长度

不能实例化具有参数化类型的数组

创建测试数据：

Arrays.asList()返回的是一个List (List是一个接口，返回List实际是返回List接口的一个实现)，这个List在底层是有数组实现的，所以size是fixed的

Arrays.fill（Arrays，parameters）：用同一个parameter填充整个数组（引用）

Arrays实用功能：

System.arraycopy（源数组，偏移量，目标组，复制偏移量，复制个数）



***

## Chapter17-Containers in Depth





***

## Chapter18-I/O

>推荐资料：[java回忆录—输入输出流详细讲解（入门经典）](http://blog.csdn.net/qq_22063697/article/details/52137369)

![11](http://img.blog.csdn.net/20160806165130911)

### File类（处理文件目录问题）

file既可以代表一个文件名，也可以代表一个文件夹的名字

>延伸点：[ java.io.File类中mkdir()与mkdirs()区别](http://blog.csdn.net/a_woxinfeiyang_a/article/details/51419980)
>
>[File类(操作文件)](https://www.jianshu.com/p/389b912faa1a)

bio，nio区别要熟知，了解nio中的ByteBuffer，Selector，Channel可以帮助面试者度过不少难关。几乎提到nio必定会问netty，其实我分析了一下，问这个的面试官自己也不一定会，但就是有人喜欢问，所以咱们适当应付一下就好：一个封装很好扩展很好的nio框架，常用于RPC框架之间的传输层通信。



***

### 流

后缀是stream的都是字节流，其他的都是字符流

声明为static和transient类型的成员数据不能被串行化。因为static代表类的状态， transient代表对象的临时数据。

  inputstream的close方法用来关闭流 

  skip()用来跳过一些字节 

  mark（）用来标记流 

  reset（）复位流 

#### I/O的典型使用方法

1、缓冲输入文件

缓冲区的作用的主要目的是：避免每次和硬盘打交道，提高数据访问的效率。

```java
public class BufferedInputFile {
  public static String read(String filename) throws IOException {
    BufferedReader in = new BufferedReader(new FileReader(filename));
    String s;
    StringBuilder sb = new StringBuilder();
    while((s = in.readLine())!= null)
      sb.append(s + "\n");
    in.close();
    return sb.toString();
  }
  public static void main(String[] args)throws IOException {
    System.out.print(read("BufferedInputFile.java"));
  }
} 
```

2、从内存输入

```java
public class MemoryInput {
  public static void main(String[] args)throws IOException {
    StringReader in = new StringReader(BufferedInputFile.read("MemoryInput.java"));
    int c;
    //read()是以int形式返回下一字节，必须转为char才能正确打印
    while((c = in.read()) != -1)
      System.out.print((char)c);
  }
} 
```

3、格式化的内存输入

```java
public class FormattedMemoryInput {
  public static void main(String[] args)
  throws IOException {
    try {
      DataInputStream in = new DataInputStream(new ByteArrayInputStream(
         BufferedInputFile.read("FormattedMemoryInput.java").getBytes()));
      while(true)
        System.out.print((char)in.readByte());
    } catch(EOFException e) {
      System.err.println("End of stream");
    }
  }
}
```

4、基本的文件输出

```java
public class BasicFileOutput {
  static String file = "BasicFileOutput.out";
  public static void main(String[] args)throws IOException {
    BufferedReader in = new BufferedReader(
      new StringReader(BufferedInputFile.read("BasicFileOutput.java")));
    PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter(file)));
    int lineCount = 1;
    String s;
    while((s = in.readLine()) != null )
      out.println(lineCount++ + ": " + s);
    out.close();
    // Show the stored file:
    System.out.println(BufferedInputFile.read(file));
  }
}
```





为了提高读写性能，可以采用什么流

Java中有几种类型的流

JDK 为每种类型的流提供了一些抽象类以供继承，分别是哪些类

对文本文件操作用什么I/O流

对各种基本数据类型和String类型的读写，采用什么流

能指定字符编码的 I/O 流类型是什么



>延伸点：装饰者模式

>BIO：同步阻塞式IO，简单理解：一个连接一个线程；Apache，Tomcat。主要是并发量要求不高的场景
>NIO：同步非阻塞IO，简单理解：一个请求一个线程；Nginx，Netty。主要是高并发量要求的场景
>AIO：异步非阻塞IO，简单理解：一个有效请求一个线程；还不是特别成熟，底层也基本是多线程模拟，所以应用场景不多，Netty曾经用了，但又放弃了。





***

### 序列化

什么是序列化？如何实现 Java 序列化及注意事项

Serializable 与 Externalizable 的区别



### Socket

- - socket 选项 TCP NO DELAY 是指什么
  - Socket 工作在 TCP/IP 协议栈是哪一层
  - TCP、UDP 区别及 Java 实现方式
- 说几点 IO 的最佳实践
- 直接缓冲区与非直接缓冲器有什么区别？
- 怎么读写 ByteBuffer？ByteBuffer 中的字节序是什么
- 当用System.in.read(buffer)从键盘输入一行n个字符后，存储在缓冲区buffer中的字节数是多少
- 如何使用扫描器类（Scanner Class）令牌化

***

## Chapter19-Enumerated Types

### 方法

| 方法                  | 功能                                                         | 返回值 |
| --------------------- | ------------------------------------------------------------ | ------ |
| values()              | 返回enum实例的数组                                           | T[]    |
| compareTo(E o)        | 顺序比较，如果该枚举对象位于指定枚举对象之后，则返回正整数；反之返回负整数；否则返回零； | int    |
| name()                | 返回此枚举实例的名称                                         |        |
| getDeclaringClass（） | 返回实例所属于的enum类                                       | T      |
| ordinal()             | 返回实例声明时候的顺序                                       | int    |

### 添加方法

>定义实例方法必须在任何方法或者属性前
>
>定义实例方法后面加上“;”，然后开始写方法



1. 枚举不允许继承类。Jvm在生成枚举时已经继承了Enum类，由于Java语言是单继承，不支持再继承额外的类（唯一的继承名额被Jvm用了）。
2. 枚举允许实现接口。因为枚举本身就是一个类，类是可以实现多个接口的。
3. 枚举可以用等号比较。Jvm会为每个枚举实例对应生成一个类对象，这个类对象是用public static final修饰的，在static代码块中初始化，是一个单例。
4. 不可以继承枚举。因为Jvm在生成枚举类时，将它声明为final。
5. 枚举本身就是一种对单例设计模式友好的形式，它是实现单例模式的一种很好的方式。
6. 枚举类型的compareTo()方法比较的是枚举类对象的ordinal的值。
7. 枚举类型的equals()方法比较的是枚举类对象的内存地址，作用与等号等价。









***

## Chapter21-并发



#### 线程安全与锁

> 当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替进行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么称这个类是线程安全的

通常与锁一起出现：除了synchronized之外，还经常被问起的是juc中的Lock接口，其具体实现主要有两种：可重入锁，读写锁。这些都没问题的话，还会被询问到分布式下的同步锁，一般借助于中间件实现，如Redis，Zookeeper等，开源的Redis分布式锁实现有Redisson，回答注意点有两点：一是注意锁的可重入性（借助于线程编号），二是锁的粒度问题。除此之外就是一些juc的常用工具类如：CountdownLatch，CyclicBarrir，信号量

### 线程

#### 创建线程

1. **继承Thread**，重写run方法，创建实例并调用start方法
2. **实现runnable接口**，编写run方法，传入Thread。e.g: Thread t = new Thread(new Runnable())
3. **实现Callable接口**，重写call方法，线程结束后可以有返回值，但是该方式是依赖于线程池的，必须调用exec.submit(new Callable());

##### 使用Executor

```java
ExecutorService exec = Executors.newCacheThreadPool();
for(int i ; i <5 ; i++)
//Liftoff实现了runnable接口
	exec.execute(new Liftoff());
//防止新任务提交给这个executor
exec.shutdown();
```

newFixedThreadPool(int number);一次性预先执行高代价的线程分配

newSingleThreadExecutor; 类似于newFixedThreadPool(1)，会序列化所有提交给它的任务。

#### Thread的方法

| method          | 解释                                                         |
| --------------- | ------------------------------------------------------------ |
| run()           | 执行线程体中具体的内容，如果直接调用run方法，其实还是单线程  |
| start()         | 启动线程对象，使其进入**就绪状态**                           |
| sleep()         | 占着CPU不工作。可以抛出interruptException，但是异常抛出的时候回清理这个标志，导致在catch里面调用isInterrupt()，总为false。TimeUnit.SECONDS.sleep(1)：可读性更高 |
| suspend()       | 使线程挂起，要通过resume()方法使其重新启动                   |
| join()          | 等待该线程终止；在父线程中，子线程调用join，父线程被挂起，等待子线程结束才回复。 |
| yield()         | 当前线程放弃CPU，所有线程（包括当前线程）共同竞争CPU。       |
| setDeamon(true) | 设置为后台线程，必须在线程start之前调用。                    |
| getName()       | 有一构造函数参数为String name；获取线程的name。              |
| isAlive()       | 判断是否被挂起                                               |
| holdsLock()     | 返回true如果当且仅当当前线程拥有某个具体对象的锁。           |

##### wait 和 sleep的区别

**wait**：等待使用CPU
**sleep**：占着CPU不工作



#### 线程转换

![fff](https://upload-images.jianshu.io/upload_images/2615789-1345e368181ad779.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

#### 结束线程

1. run方法执行完成，线程正常结束 
2. 线程抛出一个未捕获的Exception或者Error 
3. 直接调用该线程的Stop方法结束线程（不建议使用，容易导致死锁） 
4. 使用interrupt方法中断线程。可以使一个被阻塞的线程抛出一个中断异常，从而使线程提前结束阻塞状态，退出堵塞代码。


> 延伸点：注意`interrupted()`会清楚中断状态
>
> 参考链接：[如何正确地停止一个线程？](http://www.cnblogs.com/greta/p/5624839.html)





***

#### 优先级

调度器倾向于让优先级最高的线程先执行，通常在run的开头部分设定

getPriority():读取现有线程的优先级

setPriority():修改线程优先级。

***

#### LocalThread

每个线程都有一个ThreadLocal就是每个线程都拥有了自己独立的一个变量，竞争条件被彻底消除了。

它是为创建代价高昂的对象获取线程安全的好方法，比如你可以用ThreadLocal让SimpleDateFormat变成线程安全的，因为那个类创建代价高昂且每次调用都需要创建不同的实例所以不值得在局部范围使用它，如果为每个线程提供一个自己独有的变量拷贝，将大大提高效率。

首先，通过复用减少了代价高昂的对象的创建个数。其次，你在没有使用高代价的同步或者不变性的情况下获得了线程安全。线程局部变量的另一个不错的例子是ThreadLocalRandom类，它在多线程环境中减少了创建代价高昂的Random对象的个数。

>参考资料：[深入分析 ThreadLocal 内存泄漏问题](http://blog.xiaohansong.com/2016/08/06/ThreadLocal-memory-leak/) 



***

### 锁

#### 粒度



***

#### 乐观锁和悲观锁

悲观锁是独占锁，阻塞锁，写入频繁使用悲观锁

乐观锁是非独占锁，非阻塞锁。读取频繁使用乐观锁。

>乐观锁（ `Optimistic Locking`）其实是一种思想。相对悲观锁而言，乐观锁假设认为数据一般情况下不会造成冲突，所以在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则让返回用户错误的信息，让用户决定如何去做。
>
>有一种乐观锁的实现方式就是CAS ，这种算法在JDK 1.5中引入的`java.util.concurrent`(J.U.C)中有广泛应用。但是值得注意的是这种算法会存在ABA问题（部分通过版本号来解决）



这里就是悲观锁乐观锁的应用处理并发问题

>参考资料：[乐观锁的一种实现方式——CAS](http://www.importnew.com/20472.html)



***

#### 死锁

死锁是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。这是一个严重的问题，因为死锁会让你的程序挂起无法完成任务。

死锁的发生必须满足以下四个条件：

1. 互斥条件：一个资源每次只能被一个进程使用。
2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件：进程已获得的资源，在末使用完之前，不能强行剥夺。
4. 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。

避免死锁最简单的方法就是**阻止循环等待条件**，将系统中所有的资源设置标志位、排序，规定所有的进程申请资源必须以一定的顺序（升序或降序）做操作来避免死锁。

>Q：如何检查死锁?
>
>A：通过jConsole检查死锁
>
>参考资料：[java 查看线程死锁](http://www.cnblogs.com/ilahsa/archive/2013/06/03/3115410.html)



***

#### 可重入锁

可重入锁，也叫做递归锁，指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响。

在JAVA环境下 ReentrantLock 和synchronized 都是 可重入锁





***

#### synchronized

加到 static 方法上是给 Class 上锁，即整个类上锁。



***

#### Lock

Lock对象必须被**显式**地创建、锁定和释放。

紧接着lock的调用，后面是try-finally语句，同时finally里面要有unclock来释放锁

```java
public class MutexEvenGenerator extends IntGenerator {
  private int currentEvenValue = 0;
  private Lock lock = new ReentrantLock();
  public int next() {
    lock.lock();
    try {
      ++currentEvenValue;
      Thread.yield(); // Cause failure faster
      ++currentEvenValue;
      return currentEvenValue;
} finally {
      lock.unlock();
    }
}
public static void main(String[] args) {
    EvenChecker.test(new MutexEvenGenerator());
  }
} ///:~
```







***

#### volatile

1. volatile**保证了可视性和有序性**，没有原子性。
2. 由于有些时候对 volatile的操作，不会被保存，说明不会造成阻塞。不可用与多线程环境下的计数器。 
3. **禁止指令重排序优化** 

​       禁止指令重排序优化。编译器只保证程序执行结果与源代码相同，却不保证实际指令的顺序与源代码相同。这在单线程看起来没什么问题，然而一旦引入多线程，这种乱序就可能导致严重问题。volatile关键字就可以从语义上解决这个问题。 jdk1.5以后生效。

参考：[Java面试官最爱的volatile关键字]()

***

#### JMM(java内存模型)

>关注点：主存和工作内存， happens-before原则（主要是前三条）

JMM规定所有变量都是存在主存中的，每个线程又包含自己的工作内存。所以线程的操作都是以工作内存为主，它们只能访问自己的工作内存，且工作前后都要把值在同步回主内存。

![java内存模型](https://pic1.zhimg.com/80/v2-62a280b6092df4990cb329e05a82afe0_hd.jpg)

##### 原子性

基本类型变量，除了long和double，都能保证原子性的操作。

long和double由于是分离的32位来操作，不能保证原子性。

##### 可见性

当一个变量被volatile修饰时，那么**对它的修改会立刻刷新到主存**，当其它线程需要读取该变量时，会去内存中读取新值。而普通变量则不能保证这一点。

其实通过synchronized和Lock也能够保证可见性，线程在释放锁之前，会把共享变量值都刷回主存，但是synchronized和Lock的开销都更大。

##### 有序性

JMM是允许编译器和处理器对指令重排序的，但是规定了as-if-serial语义，即不管怎么重排序，**程序的执行结果不能改变。**

***

##### happens-before原则

1. **程序顺序规则**： 一个线程中的每个操作，happens-before于该线程中的任意后续操作
2. **监视器锁规则**：对一个线程的解锁，happens-before于随后对这个线程的加锁
3. **volatile变量规则**： 对一个volatile域的写，happens-before于后续对这个volatile域的读
4. **传递性**：如果A happens-before B ,且 B happens-before C, 那么 A happens-before C
5. **start()规则**： 如果线程A执行操作`ThreadB_start()`(启动线程B) ,  那么A线程的`ThreadB_start()`happens-before 于B中的任意操作
6. **join()原则**： 如果A执行`ThreadB.join()`并且成功返回，那么线程B中的任意操作happens-before于线程A从`ThreadB.join()`操作成功返回。
7. **interrupt()原则**： 对线程`interrupt()`方法的调用先行发生于被中断线程代码检测到中断事件的发生，可以通过`Thread.interrupted()`方法检测是否有中断发生
8. **finalize()原则**：一个对象的初始化完成先行发生于它的`finalize()`方法的开始

***

#### 优化

- 减少锁持有时间（尽量缩小锁的范围）
- 减小锁粒度
- 锁分离
- 锁粗化
- 锁消除

***

### 同步器（AQS）

同步器是一些使线程能够等待另一个线程的对象，允许它们协调动作。最常用的同步器是CountDownLatch和Semaphore，不常用的是Barrier 和Exchanger

>延伸点：[java多线程面试汇总](http://www.dwring.com/interview/2017/08/04/thread.html)
>
>[并发工具类（三）控制并发线程数的Semaphore](http://ifeve.com/concurrency-semaphore/)

***

## 其他

### Java程序的种类

1. 内嵌于Web文件中，由浏览器来观看的_Applet 
2. 可独立运行的 Application 
3. 服务器端的 Servlets

***

### 关键字和保留字

![ggg](http://uploadfiles.nowcoder.net/images/20160129/828061_1454054455847_8A5F006FFD4CE801C8E888EC3BC68361)

要注意true,false,null,　friendly，sizeof不是java的关键字,但是你不能把它们作为java标识符用。 

***

#### null

1. null是代表不确定的对象，null是一个关键字。因此可以**将null赋给引用类型变量，但不可以将null赋给基本类型变量。**


2. null本身不是对象，也不是Objcet的实例
3. Java默认给引用变量赋值
4. 容器类型与null

List：允许重复元素，可以加入任意多个null。

Set：不允许重复元素，最多可以加入一个null。

Map：Map的key最多可以加入一个null，value字段没有限制。

数组：基本类型数组，定义后，如果不给定初始值，则java运行时会自动给定值。引用类型数组，不给定初始值，则所有的元素值为null。

5. null的其他作用

1、判断一个引用类型数据是否null。 用==来判断。

2、释放内存，让一个非null的引用类型变量指向null。这样这个对象就不再被任何对象应用了。等待JVM垃圾回收机制去回收。

### 包导入

java.lang包定义了一些基本的类型，包括Integer,String之类的，是java程序必备的包，有解释器自动引入，无需手动导入 

| 工具           | 功能                                 |
| ------------ | ---------------------------------- |
| javac.exe    | 把.java文件编译为.class文件（每个类一个.class文件） |
| java.exe     | 执行编译好的.class文件                     |
| javadoc.exe  | 生成Java说明文档，api文档                   |
| jdb.exe      | Java调试器                            |
| javaprof.exe | 剖析工具                               |

### java代码执行过程

- 编译器将Java源代码编译成字节码class文件 
- 类加载到JVM里面后，执行引擎把字节码转为可执行代码 
- 执行的过程，再把可执行代码转为机器码，由底层的操作系统完成执行。 

***

### 常见码表

Java语言使用的是Unicode字符集。

ascii:美国标准信息交换码。使用的是1个字节的7位来表示该表中的字符。

ISO8859-1:拉丁码表。使用1个字节来表示。

GB2312:简体中文码表。

GBK：简体中文码表，比GB2312融入更多的中文文件和符号。

unicode:国际标准码表。都用两个字节表示一个字符。

UTF-8：对unicode进行优化，每一个字节都加入了标识头。

***

### 面向对象的五大原则

| 名称                                       | 功能                                       |
| ---------------------------------------- | ---------------------------------------- |
| 单一职责原则（Single-Resposibility Principle）   | 一个类，最好只做一件事，只有一个引起它的变化。单一职责原则可以看做是低耦合、高内聚在面向对象原则上的引申，将职责定义为引起变化的原因，以提高内聚性来减少引起变化的原因。 |
| 开放封闭原则（Open-Closed principle）            | 软件实体应该是可扩展的，而不可修改的。也就是，对扩展开放，对修改封闭的。     |
| Liskov替换原则（Liskov-Substituion Principle） | 子类必须能够替换其基类。这一思想体现为对继承机制的约束规范，只有子类能够替换基类时，才能保证系统在运行期内识别子类，这是保证继承复用的基础。 |
| 依赖倒置原则（Dependecy-Inversion Principle）    | 依赖于抽象。具体而言就是高层模块不依赖于底层模块，二者都同依赖于抽象；抽象不依赖于具体，具体依赖于抽象。 |
| 接口隔离原则（Interface-Segregation Principle）  | 使用多个小的专门的接口，而不要使用一个大的总接口                 |



***

### Session

session.setAttribute()和session.getAttribute()配对使用，作用域是整个会话期间，在所有的页面都使用这些数据的时候使用。

request.getAttribute()表示从request范围取得设置的属性，必须要先setAttribute设置属性，才能通过getAttribute来取得，设置与取得的为Object对象类型。

***

### J2EE相关名词

1. **web容器**：给处于其中的应用程序组件（JSP，SERVLET）提供一个环境，使JSP,SERVLET直接和容器中的环境变量接接口互，不必关注其它系统问题。主要有WEB服务器来实现。例如：TOMCAT,WEBLOGIC,WEBSPHERE等。该容器提供的接口严格遵守J2EE规范中的WEB APPLICATION 标准。我们把遵守以上标准的WEB服务器就叫做J2EE中的WEB容器。 
2. **Web  container**：实现J2EE体系结构中Web组件协议的容器。这个协议规定了一个Web组件运行时的环境，包括安全，一致性，生命周期管理，事务，配置和其它的服务。一个提供和JSP和J2EE平台APIs界面相同服务的容器。一个Web container 由Web服务器或者J2EE服务器提供。 
3. **EJB容器**：Enterprise java bean 容器。更具有行业领域特色。他提供给运行在其中的组件EJB各种管理功能。只要满足J2EE规范的EJB放入该容器，马上就会被容器进行高效率的管理。并且可以通过现成的接口来获得系统级别的服务。例如邮件服务、事务管理。一个实现了J2EE体系结构中EJB组件规范的容器。  这个规范指定了一个Enterprise bean的运行时环境，包括安全，一致性，生命周期，事务，  配置，和其他的服务。
4. **JNDI**：（Java Naming & Directory Interface）JAVA命名目录服务。主要提供的功能是：提供一个目录系统，让其它各地的应用程序在其上面留下自己的索引，从而满足快速查找和定位分布式应用程序的功能。 
5. **JMS**：（Java Message Service）JAVA消息服务。主要实现各个应用程序之间的通讯。包括点对点和广播。 
6. **JTA**：（Java Transaction API）JAVA事务服务。提供各种分布式事务服务。应用程序只需调用其提供的接口即可。 
7. **JAF**：（Java Action FrameWork）JAVA安全认证框架。提供一些安全控制方面的框架。让开发者通过各种部署和自定义实现自己的个性安全控制策略。 
8. **RMI/IIOP**:（Remote Method Invocation /internet对象请求中介协议）他们主要用于通过远程调用服务。例如，远程有一台计算机上运行一个程序，它提供股票分析服务，我们可以在本地计算机上实现对其直接调用。当然这是要通过一定的规范才能在异构的系统之间进行通信。RMI是JAVA特有的。RMI-IIOP出现以前，只有RMI和CORBA两种选择来进行分布式程序设计。RMI-IIOP综合了RMI和CORBA的优点，克服了他们的缺点，使得程序员能更方便的编写分布式程序设计，实现分布式计算。首先，RMI-IIOP综合了RMI的简单性和CORBA的多语言性（兼容性），其次RMI-IIOP克服了RMI只能用于Java的缺点和CORBA的复杂性（可以不用掌握IDL）。

>延伸点：[JDK, JRE 和JVM的区别](http://www.importnew.com/7021.html)



***

### Properties类

继承自Hashtable

我自己实际使用应该只有在xml里

```xml
   <context:property-placeholder location="classpath*:conf/conf_a.properties"/>  
   <bean class="com.xxx.aaa.Bean1"  
          p:driverClassName="${modulea.jdbc.driverClassName}"  
          p:url="${modulea.jdbc.url}"  
          p:username="${modulea.jdbc.username}"  
          p:password="${modulea.jdbc.password}"/> 
```



***

### 可变参数

兼容数组类参数的。

必须作为参数列表的最后一项

能匹配定长的方法，那么优先匹配该方法。含有不定参数的那个重载方法是最后被选中的。

>参考： [ Java方法的可变参数个数](http://blog.csdn.net/testcs_dn/article/details/38920323)                            



