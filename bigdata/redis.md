# Redis的主键争用问题如何解决？

使用watch 他会监测键，确保被修改后，后面的修改会失败SETNX 如果key不存在等同于set返回0，存在返回1  
[https://blog.csdn.net/youxijishu/article/details/41956983](https://blog.csdn.net/youxijishu/article/details/41956983)



# Redis的事物原理

满足一致性和隔离性，不满足原子性和持久性（依赖具体持久模型）

watch unwatch 和 muti exec 

当两者一起使用的时候，首先key被watch监视，若在调用 EXEC 命令执行事务时， 如果任意一个被监视的键被其他客户端修改了， 那么整个事务不再执行， 直接返回失败（之后可以荀泽重试事物或者放弃）

因为exec之前不会有任何实际操作（通过queue队列），就是没有办法根据读取到的数据来做决定（可能读到的已经是脏数据）

目的：使用select for update 普通加锁，第一个执行的执行完成之前，其余的事物都必须阻塞，造成时间上的等待，所以watch并不会对数据进行加锁，redis只会在数据改变的情况下，通知执行了watch的客户端，叫做乐观锁optimistic locking

  


Redis的事务没有关系数据库事务提供的回滚（rollback）功能,需要自行回滚，但是一般不会有这种强一致的需求。否则是需求不合理

  


而只要有一个命令有语法错误，执行EXEC命令后Redis就会直接返回错误，连语法正确的命令也不会执行。



