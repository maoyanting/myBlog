---
title: JavaWeb面试-设计模式
date: 2018-01-26 15:12:19
categories: 
- JavaWeb面试
tags:
- 设计模式
- 面试
- Java
---

设计模式简单整理

设计模式是种思路，概念什么的只是为了应付面试。

<!--more-->

## 分类

1. 创建型模式：对象怎么来
2. 结构型模式：对象和谁有关
3. 行为型模式：对象与对象在干嘛
4. J2EE 模式：对象合起来要干嘛

***

## 六大原则

开闭原则：**对扩展开放，对修改关闭**。

里氏代换原则：**子父类互相替换**，实现抽象的规范；

依赖倒转原则：**针对接口编程**，依赖于抽象而不依赖于具体，实现开闭原则的基础；

接口隔离原则：降低耦合度，接口单独设计，互相隔离；

迪米特法则，又称不知道原则：功能模块尽量独立；

合成复用原则：**尽量使用聚合、组合**，而不是继承；

***

## 策略模式

### 目的

定义了算法族，分别分装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

>关注点：组合
>
>[《JAVA与模式》之策略模式](http://www.cnblogs.com/java-my-life/archive/2012/05/10/2491891.html)







***

## 单例模式

### 目的

**防止其他开发人员不小心new出多个实例**

**优点**：1. 节省内存；2. 方便管理

**缺点：**没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。

>关注点：线程安全，序列化，懒加载，反射攻击

***

### 应用实例

- **无状态的工具类**：比如日志工具类，不管是在哪里使用，我们需要的只是它帮我们记录日志信息，除此之外，并不需要在它的实例对象上存储任何状态，这时候我们就只需要一个实例对象即可。
- **全局信息类**：比如我们在一个类上记录网站的访问次数，我们不希望有的访问被记录在对象A上，有的却记录在对象B上，这时候我们就让这个类成为单例。
- 默认情况下Spring中定义的Bean是以单例的饿汉模式创建的，启动容器时，为所有spring配置文件中定义的bean都生成一个实例

***

### 不建议——懒汉模式

- **私有构造函数**
- 在类的内部创建实例，使用**Double-Check + synchronized同步锁**，解决**多线程并发访问**可能导致的在内部调用多次new的问题
- 使用**volatile关键字**，解决由于**指令重排**而可能出现的在内部调用多次new的问题
- 提供获取唯一实例的方法

```java
public class PerfectLazyManSingleton {
   private volatile static PerfectLazyManSingleton instance = null;
   private PerfectLazyManSingleton() {
   }
   public static PerfectLazyManSingleton getInstance() {
     //这里就是双检锁了
       if(instance == null) {
           synchronized (PerfectLazyManSingleton.class) {
               if(instance == null) {
                   instance = new PerfectLazyManSingleton();
               }
           }
       }
       return instance;
   }
}
```



>看了一圈评论，对于这个懒汉模式的评价都不高啊，好像实际使用并不好，但是面试的提及率挺高的。
>
>懒汉分为基本的线程安全和线程不安全，还有静态内部类
>
>线程安全优化递进：1、在getInstance()前面加一个synchronized；2、改为双检锁 ；3、加volatile

#### 双检锁

>延伸点：锁的粒度

#### volatile

> instance = new PerfectLazyManSingleton()；这句，这并非是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情。
>
> 1. 给 instance 分配内存
> 2. 调用 Singleton 的构造函数来初始化成员变量
> 3. 将instance对象指向分配的内存空间（执行完这步 instance 就为非 null 了）
>
> 但是在 JVM 的即时编译器中存在指令重排序的优化。也就是说上面的第二步和第三步的顺序是不能保证的，最终的执行顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，则在 3 执行完毕、2 未执行之前，被线程二抢占了，这时 instance 已经是非 null 了（但却没有初始化），所以线程二会直接返回 instance，然后使用，然后顺理成章地报错。
>
> 我们只需要将 instance 变量声明成 volatile 就可以了。
>
> ————来自[如何正确地写出单例模式](http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/)

volatile更多详见[JavaWeb面试-Java](https://maoyanting.github.io/2018/01/17/JavaWeb%E9%9D%A2%E8%AF%95-Java/)

#### 静态内部类

```java
public class Singleton {  
    private static class SingletonHolder {  
        private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
        return SingletonHolder.INSTANCE; 
    }  
}
```

当Singleton被加载时，其内部类并不会被初始化，故可以确保当 Singleton类被载入JVM时，不会初始化单例类。只有 `getInstance()` 方法调用时，才会初始化 instance。同时，由于实例的建立是时在类加载时完成，故天生对多线程友好，`getInstance()` 方法也无需使用同步关键字。

***

### 最简单——饿汉模式

**基于 classloder 机制避免了多线程的同步问题**

在类加载的过程中被初始化，而Java类加载的过程默认是线程安全的，除非自定义的类加载器覆写了loadClass函数。

>延伸点：双亲委托模型
>
>关注点：final static修饰

```java
public class SingletonHungryMan {
    public final static SingletonHungryMan INSTANCE = new SingletonHungryMan();
    private SingletonHungryMan() {
        // Exists only to defeat instantiation.
    }
    public void sayHello() {
        System.out.println("hello");
    }
}
```

***

### 最佳——枚举

线程安全问题：默认枚举实例的创建是线程安全的.(创建枚举类的单例在JVM层面也是能保证线程安全的),

序列化和反射问题都不影响，这个是为嘛呢？

**--------------------------------待补-----------------------------**

```java
public enum Singleton{
    INSTANCE;
}
```

>延伸点：枚举（jdk1.5以后）

标准的enum单例模式：

```java
// 定义单例模式中需要完成的代码逻辑
public interface MySingleton {
    void doSomething();
}

public enum Singleton implements MySingleton {
    INSTANCE {
        @Override
        public void doSomething() {
            System.out.println("complete singleton");
        }
    };
    public static MySingleton getInstance() {
        return Singleton.INSTANCE;
    }
}
链接：https://www.jianshu.com/p/4e8ca4e2af6c
```



***

### 序列化问题

除了枚举，其他的单例模式实现了Serializable接口后，反序列化时单例会被破坏。

实现Serializable接口需要重写`readResolve`，才能保证其反序列化依旧是单例

延伸阅读：[深度解析单例与序列化之间的爱恨情仇~](https://mp.weixin.qq.com/s/iXC47w4tMfpZzTNxS_JQOw)

***

### 懒加载

也就是延迟加载;

比如饿汉模式，会在类加载阶段完成了对对象的创建:

```java
public final static SingletonHungryMan INSTANCE = new SingletonHungryMan();
```

***

### 反射攻击

讲道理，本来单例模式的目的就是防止其他开发人员**不小心**new出多个实例，都用上反射了，这就不算不小心了。

这里先空着

**--------------------------------待补-----------------------------**

资料：[ 如何防止单例模式被JAVA反射攻击 ](http://blog.csdn.net/u013256816/article/details/50525335)

***

## 工厂模式

### 目的

为创建对象提供过渡接口，以便将创建对象的具体过程屏蔽隔离起来，达到提高灵活性的目的。 

### 应用实例

1. Spring Bean 的创建
2. ​

***

### 简单工厂模式（Simple Factory）







***

### 工厂方法模式（Factory Method）

一个抽象产品类，可以派生出多个具体产品类。   
一个抽象工厂类，可以派生出多个具体工厂类。   
每个具体工厂类只能创建一个具体产品类的实例。





***

### 抽象工厂模式（Abstract Factory） 

多个抽象产品类，每个抽象产品类可以派生出多个具体产品类。   
一个抽象工厂类，可以派生出多个具体工厂类。   
每个具体工厂类可以创建多个具体产品类的实例。   









***

## 代理模式

### 目的

给某一个委托类创建一个代理类，而**由这个代理类控制对委托类的引用**，而创建这个代理类就是可以在调用委托类是可以增加一些额外的操作。

他的特征是代理类与委托类**有同样的接口**，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。

代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。

### 应用实例

1. Spring Aop 中 Jdk 动态代理
2. ​





工厂模式，观察者模式，模板方法模式，策略模式，职责链模式等等，通常会结合Spring和UML类图提问。

***

## 观察者模式

这种设计模式也是常用的设计方法通常也叫发布 - 订阅模式，也就是事件监听机制，通常在某个事件发生的前后会触发一些操作。

### 应用实例

1. Tomcat中控制组件生命周期的 Lifecycle，对 Servlet 实例的创建、Session 的管理、Container 等














***

## 瞎扯

感觉对于面试来说写的太多了，总是一不小心就深入进去了……