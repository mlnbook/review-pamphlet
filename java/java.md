# Java 基础

## 1. 分析线程池的实现原理和线程的调度过程

### 1.1 什么时候使用线程池？

- 单个任务处理时间比较短
- 需要处理的任务数量很大

### 1.2 使用线程池的好处

引用自 [http://ifeve.com/java-threadpool/](http://ifeve.com/java-threadpool/) 的说明：

- **降低资源消耗** 。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度** 。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- **提高线程的可管理性** 。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

### 1.3 ThreadPoolExecutor

![image](http://static.lovedata.net/jpg/2018/5/28/27abf026c12c631217100176ef0d7528.jpg)

![image](http://static.lovedata.net/jpg/2018/5/28/ec7340130d0fb9cf76247972c89e73eb.jpg)

参考

1. [探秘线程池 ThreadPoolExecutor 的任务调度过程 | 指间生活](http://www.zhenchao.org/2017/08/31/java-thread-pool-executor/)
2. [深入理解 Java 线程池：ThreadPoolExecutor - 后端 - 掘金](https://juejin.im/entry/58fada5d570c350058d3aaad)
3. [聊聊并发（三）Java线程池的分析和使用 | 并发编程网 – ifeve.com](http://ifeve.com/java-threadpool/)

## 2. 动态代理的几种方式

## 3. Spring AOP与IOC的实现

## 4. Dubbo的底层实现原理和机制，

## 5. 接口的幂等性的概念

1. **错误定义** ：幂等性是指重复使用 **同样的参数** 调用同一方法时总能获得 **同样的结果** 。比如对同一资源的GET请求访问结果都是一样的。
2. GET方法是向服务器查询，不会对系统产生副作用，具有幂等性（不代表每次请求都是相同的结果)
3. **HTTP的幂等性** 指的是一次和多次请求某一个资源应该具有相同的副作用。如通过PUT接口将数据的Status置为1，无论是第一次执行还是多次执行，获取到的结果应该是相同的，即执行完成之后Status =1。
4. 随着分布式系统及微服务的普及，因为网络原因而导致调用系统未能获取到确切的结果从而导致重试，这就需要被调用系统具有幂等性。
5. 定义： **幂等性是系统的接口对外一种承诺(而不是实现),** 承诺只要调用接口成功, 外部多次调用对系统的影响是一致的. 声明为幂等的接口会认为外部调用失败是常态, **并且失败之后必然会有重试.**
6. 解决办法
    1. 美团GTIS，通过Tair生成全局的业务id，在执行前和执行后进行判断
        1. ![image](http://static.lovedata.net/jpg/2018/5/28/4fbdeeef48fed51ed59db8109cbf494e.jpg)
7. 参考
    1. [分布式系统接口幂等性 - BruceFeng](https://blog.brucefeng.info/post/api-idempotent)
    2. [接口的幂等性 - 简书](https://www.jianshu.com/p/b09a2e9bcd29)
    3. [分布式系统互斥性与幂等性问题的分析与解决 -](https://tech.meituan.com/distributed-system-mutually-exclusive-idempotence-cerberus-gtis.html)

## 6. Synchronized和Lock的区别

[死磕Java并发：深入分析synchronized的实现原理](https://mp.weixin.qq.com/s/wHz0uL_LEe4OgLsSFGEZEg)

## 7. HashMap的原理和并发问题

### 7.1 原理

[HashMap实现原理分析 - CSDN博客](http://blog.csdn.net/vking_wang/article/details/14166593)

### 7.2 并发问题

## 8. 了解LinkedHashMap的应用吗

## 9. 线程池实现原理，Lock机制的实现

## 10. ConcurrentHashMap深入分析

### 10.1 concurrenthashmap怎么实习同步？各个版本的实现方案？

## 11. callable runnable 区别；

## 12. 进程线程区别；

## 13. hashMap和treeMap的区别，以及实现；

## 14. ByteBuffer的原理和使用

## 15. java的重载、重写、覆盖

![image](http://static.lovedata.net/jpg/2018/6/22/b1ac6f6922a609e910e0ed608f45a3dd.jpg)
![image](http://static.lovedata.net/jpg/2018/6/22/8484c892ff673e69d15a29646a020916.jpg)

方法的重写(Overriding)和重载(Overloading)是java多态性的不同表现，重写是父类与子类之间多态性的一种表现，重载可以理解成多态的具体表现形式。

### 15.1 重载

[Java 实例 – 方法重载 | 菜鸟教程](http://www.runoob.com/java/method-overloading.html)

重载(overloading) 是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同。

每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。
最常用的地方就是构造器的重载。

如果有两个方法的方法名相同，但参数不一致，哪么可以说一个方法是另一个方法的重载。 具体说明如下：

- 方法名相同
- 方法的参数类型，参数个不一样
- 方法的返回类型可以不相同
- 方法的修饰符可以不相同
- main 方法也可以被重载

重载规则:

- 被重载的方法必须改变参数列表(参数个数或类型不一样)；
- 被重载的方法可以改变返回类型；
- 被重载的方法可以改变访问修饰符；
- 被重载的方法可以声明新的或更广的检查异常；
- 方法能够在同一个类中或者在一个子类中被重载。
- 无法以返回值类型作为重载函数的区分标准。

### 15.2 覆盖（重写）

重写是子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变。即外壳不变，核心重写！

重写方法不能抛出新的检查异常或者比被重写方法申明更加宽泛的异常   父类的一个方法申明了一个检查异常 IOException，但是在重写这个方法的时候不能抛出 Exception 异常，因为 Exception 是 IOException 的父类，只能抛出 IOException 的子类异常。

方法的重写规则

- 参数列表必须完全与被重写方法的 **相同；**
- 返回类型必须完全与被重写方法的返回类型 **相同；**
- 访问权限不能比父类中被重写的方法的访问权限更低。例如：如果父类的一个方法被声明为public，那么在子类中重写该方法就不能声明为protected。
- 父类的成员方法只能被它的子类重写。
- **声明为final的方法不能被重写。**
- **声明为static的方法不能被重写，但是能够被再次声明。**
- 子类和父类在同一个包中，那么子类可以重写父类所有方法，除了声明为private和final的方法。
- 子类和父类不在同一个包中，那么子类只能够重写父类的声明为public和protected的非final方法。
- 重写的方法能够抛出任何非强制异常，无论被重写的方法是否抛出异常。但是，重写的方法不能抛出新的强制性异常，或者比被重写方法声明的更广泛的强制性异常，反之则可以。
- 构造方法不能被重写。
- 如果不能继承一个方法，则不能重写这个方法。

[Java 重写(Override)与重载(Overload) | 菜鸟教程](http://www.runoob.com/java/java-override-overload.html)
