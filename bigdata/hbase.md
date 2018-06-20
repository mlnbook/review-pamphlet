# Hbase

![知识图谱](http://static.lovedata.net/jpg/2018/6/20/7620334c24d3e79d5ec4954bd5003e87.jpg)

## 1.hbase 架构讲解

![image](http://static.lovedata.net/jpg/2018/6/20/08bd66f5cd400fe609a745de9bd16dab.jpg)

参考
[深入HBase架构解析（一） - 上善若水 - BlogJava](http://www.blogjava.net/DLevin/archive/2015/08/22/426877.html)

1. HMaster
    1. 管理HRegionServer，实现其负载均衡
    2. 管理和分配HRegion，比如在HRegion split时分配新的HRegion；在HRegionServer退出时迁移其内的HRegion到其他HRegionServer上。
    3. 实现DDL操作（Data Definition Language，namespace和table的增删改，column familiy的增删改等）。
    4. 管理namespace和table的元数据（实际存储在HDFS上）。
    5. 权限控制（ACL）。
    6. 使用ZK做HA，一般启动了两个
    7. HMaster 通过监听ZooKeeper中的Ephemeral节点(默认：/hbase/rs/*)来监控HRegionServer的加入和宕机。在第一个HMaster连接到ZooKeeper时会创建Ephemeral节点(默认：/hbasae/master)来表示Active的HMaster，其后加进来的HMaster则监听该Ephemeral节点，如果当前Active的HMaster宕机，则该节点消失，因而其他HMaster得到通知，而将自身转换成Active的HMaster，在变为Active的HMaster之前，它会创建在/hbase/back-masters/下创建自己的Ephemeral节点
    8. ![image](http://static.lovedata.net/jpg/2018/6/20/b6ce91f9b5887b8dd2d53917bd50cd63.jpg)

2. HRegionServer
    1. 存放和管理本地HRegion。
    2. 读写HDFS
    3. Client直接通过HRegionServer读写数据
    4. HRegionServer一般和DataNode在同一台机器上运行，实现数据的本地性
    5. ![image](http://static.lovedata.net/jpg/2018/6/20/95477e9effd6513255d3abbc0e843f16.jpg)
    6. 虽然上面这张图展现的是最新的HRegionServer的架构(但是并不是那么的精确)，但是我一直比较喜欢看以下这张图，即使它展现的应该是0.94以前的架构。![image](http://static.lovedata.net/jpg/2018/6/20/1775a7af63906eb6ef195615389a81cf.jpg)
3. ZooKeeper集群是协调系统
    1. 存放整个 HBase集群的元数据以及集群的状态信息。
    2. ZooKeeper为HBase集群提供协调服务，它管理着HMaster和HRegionServer的状态(available/alive等)
    3. HRegionServer中的HRegion
    4. 实现HMaster主从节点的failover。
    5. ![image](http://static.lovedata.net/jpg/2018/6/20/1c18c1a9e3bbc3d6c8b5cdd266c09023.jpg)
4. HRegion
    1. HBase使用RowKey将表水平切割成多个HRegion，从HMaster的角度，每个HRegion都纪录了它的StartKey和EndKey
    2. ![image](http://static.lovedata.net/jpg/2018/6/20/3fd3537bfa6fa5e18c36b12d4fe936d0.jpg)
5. Together
    1. ![image](http://static.lovedata.net/jpg/2018/6/20/e7773b0280338de7561b79777ec1f5ba.jpg)



## 2.hbase宕机了如何处理

## 3. Hbase  热点现象及解决办法

## 4. RowKey的设计原则？

rowkey的设计原则：各个列簇数据平衡，长度原则、相邻原则，创建表的时候设置表放入regionserver缓存中，避免自动增长和时间，使用字节数组代替string，最大长度64kb，最好16字节以内，按天分表，两个字节散列，四个字节存储时分毫秒。

## 5. HBase和传统数据库的区别；

## 6. x

## 7. HBase Master和Regionserver的交互；

## 8. HBase的HA，Zookeeper在其中的作用；

## 9. Master宕机的时候，哪些能正常工作，读写数据；

## 10. region分裂的过程？

## 11. Hbase 列簇的设计原则

列族的设计原则：尽可能少（按照列族进行存储，按照region进行读取，不必要的io操作），经常和不经常使用的两类数据放入不同列族中，列族名字尽可能短。

## 12. hbase性能解决方案：

1、hbase怎么给web前台提供接口来访问?
hbase有一个web的默认端口60010，是提供客户端用来访问hbase的

2、hbase有没有并发问题?

3、Hbase中的metastore用来做什么的?
Hbase的metastore是用来保存数据的，其中保存数据的方式有有三种第一种与第二种是本地储存，第三种是远程储存这一种企业用的比较多

4、HBase在进行模型设计时重点在什么地方?一张表中定义多少个Column Family最合适?为什么?
具体看表的数据，一般来说划分标准是根据数据访问频度，如一张表里有些列访问相对频繁，而另一些列访问很少，这时可以把这张表划分成两个列族，分开存储，提高访问效率

5、如何提高HBase客户端的读写性能?请举例说明。
①开启bloomfilter过滤器，开启bloomfilter比没开启要快3、4倍
②Hbase对于内存有特别的嗜好，在硬件允许的情况下配足够多的内存给它
③通过修改hbase-env.sh中的
export HBASE_HEAPSIZE=3000 #这里默认为1000m
④增大RPC数量
通过修改hbase-site.xml中的
hbase.regionserver.handler.count属性，可以适当的放大。默认值为10有点小

6、直接将时间戳作为行健，在写入单个region 时候会发生热点问题，为什么呢?
HBase的rowkey在底层是HFile存储数据的，以键值对存放到SortedMap中。并且region中的rowkey是有序存储，若时间比较集中。就会存储到一个region中，这样一个region的数据变多，其它的region数据很好，加载数据就会很慢。直到region分裂可以解决。

## 13. HBase － 数据写入流程解析

> HBase默认适用于写多读少的应用

客户端

1. 用户提交put请求后，HBase客户端会将put请求添加到本地buffer中，符合一定条件就会通过AsyncProcess异步批量提交 HBase默认设置autoflush=true 可设置为false,没有保护机制
2. 在提交之前，HBase会在元数据表.meta.中根据rowkey找到它们归属的region server，这个定位的过程是通过HConnection的locateRegion方法获得的
3. HBase会为每个HRegionLocation构造一个远程RPC请求MultiServerCallable<Row>，然后通过rpcCallerFactory.<MultiResponse> newCaller()执行调用

服务端

> 服务器端RegionServer接收到客户端的写入请求后，首先会反序列化为Put对象，然后执行各种检查操作，比如检查region是否是只读、memstore大小是否超过blockingMemstoreSize 

![image](http://static.lovedata.net/jpg/2018/6/20/183daf0ec61cdcf47aaff6fe85ae9389.jpg)

1. 获取行锁、Region更新共享锁 同行数据的原子性
2. 开始写事务：获取write number，用于实现MVCC，实现数据的非锁定读，在保证读写一致性的前提下提高读取性能
3. 写缓存memstore：HBase中每列族都会对应一个store，用来存储该列数据。每个store都会有个写缓存memstore，用于缓存写入数据。HBase并不会直接将数据落盘，而是先写入缓存，等缓存满足一定大小之后再一起落盘。
4. Append HLog：HBase使用WAL机制保证数据可靠性，即首先写日志再写缓存，即使发生宕机，也可以通过恢复HLog还原出原始数据。
5. 释放行锁以及共享锁
6. Sync HLog：HLog真正sync到HDFS，在释放行锁之后执行sync操作是为了尽量减少持锁时间，提升写性能 如果Sync失败，执行回滚操作将memstore中已经写入的数据移除。
7. 结束写事务
8. flush memstore：当写缓存满64M之后，会启动flush线程将数据刷新到硬盘

参考
[HBase － 数据写入流程解析 – 有态度的HBase/Spark/BigData](http://hbasefly.com/2016/03/23/hbase_writer/)

## 14. HBase 数据读取流程

> 查询比较复杂，一是因为整个HBase存储引擎基于LSM-Like树实现   其二是因为HBase中更新操作以及删除操作实现都很简单，更新操作并没有更新原有数据  是插入了一条打上”deleted”标签的数据，而真正的数据删除发生在系统异步执行Major_Compact的时候 但是对于数据读取来说却意味着套上了层层枷锁

![image](http://static.lovedata.net/jpg/2018/6/20/d0f9a3466084169a700b73db005584d6.jpg)

![image](http://static.lovedata.net/jpg/2018/6/20/d358a28eb3558b62c1ee23169f5b0620.jpg)

![客户端缓存RegionServer地址信息](http://static.lovedata.net/jpg/2018/6/20/3f6ec491d3504cbb4a163fa7ff7b0998.jpg)

scan数据就和开发商盖房一样，也是分成两步：组建施工队体系，明确每个工人的职责；一层一层盖楼。

 scanner体系的核心在于三层scanner：RegionScanner、StoreScanner以及StoreFileScanner。三者是层级的关系，一个RegionScanner由多个StoreScanner构成，一张表由多个列族组成，就有多少个StoreScanner负责该列族的数据扫描。一个StoreScanner又是由多个StoreFileScanner组成。每个Store的数据由内存中的MemStore和磁盘上的StoreFile文件组成，相对应的，StoreScanner对象会雇佣一个MemStoreScanner和N个StoreFileScanner来进行实际的数据读取，每个StoreFile文件对应一个StoreFileScanner，注意：StoreFileScanner和MemstoreScanner是整个scan的最终执行者。 

![image](http://static.lovedata.net/jpg/2018/6/20/931c83f325dd513936b66fafdf085282.jpg)

 HBase中KeyValue是什么样的结构？

 ![image](http://static.lovedata.net/jpg/2018/6/20/9d1f2731d5383ed8461fdf2a8908ee8c.jpg)

参考
[HBase原理－迟到的‘数据读取流程’部分细节 – 有态度的HBase/Spark/BigData](http://hbasefly.com/2017/06/11/hbase-scan-2/)
[HBase原理－数据读取流程解析 – 有态度的HBase/Spark/BigData](http://hbasefly.com/2016/12/21/hbase-getorscan/)

## 15. Hbase BlockCache的理解

BlockCache也称为读缓存，HBase会将一次文件查找的Block块缓存到Cache中，以便后续同一请求或者邻近数据查找请求直接从内存中获取，避免昂贵的IO操作，重要性不言而喻。BlockCache有两种实现机制：LRUBlockCache和BucketCache（通常是off-heap）

1. BlockCache的内容