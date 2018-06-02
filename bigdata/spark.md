# Spark

## 1. 基础知识

### 1.1.Spark性能优化主要有哪些手段？

### 1.2. 对于Spark你觉得他对于现有大数据的现状的优势和劣势在哪里？

### 1.3. Spark的Shuffle原理及调优？

### 1.4.spark如何保证宕机迅速恢复?

### 1.5. RDD持久化原理

### 1.6. spark排序实现流程，reduce端怎么实现的；

### 1.7.Spark的特点是什么

### 1.8.Spark的三种提交模式是什么

### 1.9.spark 实现高可用性：High Availability？

### 1.10. spark中怎么解决内存泄漏问题？

### 1.11. Spark-submit模式yarn-cluster和yarn-client的区别

1. yarn-client用于测试，因为他的Driver运行在本地客户端，会与yarn集群产生较大的网络通信，从而导致网卡流量激增；它的好处在于直接执行时，在本地可以查看到所有的log，方便调试；
2. yarn-cluster用于生产环境，因为Driver运行在NodeManager，相当于一个ApplicationMaster，没有网卡流量激增的问题；缺点在于调试不方便，本地用spark-submit提交后，看不到log，只能通过yarn application_id这种命令来查看，很麻烦

### 1.12.spark运行原理，从提交一个jar到最后返回结果，整个过程

### 1.13. spark的stage划分是怎么实现的？拓扑排序？怎么实现？还有什么算法实现？

### 1.14. spark rpc，spark2.0为啥舍弃了akka，而用netty

### 1.15. spark的各种shuffle，与mapreduce的对比;

### 1.16. spark的各种ha，master的ha，worker的ha，executor的ha，driver的ha,task的ha,在容错的时候对集群或是task有什么影响？

### 1.17.spark的内存管理机制，spark1.6前后对比分析

### 1.18. spark2.0做出了哪些优化？tungsten引擎？cpu与内存两个方面分别说明

### 1.19.spark rdd、dataframe、dataset区别

### 1.20. HashPartitioner与RangePartitioner的实现，以及水塘抽样；

### 1.21. spark有哪几种join，使用场景，以及实现原理

### 1.22. dagschedule、taskschedule、schedulebankend实现原理；

### 1.23. 宽依赖、窄依赖的概念？

### 1.24. Spark数据倾斜，怎么定位、怎么解决（阿里）；

### 1.25 spark有哪些组件？

- master：管理集群和节点，不参与计算。
- worker：计算节点，进程本身不参与计算，和master汇报。
- Driver：运行程序的main方法，创建spark context对象。
- spark context：控制整个application的生命周期，包括dagsheduler和task scheduler等组件。
- client：用户提交程序的入口。

## 2. Spark Streaming

### 1. Spark Streaming和Storm有何区别？ 应用场景？
