# Impala

## 1. impala 特点

[Kudu+Impala介绍 | 微店数据科学团队博客](https://juejin.im/entry/5a72d3d1f265da3e4d730b37)

1. Impala作为老牌的SQL解析引擎，其面对即席查询(Ad-Hoc Query)类请求的稳定性和速度在工业界得到过广泛的验证
2. 没有存储，负责解析sql
3. 定位类似hive，关注即席查询sql快速解析
4. 长sql还是hive更合适
5. group by sql 使用内存计算，建议内存128G以上  **Hive使用MR，效率低，稳定性好**
6. 不支持高并发，由于单个sql执行代价较高
7. 不能代替hive
8. 至少需要128G以上内存，并且把80%分配给Impala
9. 不会对表数据Cache，仅仅缓存一些表数据等元数据
10. 可以使用 hive metastore **Hive 是必选组件**

## 2. Impala为什么会这么快？

为速度而生，执行效率优化，非MR模型，MR sql转换为MR原语，需要多层迭代，造成极大浪费

- impala尽可能把数据缓存在内存中，数据不落地就能完成sql查询
- 常驻进程，避免MR启动开销
- 专为SQL设计，减少迭代次数，避免不必要shuffle和sort
- 利用现代化高性能服务器
  - LLVM 生产动态代码
  - 协调控制磁盘io，控制吞吐，吞吐量最大化
  - 代码效率曾采用C++，提速
  - 程序内存使用上，利用C++天然优势，遵循极少内存使用原则

## 3. Impala 查询流程

[大数据时代快速SQL引擎-Impala](https://blog.csdn.net/yu616568/article/details/52431835)

![image](http://static.lovedata.net/jpg/2018/5/21/84f8934b8517992c953bdf693d06b162.jpg)

下图展示了执行select t1.n1, t2.n2, count(1) as c from t1 join t2 on t1.id = t2.id join t3 on t1.id = t3.id where t3.n3 between ‘a’ and ‘f’ group by t1.n1, t2.n2 order by c desc limit 100;查询的执行逻辑，首先Query Planner生成单机的物理执行计划，如下图所示：

![image](http://static.lovedata.net/jpg/2018/5/21/379355edbd81503c0f525b698f70e543.jpg)

![image](http://static.lovedata.net/jpg/2018/5/21/293fc15dcc24eafafc9e577f9850a1a0.jpg)

## 4. Impala的部署方式

## 4.1 混合部署

混合部署意味着将Impala集群部署在Hadoop集群之上，共享整个Hadoop集群的资源，
前者的优势是Impala可以和Hadoop集群共享数据，不需要进行数据的拷贝，但是存在Impala和Hadoop集群抢占资源的情况，进而可能影响Impala的查询性能

![image](http://static.lovedata.net/jpg/2018/5/21/8651108dd21a447b7f781417cfb4a353.jpg)

## 4.2 独立部署

独立部署则是单独使用部分机器只部署HDFS和Impala
而后者可以提供稳定的高性能，但是需要持续的从Hadoop集群拷贝数据到Impala集群上，增加了ETL的复杂度

![image](http://static.lovedata.net/jpg/2018/5/21/a022911b475d7424a30cc3b68673820a.jpg)