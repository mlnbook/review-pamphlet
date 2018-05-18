# Redis的主键争用问题如何解决？

使用watch 他会监测键，确保被修改后，后面的修改会失败SETNX 如果key不存在等同于set返回0，存在返回1  
[https://blog.csdn.net/youxijishu/article/details/41956983](https://blog.csdn.net/youxijishu/article/details/41956983)

