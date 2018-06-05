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
  - 后果：处理此特殊值的reduce耗时。
  - **连接 join**
  - 情形1：其中一个表较小，但是key集中。
  - 后果1：分发到某一个或几个Reduce上的数据远高于平均值
  - 情形2：大表与大表，但是分桶的判断字段0值或空值过多
  - 后果2：这些空值都由一个reduce处理，非常慢。

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

1. Hadoop MapReduce 二次排序原理及其应用
    在0.20.0以前使用的是
        setPartitionerClass
        setOutputkeyComparatorClass
        setOutputValueGroupingComparator
    在0.20.0以后使用是
        job.setPartitionerClass(Partitioner p);
        **job.setSortComparatorClass(RawComparator** c);
        **job.setGroupingComparatorClass(RawComparator** c);
        setGroupingComparatorClass - **就是通过一个comparator比较两个值是否返回0，如果是0，则就表示是一个组中的。**  然后开始构造一个key对应的value迭代器。这时就要用到分组，使用jobjob.setGroupingComparatorClass设置的分组函数类。只要这个比较器比较的两个key相同，他们就属于同一个组，它们的value放在一个value迭代器，而这个迭代器的key使用属于同一个组的所有key的第一个key   **如果不用分组，那么同一组的记录就要在多次reduce方法中独立处理，那么有些状态数据就要传递了，就会增加复杂度，在一次调用中处理的话，这些状态只要用方法内的变量就可以的。比如查找最大值，只要读第一个值就可以了。**
        ![image](http://static.lovedata.net/jpg/2018/6/5/77cdfa80aa37a7f44712c93d0fed25f1.jpg)
2. 参考
    1. [[转]Hadoop MapReduce 二次排序原理及其应用 | 四号程序员](https://www.coder4.com/archives/4248)
    2. [Hadoop SecondrySort 中有了sort为什么要使用setGroupingComparatorClass排序的解释](http://www.360doc.com/content/15/0428/21/23016082_466665862.shtml)
    3. [bigdata-practice/SortMapReduce.java at master · pengshuangbao/bigdata-practice · GitHub](https://github.com/pengshuangbao/bigdata-practice/blob/master/src/main/java/com/lovedata/bigdata/hadoop/sort/secondary/SortMapReduce.java)

### 1.7 如何减少Hadoop Map端到Reduce端的数据传输量？

### 1.8 hadoop常见的链接join操作？

![内连接和外连接](http://static.lovedata.net/jpg/2018/5/24/8f84a6747faa534c0b03a90b356cd383.jpg)

为了实现内连接和外连接，MapReduce中有三种连接策略，如下所示。这三种连接策略有的在map阶段，有的在reduce阶段。它们都针对MapReduce的排序-合并（sort-merge）的架构进行了优化。

- 重分区连接（Repartition join）—— reduce端连接。使用场景：连接两个或多个大型数据集。
- 复制连接（Replication join）—— map端连接。使用场景：待连接的数据集中有一个数据集足够小到可以完全放在缓存中。
- 半连接（Semi-join）—— 另一个map端连接。使用场景：待连接的数据集中有一个数据集非常大，但同时这个数据集可以被过滤成小到可以放在缓存中。

1. Reduce side join
    ![image](http://static.lovedata.net/jpg/2018/5/22/9659eb7d2f3b0b34f51f7bcfabff4d7a.jpg)

    1. **Map阶段**
    读取源表的数据，Map输出时候以Join on条件中的列为key，如果Join有多个关联键，则以这些关联键的组合作为key；Map输出的value为join之后所关心的(select或者where中需要用到的)列，同时在value中还会包含表的Tag信息，用于标明此value对应哪个表。
    2. **Shuffle阶** 
    根据key的值进行hash，并将key/value按照hash值推送至不同的reduce中，这样确保两个表中相同的key位于同一个reduce中。
    3. **Reduce阶段**
      根据key的值完成join操作，期间通过Tag来识别不同表中的数据。

2. Map Side join
  ![image](http://static.lovedata.net/jpg/2018/5/22/8358ada057f8cbd0ba792d3841058bda.jpg)
    - 没有reduce 直接输出结果
    - 独立task 读取小表 放入 DistributeCache

3. SemiJoin
  半连接  SemiJoin，也叫半连接，是从分布式数据库中借鉴过来的方法。它的产生动机是：对于reduce side join，跨机器的数据传输量非常大，这成了join操作的一个瓶颈，如果能够在map端过滤掉不会参加join操作的数据，则可以大大节省网络IO。
  实现方法很简单：选取一个小表，假设是File1，将其参与join的key抽取出来，保存到文件File3中，File3文件一般很小，可以放到内存中。在map阶段，使用DistributedCache将File3复制到各个TaskTracker上，然后将File2中不在File3中的key对应的记录过滤掉，剩下的reduce阶段的工作与reduce side join相同。

  [大牛翻译系列Hadoop（3）MapReduce 连接：半连接（Semi-join）](https://www.cnblogs.com/datacloud/p/3579975.html)
4.  reduce side join + BloomFilter
   在某些情况下，SemiJoin抽取出来的小表的key集合在内存中仍然存放不下，这时候可以使用BloomFiler以节省空间。
**BloomFilter** 最常见的作用是：判断某个元素是否在一个集合里面。它最重要的两个方法是：add() 和contains()。最大的特点是不会存在false negative，即：如果contains()返回false，则该元素一定不在集合中，但会存在一定的true negative，即：如果contains()返回true，则该元素可能在集合中。
因而可将小表中的key保存到BloomFilter中，在map阶段过滤大表，可能有一些不在小表中的记录没有过滤掉（但是在小表中的记录一定不会过滤掉），这没关系，只不过增加了少量的网络IO而已。
更多关于BloomFilter的介绍，可参考：[Bloom Filter概念和原理 - CSDN博客](http://blog.csdn.net/jiaomeng/article/details/1495500)

### 1.9 简答说一下hadoop的map-reduce编程模型

1. map task会从本地文件系统读取数据，转换成key-value形式的键值对集
2. 使用的是hadoop内置的数据类型，比如longwritable、text等
3. 将键值对集合输入mapper进行业务处理过程，将其转换成需要的key-value在输出
4. 之后会进行一个partition分区操作，默认使用的是hashpartitioner，可以通过重写hashpartitioner的getpartition方法来自定义分区规则
5. 之后会对key进行进行sort排序，grouping分组操作将相同key的value合并分组输出，在这里可以使用自定义的数据类型，重写WritableComparator的Comparator方法来自定义排序规则，重写RawComparator的compara方法来自定义分组规则
6. 之后进行一个combiner归约操作，其实就是一个本地段的reduce预处理，以减小后面shufle和reducer的工作量
7. reduce task会通过网络将各个数据收集进行reduce处理，最后将数据保存或者显示，结束整个job

### 1.10 hadoop的TextInputFormat作用是什么，如何自定义实现

InputFormat会在map操作之前对数据进行两方面的预处理

1. 是 **getSplits** ，返回的是InputSplit数组，对数据进行split分片，每片交给map操作一次
2. 是getRecordReader，返回的是RecordReader对象，对每个split分片进行转换为key-value键值对格式传递给map

常用的InputFormat是TextInputFormat，使用的是LineRecordReader对每个分片进行键值对的转换，以行偏移量作为键，行内容作为值
自定义类继承InputFormat接口，重写createRecordReader和isSplitable方法
在createRecordReader中可以自定义分隔符

### 1.11 hadoop和spark的都是并行计算，那么他们有什么相同和区别

1. 相同点
   1. 都是用mr模型来进行并行计算 
2. 不同点
   1. hadoop的一个作业称为job，job里面分为map task和reduce task，每个task都是在自己的进程中运行的，当task结束时，进程也会结束
   2. spark用户提交的任务成为application，一个application对应一个sparkcontext，app中存在多个job，每触发一次action操作就会产生一个job,每个job中有多个stage，stage是shuffle过程中DAGSchaduler通过RDD之间的依赖关系划分job而来的，每个stage里面有多个task，组成taskset有TaskSchaduler分发到各个executor中执行
   3. **hadoop的job只有map和reduce操作，表达能力比较欠缺而且在mr过程中会重复的读写hdfs，造成大量的io操作，多个job需要自己管理关系**
   4. **spark的迭代计算都是在内存中进行的，API中提供了大量的RDD操作如join，groupby等，而且通过DAG图可以实现良好的容错**

### 1.12 map-reduce程序运行的时候会有什么比较常见的问题

比如说作业中大部分都完成了，但是总有几个reduce一直在运行
这是因为这几个reduce中的处理的数据要远远大于其他的reduce，可能是因为对键值对任务划分的不均匀造成的数据倾斜
解决的方法可以在分区的时候重新定义分区规则对于value数据很多的key可以进行拆分、均匀打散等处理，或者是在map端的combiner中进行数据预处理的操作

### 1.13 WritableComparator 如何使用

## 2.HDFS

### 2.1 hdfs 的数据压缩算法

### 2.2 ~~文件大小默认为 64M，改为 128M 有啥影响~~ bloack大小为什么增大默认为128M

1. 减轻了namenode的压力
  原因是hadoop集群在启动的时候，datanode会上报自己的block的信息给namenode。namenode把这些信息放到内存中。那么如果块变大了，那么namenode的记录的信息相对减少，所以namenode就有更多的内存去做的别的事情，使得整个集群的性能增强。
2. 增大会不会带来负面相应。
  因为这个可以灵活设置，所以这里不是问题。关键是什么时候，该如何设置。
  如果对于数两级别为PB的话，建议可以block设置的大一些。
  如果数据量相对较少，可以设置的小一些64M也未尝不可。
3. 参考
   1. [Hadoop-2.X中HDFS文件块bloack大小为什么增大默认为128M-Hadoop|YARN-about云开发-活到老 学到老](http://www.aboutyun.com/thread-7514-1-1.html)

### 2.3 讲述HDFS上传文件和读文件的流程？

### 2.3.1 读数据流程

![image](http://static.lovedata.net/jpg/2018/5/31/459e0d017d85b47b3e1380b1985d2225.jpg)

1. 客户端(Client)调用FileSystem的open()函数打开文件。
2. DistributeFileSystem通过RPC调用元数据节点，得到文件的数据块信息。 **对于每一个数据块，元数据节点返回数据块的数据节点位置。**
3. DistributedFileSystem返回FSDataInputStream给客户端，用来读取数据。客户端调用stream的read()方法读取数据。
4. FSDataInputStream连接保存此文件第一个数据块的 **最近的数据节点** ，Data从数据节点读到客户端(Client)。
5. 当此数据块读取完毕后，FSDataInputStream关闭和此数据节点的连接，然后读取保存下一个数据块的最近的数据节点。
6. 当数据读取完毕后，调用FSDataInputStream的close()函数。
7. 在数据读取过程中，如果客户端在与数据节点通信时出现错误， **则会尝试读取包含有此数据块的下一个数据节点** ，并且失败的数据节点会被记录，以后不会再连接

### 2.3.2 写数据流程

![image](http://static.lovedata.net/jpg/2018/5/31/3d11422458fe21f376f2f478a7bf1073.jpg)

1. 客户端调用create()函数创建文件。
2. DistributedFileSystem通过RPC调用 **元数据节点** ，在文件系统的 **命名空间** 中创建一个新的文件。元数据节点会首先确定文件原先不存在，并且客户端有创建文件的权限，然后创建新文件。
3. DistributedFileSystem返回FSDataOutputStream，客户端用于写数据。
4. **FSDataOutputStream将数据分成块** ，写入Data Queue。Data Queue由Data Streamer读取，并通知元数据节点分配数据节点用来存储数据块(每块默认复制3份)。分配的数据节点放在一个Pipeline中。Data Streamer将数据块写入Pipeline中的第一个数据节点；第一个数据节点再将数据块发送给第二个数据节点；第二个数据节点再将数据发送给第三个数据节点。
5. FSDataoutputStream为发出去的数据块保存了 ACK Queue ,等待Pipeline中的数据节点告知数据已成功写入。如果数据节点在写入过程中失败了，则关闭Pipeline，将Ack Queue中的数据块放入到Data Queue的开始。
6. 当前数据块在已经写入的数据节点中会被元数据节点赋予新的标识，则错误节点重启后能察觉到其数据块是过时的，将会被删除。失败的数据节点从Pipeline中移除，另外的数据块则写入Pipeline中的另外两个数据节点。元数据节点则被通知此数据块复制块数不足，将来会再创建第三份备份。
7. 当客户端结束写入数据后，则调用stream的close()方法。此操作将所有的数据块写入pipeline中的数据节点，并等待ACK Queue成功返回。最后通知元数据节点写入完毕。

参考

1. [HDFS文件读取、写入过程详解 - CSDN博客](https://blog.csdn.net/xu__cg/article/details/68106221)

### 2.4 HDFS在上传文件的时候，如果其中一个块突然损坏了怎么办？（读取文件的异常处理）

[HDFS 异常处理与恢复 - mindwind - 博客园](https://www.cnblogs.com/mindwind/p/4833098.html)

### 2.5 HDFS和HBase各自使用场景

HBase作为面向列的数据库运行在HDFS之上，HDFS缺乏随即读写操作，HBase正是为此而出现,以键值对的形式存储。项目的目标就是快速在主机内数十亿行数据中定位所需的数据并访问它。
HBase是一个数据库，一个NoSql的数据库，像其他数据库一样提供随即读写功能，Hadoop不能满足实时需要，HBase正可以满足。如果你需要实时访问一些数据，就把它存入HBase

#### 什么场景下应用Hbase

1. 成熟的数据分析主题，查询模式已经确立，并且不会轻易改变。
2. 传统的关系型数据库已经无法承受负荷，高速插入，大量读取。
3. 适合海量的，但同时也是简单的操作(例如：key-value)。
4. **半结构化或非结构化数据**  对于数据结构字段不够确定或杂乱无章很难按一个概念去进行抽取的数据适合用HBase。以上面的例子为例，当业务发展需要存储author的email，phone，address信息时RDBMS需要停机维护，而HBase支持动态增加.
5. 记录非常稀疏  RDBMS的行有多少列是固定的，为null的列浪费了存储空间。而如上文提到的，HBase为null的Column不会被存储，这样既节省了空间又提高了读性能。
6. 多版本数据 如上文提到的根据Row key和Column key定位到的Value可以有任意数量的版本值，因此对于需要存储变动历史记录的数据，用HBase就非常方便了。比如上例中的author的Address是会变动的，业务上一般只需要最新的值，但有时可能需要查询到历史值。
7. 超大数据量 当数据量越来越大，RDBMS数据库撑不住了，就出现了读写分离策略，通过一个Master专门负责写操作，多个Slave负责读操作，服务器成本倍增。随着压力增加，Master撑不住了，这时就要分库了，把关联不大的数据分开部署，一些join查询不能用了，需要借助中间层。随着数据量的进一步增加，一个表的记录越来越大，查询就变得很慢，于是又得搞分表，比如按ID取模分成多个表以减少单个表的记录数。经历过这些事的人都知道过程是多么的折腾。采用HBase就简单了，只需要加机器即可，HBase会自动水平切分扩展，跟Hadoop的无缝集成保障了其数据可靠性（HDFS）和海量数据分析的高性能（MapReduce）。

#### hadoop主要应用于数据量大的离线场景。特征为：

1. 数据量大。一般真正线上用Hadoop的，集群规模都在上百台到几千台的机器。这种情况下，T级别的数据也是很小的。Coursera上一门课了有句话觉得很不错：Don't use hadoop, your data isn't that big
2. 离线。Mapreduce框架下，很难处理实时计算，作业都以日志分析这样的线下作业为主。另外，集群中一般都会有大量作业等待被调度，保证资源充分利用。
3. 数据块大。由于HDFS设计的特点，Hadoop适合处理文件块大的文件。大量的小文件使用Hadoop来处理效率会很低。举个例子，百度每天都会有用户对侧边栏广告进行点击。这些点击都会被记入日志。然后在离线场景下，将大量的日志使用Hadoop进行处理，分析用户习惯等信息。

#### Hbase 八大场景

- 对象存储：我们知道不少的头条类、新闻类的的新闻、网页、图片存储在HBase之中，一些病毒公司的病毒库也是存储在HBase之中
- 时序数据：HBase之上有OpenTSDB模块，可以满足时序类场景的需求
- 推荐画像：特别是用户的画像，是一个比较大的稀疏矩阵，蚂蚁的风控就是构建在HBase之上
- 时空数据：主要是轨迹、气象网格之类，滴滴打车的轨迹数据主要存在HBase之中，另外在技术所有大一点的数据量的车联网企业，数据都是存在HBase之中
- CubeDB OLAP：Kylin一个cube分析工具，底层的数据就是存储在HBase之中，不少客户自己基于离线计算构建cube存储在hbase之中，满足在线报表查询的需求
- 消息/订单：在电信领域、银行领域，不少的订单查询底层的存储，另外不少通信、消息同步的应用构建在HBase之上
- Feeds流：典型的应用就是xx朋友圈类似的应用
- NewSQL：之上有Phoenix的插件，可以满足二级索引、SQL的需求，对接传统数据需要SQL非事务的需求

#### 不适用于HDFS的场景：

1) 低延迟
HDFS不适用于实时查询这种对延迟要求高的场景，例如：股票实盘。往往应对低延迟数据访问场景需要通过数据库访问索引的方案来解决，Hadoop生态圈中的Hbase具有这种随机读、低延迟等特点。

2) 大量小文件
对于Hadoop系统，小文件通常定义为远小于HDFS的block size（默认64MB）的文件，由于每个文件都会产生各自的MetaData元数据，Hadoop通过Namenode来存储这些信息，若小文件过多，容易导致Namenode存储出现瓶颈。

3) 多用户更新
为了保证并发性，HDFS需要一次写入多次读取，目前不支持多用户写入，若要修改，也是通过追加的方式添加到文件的末尾处，出现太多文件需要更新的情况，Hadoop是不支持的。
针对有多人写入数据的场景，可以考虑采用Hbase的方案。

4) 结构化数据
HDFS适合存储半结构化和非结构化数据，若有严格的结构化数据存储场景，也可以考虑采用Hbase的方案。

5) 数据量并不大

参考

1. [区分 hdfs hbase hive hbase适用场景 - 李玉龙 - 博客园](https://www.cnblogs.com/liyulong1982/p/6001822.html)
2. [（第3篇）HDFS是什么？HDFS适合做什么？我们应该怎样操作HDFS系统？ - 何石-博客 - 博客园](https://www.cnblogs.com/shijiaoyun/p/6761637.html)
3. [hbase常识及habse适合什么场景 - 天下尽好 - 博客园](https://www.cnblogs.com/Little-Li/p/7878219.html)

### 2.6 Hadoop namenode的ha，主备切换实现原理，日志同步原理，QJM中用到的分布式一致性算法（就是paxos算法）

## 3.YARN

### 3.1 YARN的新特性

### 3.2 hadoop的调度策略的实现，你们使用的是那种策略，为什么？

### 3.3 画一个yarn架构图，及其通信流程；

## 4.其他

### 4.1 简单概述hadoop中的角色的分配以及功能

### 4.2 hadoop的优化（性能调优）

### 4.3 hadoop1与hadoop2的区别

### 4.4 hadoop3的新特性

### 4.5 hadoop中两个大表实现join的操作，简单描述？