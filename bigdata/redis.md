[TOC]

#1. Redis的主键争用问题如何解决？

使用watch 他会监测键，确保被修改后，后面的修改会失败SETNX 如果key不存在等同于set返回0，存在返回1
[Redis的乐观同步方法 Redis的并发写入同步](https://blog.csdn.net/youxijishu/article/details/41956983)

# 2. Redis的事物原理？

1. 满足一致性和隔离性，不满足原子性和持久性（依赖具体持久模型）

2. watch unwatch 和 muti exec
当两者一起使用的时候，首先key被watch监视，若在调用 EXEC 命令执行事务时， 如果任意一个被监视的键被其他客户端修改了， 那么整个事务不再执行， 直接返回失败（之后可以选择重试事物或者放弃）
因为exec之前不会有任何实际操作（通过queue队列），就是没有办法根据读取到的数据来做决定（可能读到的已经是脏数据）

3. watch的目的
目的：使用select for update 普通加锁，第一个执行的执行完成之前，其余的事物都必须阻塞，造成时间上的等待，所以watch并不会对数据进行加锁，redis只会在数据改变的情况下，通知执行了watch的客户端，叫做乐观锁optimistic locking 。
Redis的事务没有关系数据库事务提供的回滚（rollback）功能,需要自行回滚，但是一般不会有这种强一致的需求。否则是需求不合理

4. 报错了怎么处理？
 - 语法错误：只要有一个命令有语法错误，执行EXEC命令后Redis就会直接返回错误，连语法正确的命令也不会执行。
![image](http://static.lovedata.net/jpg/2018/5/18/e58f5d71439a34699548842b85c9d413.jpg)
  
  - 运行时错误： 运行错误指在命令执行时出现的错误，比如使用散列类型的命令操作集合类型的键，这种错误在实际执行之前Redis是无法发现的，所以在事务里这样的命令是会被Redis接受并执行的。如果事务里的一条命令出现了运行错误，事务里其他的命令依然会继续执行（包括出错命令之后的命令）
![image](http://static.lovedata.net/jpg/2018/5/18/6971ad099e1afbb9f65823c9749bc90b.jpg)
5. 参考
[Redis的并发控制](https://juejin.im/entry/5964bcd851882568b20dbd73)
[redis的事务和watch](https://www.jianshu.com/p/361cb9cd13d5) 