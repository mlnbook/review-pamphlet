# Spark

## 1. 基础知识

### 1.1.Spark性能优化主要有哪些手段？

### 1.2. 对于Spark你觉得他对于现有大数据的现状的优势和劣势在哪里？

### 1.3. Spark的Shuffle原理及调优？

![image](http://static.lovedata.net/jpg/2018/6/14/cf900b9cbd52d3a84fd8f04dba7c199f.jpg)

### 1.4.spark如何保证宕机迅速恢复

### 1.5. RDD的原理以及持久化原理

![image](http://static.lovedata.net/jpg/2018/6/14/e224b35496ac4de184008bbb09894893.jpg)

### 1.6. spark排序实现流程，reduce端怎么实现的；

### 1.7.Spark的特点是什么

[Spark 特点 - CSDN博客](https://blog.csdn.net/qq_41455420/article/details/79451722)

1. 快速
    Spark函数（类似于MapReduce）运行的时候，绝大多数的函数是可以在内存里面去迭代的。只有少部分的函数需要落地到磁盘
2. 易用性
    1. 编写简单，支持80种以上的高级算子，支持多种语言，数据源丰富，可部署在多种集群中
3. 通用性
4. 兼容性
5. 容错性高。
    1. Spark引进了弹性分布式数据集RDD (Resilient Distributed Dataset) 的抽象，它是分布在一组节点中的只读对象集合，这些集合是弹性的，如果数据集一部分丢失，则可以根据“血统”（即充许基于数据衍生过程）对它们进行重建。另外在RDD计算时可以通过CheckPoint来实现容错，而CheckPoint有两种方式：CheckPoint Data，和Logging The Updates，用户可以控制采用哪种方式来实现容错。

### 1.8.Spark的三种提交模式是什么

- local(本地模式)：常用于本地开发测试，本地还分为local单线程和local-cluster多线程;
- standalone(集群模式)：典型的Mater/slave模式，不过也能看出Master是有单点故障的；Spark支持ZooKeeper来实现 HA
- on yarn(集群模式)： 运行在 yarn 资源管理器框架之上，由 yarn 负责资源管理，Spark 负责任务调度和计算
- on mesos(集群模式)： 运行在 mesos 资源管理器框架之上，由 mesos 负责资源管理，Spark 负责任务调度和计算
- on cloud(集群模式)：比如 AWS 的 EC2，使用这个模式能很方便的访问 Amazon的 S3;Spark 支持多种分布式存储系统：HDFS 和 S3

### 1.9.spark 实现高可用性：High Availability？

### 1.10. spark中怎么解决内存泄漏问题？

[Spark面对OOM问题的解决方法及优化总结 - CSDN博客](https://blog.csdn.net/yhb315279058/article/details/51035631)

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

### 1.26 Spark的适用场景

- 复杂的批量处理（Batch Data Processing），偏重点在于处理海量数据的能力，至于处理速度可忍受，通常的时间可能是在数十分钟到数小时；
- 基于历史数据的交互式查询（Interactive Query），通常的时间在数十秒到数十分钟之间
- 基于实时数据流的数据处理（Streaming Data Processing），通常在数百毫秒到数秒之间

### 1.27 spark 架构

![spark基础运行架构](http://static.lovedata.net/jpg/2018/6/14/a6f6512145359189c2a4e9f9afac7673.jpg)

![spark结合yarn集群背后的运行流程](http://static.lovedata.net/jpg/2018/6/14/34668f2bad595ff835fdf0823dfb99c6.jpg)

![image](http://static.lovedata.net/jpg/2018/6/14/3a907f0488496c7a39fb6bec02966e25.jpg)

## 2 Spark 优化

### 2.1 reduceByKey或者aggregateByKey与groupByKey的区别？

因为reduceByKey和aggregateByKey算子都会使用用户自定义的函数对每个节点本地的相同key进行预聚合。而groupByKey算子是不会进行预聚合的，全量的数据会在集群的各个节点之间分发和传输，性能相对来说比较差。

比如如下两幅图，就是典型的例子，分别基于reduceByKey和groupByKey进行单词计数。其中第一张图是groupByKey的原理图，可以看到，没有进行任何本地聚合时，所有数据都会在集群节点之间传输；第二张图是reduceByKey的原理图，可以看到，每个节点本地的相同key数据，都进行了预聚合，然后才传输到其他节点上进行全局聚合。

![image](http://static.lovedata.net/jpg/2018/6/14/31d0199949271ef1641a7be918818fcd.jpg)

### 2.2 如何使用高性能的算子？

1. 使用reduceByKey/aggregateByKey替代groupByKey
2. 使用mapPartitions替代普通map 可能出现OOM 因为可能一个分区数据量太大
3. 使用foreachPartitions替代foreach 比如讲分区数据写入到mysql
4. 使用filter之后进行coalesce操作 减少分区的数量
5. 使用repartitionAndSortWithinPartitions替代repartition与sort类操作 
    1. repartitionAndSortWithinPartitions是Spark官网推荐的一个算子，官方建议，如果需要在repartition重分区之后，还要进行排序，建议直接使用repartitionAndSortWithinPartitions算子。因为该算子可以一边进行重分区的shuffle操作，一边进行排序。shuffle与sort两个操作同时进行，比先shuffle再sort来说，性能可能是要高的。

参考
[Spark性能优化指南——基础篇 -](https://tech.meituan.com/spark-tuning-basic.html)

### 2.3 Spark Shuffle 数据倾斜的解决方案

1. 解决方案一：使用Hive ETL预处理数据 适用Hive join
2. 解决方案二：过滤少数导致倾斜的key  
3. 解决方案三：提高shuffle操作的并行度  spark.sql.shuffle.partitions   reduceByKey(1000)
4. 解决方案四：两阶段聚合（局部聚合+全局聚合）打随机数，预聚合，然后去掉随机数，在进行全局聚合  适用于聚合类型，如果是join类的shuffle操作，还得用其他的解决方案。
   1. ![image](http://static.lovedata.net/jpg/2018/6/14/6d4482e3b2738463ed2ce881be07244b.jpg)
5. 解决方案五：将reduce join转为map join  在对RDD使用join类操作，或者是在Spark SQL中使用join语句时，而且join操作中的一个RDD或表的数据量比较小（比如几百M或者一两G），比较适用此方案。
    1. 不使用join算子进行连接操作，而使用Broadcast变量与map类算子实现join操作，进而完全规避掉shuffle类的操作，彻底避免数据倾斜的发生和出现。
    2. 对join操作导致的数据倾斜，效果非常好，因为根本就不会发生shuffle，也就根本不会发生数据倾斜。  适用场景较少，因为这个方案只适用于一个大表和一个小表的情况
6. 解决方案六：采样倾斜key并分拆join操作
    1. 采样倾斜key并分拆join操作
    方案适用场景：两个RDD/Hive表进行join的时候，如果数据量都比较大，无法采用“解决方案五”，那么此时可以看一下两个RDD/Hive表中的key分布情况。如果出现数据倾斜，是因为其中某一个RDD/Hive表中的少数几个key的数据量过大，而另一个RDD/Hive表中的所有key都分布比较均匀，那么采用这个解决方案是比较合适的。
    方案实现思路：
        对包含少数几个数据量过大的key的那个RDD，通过sample算子采样出一份样本来，然后统计一下每个key的数量，计算出来数据量最大的是哪几个key。
        然后将这几个key对应的数据从原来的RDD中拆分出来，形成一个单独的RDD，并给每个key都打上n以内的随机数作为前缀，而不会导致倾斜的大部分key形成另外一个RDD。
        接着将需要join的另一个RDD，也过滤出来那几个倾斜key对应的数据并形成一个单独的RDD，将每条数据膨胀成n条数据，这n条数据都按顺序附加一个0~n的前缀，不会导致倾斜的大部分key也形成另外一个RDD。
        再将附加了随机前缀的独立RDD与另一个膨胀n倍的独立RDD进行join，此时就可以将原先相同的key打散成n份，分散到多个task中去进行join了。
        而另外两个普通的RDD就照常join即可。
        最后将两次join的结果使用union算子合并起来即可，就是最终的join结果。
        ![image](http://static.lovedata.net/jpg/2018/6/14/de04f1729bec2ba1c5b6569952473d38.jpg)
6. 解决方案七：使用随机前缀和扩容RDD进行join


[Spark性能优化指南——高级篇 -](https://tech.meituan.com/spark-tuning-pro.html)