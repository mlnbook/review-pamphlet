# Spark

## 基础知识

### 1.Spark性能优化主要有哪些手段？

### 2. 对于Spark你觉得他对于现有大数据的现状的优势和劣势在哪里？

### 3. Spark的Shuffle原理及调优？

### 4.spark如何保证宕机迅速恢复?

### 5. RDD持久化原理

### 6.Spark Streaming和Storm有何区别？ 应用场景？

### 7.Spark的特点是什么

### 8.Spark的三种提交模式是什么

### 9.spark 实现高可用性：High Availability？

### 10. spark中怎么解决内存泄漏问题？

### 11. Spark-submit模式yarn-cluster和yarn-client的区别

1. yarn-client用于测试，因为他的Driver运行在本地客户端，会与yarn集群产生较大的网络通信，从而导致网卡流量激增；它的好处在于直接执行时，在本地可以查看到所有的log，方便调试；
2. yarn-cluster用于生产环境，因为Driver运行在NodeManager，相当于一个ApplicationMaster，没有网卡流量激增的问题；缺点在于调试不方便，本地用spark-submit提交后，看不到log，只能通过yarn application_id这种命令来查看，很麻烦

## Spark Streaming