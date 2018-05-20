# Hadoop

## 1.Mapreduce

### 1.1 Mapreduce的过程

 ![image](http://static.lovedata.net/jpg/2018/5/18/21730e68df257d648a1c17284040c966.jpg)
 1. 由算个阶段构成 **Map、shuffle、Reduce**
 2. Map 是映射，将原始数据转换为键值对
 3. Reduce 是合并，将相同的key值得value进行处理后在输出新的键值对作为最终结果
 4. 将Map输出进行进一步整理并交给Reduce的过程就是Shuffle
 ![Map shuffle](http://static.lovedata.net/jpg/2018/5/18/f29021d32b6c5c447e53e7aebd4e326b.jpg)
 5. [MapReduce shuffle过程详解](https://blog.csdn.net/u014374284/article/details/49205885)

### 1.2 谈谈数据倾斜，如何发生的，并给出优化方案

- 集群中，某个map任务的key对应的value值远远大于其他节点的key所对应的值，导致某个节点mapreduce执行效率很慢，解决根本方法就是避免某个节点上执行任务数据量过大，可以使用map阶段的partiion对过大的数据进行分区，大数据块分成小数据块
- [http://www.docin.com/p-1443821582.html](http://www.docin.com/p-1443821582.html)
- 造成数据倾斜的原因:
  - **分组** 注：group by 优于distinct group
  - 情形：group by 维度过小，某值的数量过多
  - 后果：处理某值的reduce非常耗时
  - **去重** distinct count(distinct xx)
  - 情形：某特殊值过多
  - 后果：处理此特殊值的reduce耗时
  - **连接 join**
  - 情形1：其中一个表较小，但是key集中。
  - 后果1：分发到某一个或几个Reduce上的数据远高于平均值
  - 情形2：大表与大表，但是分桶的判断字段0值或空值过多
  - 后果2：这些空值都由一个reduce处理，非常慢

- Hive join 数据倾斜
  - set hive.map.aggr=true； map端做部分聚合操作，效率高，需要更多内存
  - set hive.groupby.skewindata=true; 生成两个 MRjob
    - 第一个MRJob 中，Map的输出结果集合会**随机**分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的GroupBy Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的
    - 第二个MRJob再根据预处理的数据结果按照GroupBy Key分布到Reduce中（这个过程可以保证相同的GroupBy Key被分布到同一个Reduce中），最后完成最终的聚合操作。

- 在 key 上面做文章，在 map 阶段将造成倾斜的key 先分成多组，例如 aaa 这个 key,map 时随机在 aaa 后面加上 1,2,3,4 这四个数字之一，把 key 先分成四组，先进行一次运算，之后再恢复 key 进行最终运算
- [https://www.zhihu.com/question/27593027](https://www.zhihu.com/question/27593027)

### 1.3 简单概述hadoop的combine与partition的区别

- combine分为map端和reduce端，作用是把同一个key的键值对合并在一起，可以自定义的。减少网络  传输
- partition是分割map每个节点的结果，按照key分别映射给不同的reduce

### 1.4 MapReduce 中排序发生在哪几个阶段？这些排序是否可以避免？为什么？

- 一个MapReduce作业由Map阶段和Reduce阶段两部分组成，这两阶段会对数据排序
- MapReduce框架本质就是一个Distributed Sort
- Map阶段，Map Task会在本地磁盘输出一个按照key排序（采用的是快速排序）的文件（中间可能产生多个文件，但最终会合并成一个）
- 在Reduce阶段，每个Reduce Task会对收到的数据排序
- Map阶段的排序就是为了减轻Reduce端排序负载
- sort对后续操作有何好处 而是这个sort为许多应用和后续应用开发带来很多好处 试想分布式计算框架不提供排序 要你自己排 真是哇哇叫 谁还用
- [https://www.zhihu.com/question/35999547/answer/65443663](https://www.zhihu.com/question/35999547/answer/65443663)
- [https://blog.csdn.net/play_chess_ITmanito/article/details/51089200](https://blog.csdn.net/play_chess_ITmanito/article/details/51089200)

### 1.5 hadoop的shuffer的概念

![image](http://static.lovedata.net/jpg/2018/5/20/9633d38b5494528b083a881c61c6d12a.jpg)

- [MapReduce:详解Shuffle(copy,sort,merge)过程](https://blog.csdn.net/luyee2010/article/details/8624469)
- Shuffle的正常意思是洗牌或弄乱
- shuffle针对多个map任务的输出按照不同的分区（Partition）通过网络复制到不同的reduce任务节点上，这个过程就称作为Shuffle。
- Map端三个步骤
  - map函数产生key value对，输出先放入缓存，缓存区默认100MB(io.sort.mb)，达到0.8阀值，spill到本次磁盘（mappred.local.dir）新建一个溢写文件
  - 写磁盘钱，进行partition sort combine 分区 不同类型分开处理，对不同分区的数据进行排序，排序后的进行combine，最后记录写完，合并为一个分区并排序的文件
  - 等待reducer来拉去
- Reduce端
  - Copy阶段，通过http拉取，从n个map中拉取，速度不尽相同，有的还没弄完
  - Merge阶段  http拉取来的放入缓存，达到阈值写入磁盘，同样进行partition combine、排序等过程。多个文件也会合并，最后一次合并作为reduce最终输入
  - 传入到reduce任务当中
- [Hadoop-Shuffle过程](https://blog.csdn.net/clerk0324/article/details/52461135)

### 1.6 hadoop的二次排序

### 1.7 如何减少Hadoop Map端到Reduce端的数据传输量？

### 1.8 hadoop常见的join操作？

## 2.HDFS

### 2.1 hdfs 的数据压缩算法

### 2.2 文件大小默认为 64M，改为 128M 有啥影响

### 2.3 讲述HDFS上传文件和读文件的流程？

### 2.4 HDFS在上传文件的时候，如果其中一个块突然损坏了怎么办？

### 2.5 HDFS和HBase各自使用场景

## 3.YARN

### 3.1 YARN的新特性

### 3.2 hadoop的调度策略的实现，你们使用的是那种策略，为什么？

## 4.其他

### 4.1 简单概述hadoop中的角色的分配以及功能

### 4.2 hadoop的优化

### 4.3 hadoop1与hadoop2的区别

### 4.4 hadoop3的新特性

### 4.5 hadoop中两个大表实现join的操作，简单描述？