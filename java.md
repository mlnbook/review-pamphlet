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

### 7.2 并发问题

## 8. 了解LinkedHashMap的应用吗

## 9. 线程池实现原理，Lock机制的实现

## 10. ConcurrentHashMap深入分析

### 10.1 concurrenthashmap怎么实习同步？各个版本的实现方案？

## 11. callable runnable 区别；

## 12. 进程线程区别；

## 13. hashMap和treeMap的区别，以及实现；
