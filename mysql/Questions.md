

## Q: MySQL主从复制原理

- MySQL主从复制涉及三个线程：主节点的线程：`log dump thread` ，从库生成两个线程：一个`I/O`线程，一个SQL 线程。

**主节点log dump 线程**

- 当从节点连接主节点时，主节点会为其创建一个 log dump 线程，用于发送和读取 Binlog 的内容。
- 在读取 Binlog 中的操作时，log dump 线程会对主节点上的 Binlog 加锁；当读取完成发送给从节点之前，锁会被释放。
- **主节点会为自己的每一个从节点创建一个 log dump 线程**。

**从节点IO线程**

- I/O 线程主要是请求主库中更新的Binlog。在接收到主节点的 log dump 进程发来的更新之后，保存在本地 relay-log（中继日志）中。

**从节点SQL线程**

- SQL 线程负责读取 relay log 中的内容，解析成具体的操作并执行，最终保证主从数据的一致性。

#### 复制模式

- **异步模式：** 主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理。
  - 缺点：主节点如果崩溃掉了，此时主节点上已经提交的事务可能并没有传到从节点上。如果强行将从提升为主，可能导致新主节点上的数据不完整。
- **半同步复制：** 主库在执行完客户端提交的事务后，等待至少一个从库接收到并写到 relay log 中才返回成功信息给客户端（只能保证主库的 Binlog 至少传输到了一个从节点上），否则需要等待直到超时时间然后切换成异步模式再提交。
- **全同步模式：** 主库执行完一个事务，然后所有的从库都复制了该事务并成功执行完才返回成功信息给客户端。




## Q: Mysql高可用架构

### 1、主从或主主半同步复制

- 优点：双节点没有主机宕机后的选主问题，资源要求少，部署简单。
- 缺点：
  - 完全依赖半同步复制。如果半同步复制退化为异步复制，数据一致性无法得到保证。
  - 需要额外考虑haproxy、keepalived的高可用机制。



### 2、MHA 

-  MHA Manager会定时探测集群中的master节点，当master出现故障时，它可以自动将最新数据的slave提升为新的master，然后将所有其他的slave重新指向新的master。

#### 优点：

- 可以进行故障的自动检测和转移;
- 可扩展性较好，可以根据需要扩展MySQL的节点数量和结构。
- 相比于双节点的MySQL复制，三节点/多节点的MySQL发生不可用的概率更低。

#### 缺点：

- 可能会因为网络产生脑裂问题。解决方案：可以加入同机架其他物理机的探测，通过对比更多信息来判断是网络故障还是单机故障。
- manager是单节点。

### MMM

优点：

缺点：

### MGR



## Q:Mysql和Redis的区别？

- 类型：
  - Mysql：关系型数据库。
  - Redis：非关系型数据库。
- 性能:
  - Mysql：读取速度慢，多线程、磁盘IO。
  - Redis：读取速度快，基于内存、单线程。
- 内存空间和数据量：
  - Mysql基于磁盘
  - Redis基于内存
- 可靠性上：MySQL可靠性高。如果Redis持久化AOF每一秒刷新一次，会丢失数据。Redis主从复制主机宕机也可能会丢失数据。
- 事务支持：
  - MySQL事务满足ACID。
  - Redis事务只能保证一次性、顺序性、排他性执行。并不能保证原子性。

