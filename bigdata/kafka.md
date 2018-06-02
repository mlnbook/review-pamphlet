# Kafka

## 1. kafka 新旧API的特点？

[Kafka 0.9 新消费者API](https://www.cnblogs.com/admln/p/5446361.html)

## 2. kafka 新版API auto.offset.reset 的含义

### 2.1 earliest

当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费

### 2.2 latest

当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据

### 2.3 none

topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常

[Kafka auto.offset.reset值详解](https://blog.csdn.net/lishuangzhe7047/article/details/74530417)

## 3. kafka enable.auto.commit 的含义

是否由客户端自动提交offset

## 4. kafka 版本的演进？

[kafka各个版本特性预览介绍](https://blog.csdn.net/hesi9555/article/details/70237744)

### 0.8.2 之前

1. kafka删除topic的功能存在bug
2. comsumer定期提交已经消费的kafka消息的offset位置到zookeeper中保存，对zookeeper而言，每次写操作代价是很昂贵的，而且zookeeper集群是不能扩展写能力的

### 0.8.2

1. 可以把comsumer提交的offset记录在compacted topic（__comsumer_offsets）中，该topic设置最高级别的持久化保证，即ack=-1
    1. __consumer_offsets由一个三元组< comsumer group, topic, partiotion> 组成的key和offset值组成，在内存也维持一个最新的视图view，所以读取很快。
    2. kafka可以频繁的对offset做检查点checkpoint，即使每消费一条消息提交一次offset。
    3. 在0.8.1中，已经实验性的加入这个功能，0.8.2中可以广泛使用.

### 0.9

1. 安全方面改进(在0.9之前，Kafka安全方面的考虑几乎为0，在进行外网传输时，只好通过Linux的防火墙、或其他网络安全方面进行配置。相信这一点，让很多用户在考虑使用Kafka进行外网消息交互时有些担心。)
    1. 客户端连接borker使用SSL或SASL进行验证
    2. borker连接ZooKeeper进行权限管理
    3. 数据传输进行加密（需要考虑性能方面的影响）
    4. 客户端读、写操作可以进行授权管理
    5. 可以对外部的可插拔模块的进行授权管理
2. Kafka Connect 它可以和外部系统、数据集建立一个数据流的连接，实现数据的输入、输出。有以下特性：
    1. 使用了一个通用的框架，可以在这个框架上非常方便的开发、管理Kafka Connect接口
    2. 支持分布式模式或单机模式进行运行
    3. 支持REST接口，可以通过REST API提交、管理 Kafka Connect集群
    4. offset自动管理
3. 新的Comsumer API
    1. 新的Comsumer API不再有high-level、low-level之分了，而是自己维护offset 这样做的好处是避免应用出现异常时，数据未消费成功但Position已经提交，导致消息未消费的情况发生
    2. Kafka可以自行维护Offset、消费者的Position。也可以开发者自己来维护Offset，实现相关的业务需求。
    3. 消费时，可以只消费指定的Partitions
    4. 可以使用外部存储记录Offset，如数据库之类的
    5. 可以使用多线程进行消费

### 0.10

1. 机架感知以便隔离副本 跨域多个机架或者可用区域，提高弹性和可用性
2. 所有消息包含了时间戳字段
3. Kafka Consumer Max Records，在Kafka 0.9.0.0，开发者们在新consumer上使用poll()函数的时候是几乎无法控制返回消息的条数。不过值得高兴的是，此版本的Kafka引入了max.poll.records参数，允许开发者控制返回消息的条数。

## 5. kafka 的leader 选举机制是怎样实现的以及各个版本的实现？

1. leader是对应partition的概念，每个partition都有一个leader。
2. 客户端生产消费消息都是只跟leader交互 (实现上简单。)
3. ISR（In-Sync Replicas）直译就是跟上leader的副本
4. ![image](http://static.lovedata.net/jpg/2018/5/29/8e14780235f97056c60785dce43f18e7.jpg)
    1. High watermark（高水位线）以下简称HW，表示消息被leader和ISR内的follow都确认commit写入本地log，所以在HW位置以下的消息都可以被消费（不会丢失）
    2. Log end offset（日志结束位置）以下简称LEO，表示消息的最后位置。LEO>=HW，一般会有没提交的部分。
5. 副本会有单独的线程（ReplicaFetcherThread），去从leader上去拉去消息同步。当follower的HW赶上leader的，就会保持或加入到 **ISR** 列表里，就说明此follower满足上述最基本的原则（跟上leader进度）。ISR列表存在zookeeper上。
    1. replica.lag.max.messages 落后的消息个数 （Kafka 0.10.0 移除了落后消息个数参数（replica.lag.max.messages），原因是这个值不好把控，需要经验值，不用的业务服务器环境，这个值可能不同，不然会频繁的移除加入ISR列表） [Kafka副本管理—— 为何去掉replica.lag.max.messages参数 - huxihx - 博客园](http://www.cnblogs.com/huxi2b/p/5903354.html)
    2. replica.lag.time.max.ms 多长时间没有发送FetchQuest请求拉去leader数据
6. producer的ack参数选择，取优先考虑可靠性，还是优先考虑高并发。以下不用的参数，会导致有可能丢消息。
    - 0表示纯异步，不等待，写进socket buffer就继续。
    - 1表示leader写进server的本地log，就返回，不等待follower的回应。
    - -1相当于all，表示等待follower回应再继续发消息。保证了ISR列表里至少有一个replica，数据就不会丢失，最高的保证级别。
7. 参考
    1. [kafka的HA设计 - 简书](https://www.jianshu.com/p/83066b4df739)
    2. [kafka的leader选举过程 - 简书](https://www.jianshu.com/p/c987b5e055b0)
8. unclean.leader.election.enable 默认是true，表示允许不在ISR列表的follower，选举为leader（最坏的打算，可能丢消息）

## 6. kafka 的 rebalance 是怎样的？

## 7. kafka中的offset状态，以及high.watermark是什么意思

![image](http://static.lovedata.net/jpg/2018/5/25/c2fa3b250b6512a80279e8140b1421d7.jpg)

例如，在下图中，消费者的位置在偏移6，其最后的提交的偏移1.
当分区重新分配给组中的另外一个使用者时，初始位置设置为最后一个已提交的偏移量。如果上面例子中的消费者突然崩溃了，那么接管的组成员将从偏移量1开始消费。在这种情况下，它必须重新处理消息直到崩溃消费者的位置6.

该图还显示了日志中的另外两个重要位置。日志结束偏移量是写入日志的最后一条消息的偏移量。高水印是成功复制到所有日志副本的最后一条消息的偏移量。从消费者的角度来看，主要知道的是，你只能读取高水印。这防止消费者读取稍后可能丢失的未复制数据。

## 8. Kafak本身提供的新的组协调协议是怎样的机制？

<<<<<<< HEAD
## 9. 如果Zookeeper宕机了，kafka还能用吗？
=======
## 9. kafka 使用场景？

- 日志收集：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等。
- 消息系统：解耦和生产者和消费者、缓存消息等。
- 用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。
- 运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。
- 流式处理：比如spark streaming和storm
- 事件源

[Kafka史上最详细原理总结 | 静水流深](http://www.thinkyixia.com/2017/10/25/kafka-2/)

## 10. partition和replica默认分配到哪个broker的策略?

- 将所有N Broker和待分配的i个Partition排序.
- 将第i个Partition分配到第(i mod n)个Broker上.
- 将第i个Partition的第j个副本分配到第((i + j) mod n)个Broker上.

## 11. kafka zookeeper中存储结构

1. ![image](http://static.lovedata.net/jpg/2018/5/29/e579c3897235853981bb911ef3328e4e.jpg)
2. ![image](http://static.lovedata.net/jpg/2018/5/29/58462246b8030bb67d3a633305cfe12b.jpg)
3. ![image](http://static.lovedata.net/jpg/2018/5/29/dc69269178701fdeae11e3388340176e.jpg)
>>>>>>> 570551cd93adacfc79159308d80c19dc43d804af
