# 设计

## 1.海量URL去重

1. 通过BloomFilter去重，有一定的错误率
2. 通过HDFS+MapReduce去重
    - 使用一个reduce，map输入后将value输出，reduce得到相同的key后，value不管，则就可以去重了
    - [Hadoop数据去重详解](https://blog.csdn.net/lzq123_1/article/details/40895705)
    
## 2.海量数据排序

1. 原理同上，使用一个reducer，map输入后将value输出，利用mr的自动排序，然后reducer中用一个全局的linenumer 进行排号，这个号自增的。



