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

## 2.HDFS

## 3.YARN

## 4.其他

### 4.1 简单概述hadoop中的角色的分配以及功能
