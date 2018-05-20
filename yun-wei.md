# 运维

## 1. 线上CPU100%，如何定位和排查问题

- top -c 显示进程运行信息列表 键入P(大写p)，线程按照CPU使用率排序
- top -Hp 10765 显示一个进程的线程运行信息列表(线程肯定是归属于某一个进程的) 键入P(大写p)，线程按照CPU使用率排序
- 将线程PID转化为16进制 printf "%x\n" 10804
- 使用 jstack 工具 jstack 10765 | grep '0x2a34' -C5 --color
- [线上服务CPU100%问题快速定位实战](http://www.cnblogs.com/winner-0715/p/7521638.html)
- [线上服务 CPU 100%？一键定位 so easy！](https://my.oschina.net/leejun2005/blog/1524687)