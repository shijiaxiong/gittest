### 脑图

### MySQL体系架构与存储引擎

### 数据类型

### 索引

### 锁机制

### 事务

### 查询

### 调优

### 分区分库分表

### 主从复制

## Q:MySQL 主从复制原理的是啥？

主库将变更写入 binlog 日志，然后从库连接到主库之后，从库有一个 IO 线程，将主库的 binlog 日志拷贝到自己本地，写入一个 relay 中继日志中。接着从库中有一个 SQL 线程会从中继日志读取 binlog，然后执行 binlog 日志中的内容，也就是在自己本地再次执行一遍 SQL，这样就可以保证自己跟主库的数据是一样的。

![](./picture/mysql-master-slave.png)



### 参考资料

- MySQL知识点与面试题目总结](https://juejin.im/post/6850037271233331208)
- [数据库面试题](https://mp.weixin.qq.com/s/lRCf17ymF1DU0ZIErclebA)
- 极客时间MySQL45讲
- 《MySQL技术内幕-InnoDB存储引擎》

### 未处理

#### Mysql-内存

mysql内存溢出 https://developer.aliyun.com/article/59256mysql

存在大字段(text blob)在建立内存表时是否会直接建立磁盘内存表mysql内部临时表建立时机及优化

相关文章

https://developer.aliyun.com/article/59256http://mysql.taobao.org/monthly/2016/06/07/https://time.geekbang.org/column/article/80477

#### 服务发现

各个框架比较zookeeper、etcd、Eureka(bilibili-discovery参考实现)

## Q:Mysql性能如何

> MySQL与版本、机器配置、架构有比较大的关系。

主从版本

SysBench参数：--table_size=25000 --tables=250。



[性能测试](https://help.aliyun.com/document_detail/109378.html?spm=a2c4g.11186623.6.1701.1cb01decGUUbLB)

