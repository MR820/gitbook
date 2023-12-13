redis是一个开源的使用c编写、支持网络、可基于内存支持可持久化的日志型、key-value数据库，提供多语言的API接口，通常用来作为分布式系统的缓存服务。分布式缓存中间件，redis集群通过RDB文件和AOF文件来保持主从同步的。slave启动时连接到master，主动发送一条SYNC命令；然后master启动后台持久化进程，在后台进程执行完毕后，master将传送整个redis数据库rdb文件到slave，完成全量同步；slave服务器接收到数据库rdb文件后将其存盘并加载到内存中；此后，master继续将更新命令（增删改）以aof文件的形式有序传送给slave，slave执行aof文件里的命令，从而slave与master保持数据同步。其中，关于rdb和aof两种文件含义如下：

1、rdb持久化
在指定的时间间隔内生成数据集的时间点快照

2、aof持久化
记录执行写操作命令,新命令会被追加到文件的末尾，对AOF文件进行重写，使得AOF文件不至于很大

redis还可以同时使用aof持久化和rbd持久化。在这种情况下，当redis重启时，优先使用aof文件来还原数据集，因为aof文件通常比rdb文件所保存的数据集更完整。
![img](https://oss.wyxxt.org.cn/images/2021/09/18/3cfbde184bb4ba27e955584bb318ff4e.png)

slave连接master后，会主动发起PSYNC命令，slave会提供master_runid和offset，master验证master_runid和offset是否有效，其中master_runid作为master身份验证，offset是全局积压空间数据的偏移量。

#### 1、完整重同步

当slave的offset不在master暂存队列时，执行完整重同步。master返回 +FULLRESYNC master_runid offset，启动BGSAVE生成rdb文件，BGSAVE结束后，向slave传输，从而完成全同步

#### 2、部分重同步

当slave的offset存在于master暂存队列时，执行部分重同步。slave读取offset偏移之后的所有更新事务日志aof，然后slave执行对应事务