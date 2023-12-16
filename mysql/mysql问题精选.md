![](https://oss.wyxxt.org.cn/images/2021/09/18/232a374e15807b6333437d28a94da497.png)

### 1. 数据库三范式是什么

- 列的原子性
- 每个事例必须可以被唯一地区分
- 不要冗余


### 2. ACID是什么

事务的四个特性：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）

原子性：不可分割，要么全部执行，要么全部不执行

一致性：数据一致性（强一致和弱一致）
一致性是指数据处于一种语义上的有意义且正确的状态。一致性是对数据可见性的约束，保证在一个事务中的多次操作的数据中间状态对其他事务不可见的。因为这些中间状态，是一个过渡状态，与事务的开始状态和事务的结束状态是不一致的。
举个栗子，张三给李四转账100元。事务要做的是从张三账户上减掉100元，李四账户上加上100元。一致性的含义是其他事务要么看到张三还没有给李四转账的状态，要么张三已经成功转账给李四的状态，而对于张三少了100元，李四还没加上100元这个中间状态是不可见的。

> 1.原子性和一致性的的侧重点不同：原子性关注状态，要么全部成功，要么全部失败，不存在部分成功的状态。而一致性关注数据的可见性，中间状态的数据对外部不可见，只有最初状态和最终状态的数据对外可见。
> 2.在未提交读的隔离级别下，会造成脏读，这就是因为一个事务读到了另一个事务操作内部的数据。ACID中是的一致性描述的是一个最理想的事务应该怎样的，是一个强一致性状态，如果要做到这点，需要使用排它锁把事务排成一队，即Serializable的隔离级别，这样性能就大大降低了。现实是骨感的，所以使用隔离性的不同隔离级别来破坏一致性，来获取更好的性能。

隔离性：事务执行中（隔离级别）
[mysql锁详解](https://wyxxt.org.cn/archives/mysql锁详解.html "mysql锁详解")

持久性：事务完成后


### 3. 索引是怎么实现的？

[mysql-索引实现原理](https://www.cnblogs.com/songwenjie/p/9415016.html "mysql-索引实现原理")

### 4. 怎么验证索引是否满足需求

explain
[mysql——explain详解和limit分页优化](https://wyxxt.org.cn/archives/mysql-explain详解和limit分页优化.html "mysql——explain详解和limit分页优化")

线上环境通过监控查看接口

### 5. 常用引擎及区别

- MyISAM存储引擎

基于ISAM存储引擎，并对其进行扩展。它是在web、数据仓储和其他应用环境下最常用的存储引擎之一。MyISAM拥有较高的插入、查询速度，但不支持事务。

B+Tree作为索引结构，叶子结点的data域存放的是数据记录的地址。

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_fe341e554d2ba264ad111ec3ee6387a2.jpg)
在MyISAM中，主索引和辅助索引（Secondary key）在结构上没有任何区别，只是主索引要求key是唯一的，而辅助索引的key可以重复。如果我们在Col2上建立一个辅助索引，则此索引的结构如下图所示：

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_6bc2baadb93373217879c24ac9b03f67.jpg)


- InnoDB存储引擎

InnoDB是事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键，InnoDB是默认的MySQL引擎。

虽然InnoDB也使用**B+Tree**作为索引结构，但具体实现方式却与MyISAM截然不同。

第一个重大区别是InnoDB的数据文件本身就是索引文件。从 上文知道，MyISAM索引文件和数据文件是分离的，索引文件仅保存数据记录的地址。而在InnoDB中，表数据文件本身就是按B+Tree组织的一个索 引结构，这棵树的叶节点data域保存了完整的数据记录。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主索引。

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_9bd05e11acea99545c8d29f736aa7b8a.jpg)

第二个与MyISAM索引的不同是InnoDB的辅助索引data域存储相应记录主键的值而不是地址。换句话说，InnoDB的所有辅助索引都引用主键作为data域。例如，下图为定义在Col3上的一个辅助索引：

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_804a5df0079f1cd9a7ca31090ce20997.jpg)

### 6. 行锁与表锁

[mysql锁详解](https://wyxxt.org.cn/archives/mysql锁详解.html "mysql锁详解")

### 7. 乐观锁与悲观锁

乐观锁 （version）
悲观锁 （互斥）

### 8. MySQL问题排查的手段

- 使用 SHOW PROCESSLIST 命令查看当前所有连接信息
- 使用explain 命令查询sql语句执行计划
- 开启慢查询日志，查看慢查询的sql

### 9. 性能优化方案

- 为搜索字段创建索引
- 避免使用select *，列出需要查询的字段
- 垂直分割分表
- 选择正确的存储引擎
- 索引优化，覆盖索引...