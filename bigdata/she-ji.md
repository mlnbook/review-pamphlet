# 设计

## 1.海量URL去重

1. 通过BloomFilter去重，有一定的错误率
2. 通过HDFS+MapReduce去重
    - 使用一个reduce，map输入后将value输出，reduce得到相同的key后，value不管，则就可以去重了
    - [Hadoop数据去重详解](https://blog.csdn.net/lzq123_1/article/details/40895705)
    
## 2.海量数据排序

1. 原理同上，使用一个reducer，map输入后将value输出，利用mr的自动排序，然后reducer中用一个全局的linenumer变量 进行排号，这个号自增的，根据value list ，有多少个value，就输出多少次,用于处理相同值  
 
## 3. 10亿个数中找出最大的10000个数（top K问题）

1. top K问题很适合采用MapReduce框架解决，用户只需编写一个Map函数和两个Reduce 函数，然后提交到Hadoop（采用Mapchain和Reducechain）上即可解决该问题。具体而言，
2. 就是首先根据数据值或者把数据hash(MD5)后的值按照范围划分到不同的机器上，最好可以让数据划分后一次读入内存，这样不同的机器负责处理不同的数值范围，实际上就是Map。
3. 得到结果后，各个机器只需拿出各自出现次数最多的前N个数据，然后汇总，选出所有的数据中出现次数最多的前N个数据，这实际上就是Reduce过程。
4. 对于Map函数，采用Hash算法，将Hash值相同的数据交给同一个Reduce task；
5. 对于第一个Reduce函数，采用HashMap统计出每个词出现的频率，对于第二个Reduce 函数，统计所有Reduce task，输出数据中的top K即可。  
参考
[海量数据处理 - 10亿个数中找出最大的10000个数（top K问题）](https://blog.csdn.net/zyq522376829/article/details/47686867)




   
 
 