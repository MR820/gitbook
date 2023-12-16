## InnoDB 锁

#### my.ini配置文件

```vi
transaction-isolation = {READ-UNCOMMITTED | READ-COMMITTED | REPEATABLE-READ | SERIALIZABLE}
```

#### 用户可以用SET TRANSACTION语句改变单个会话或者所有新进连接的隔离级别。它的语法如下：
```sql
SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}
```

#### 你可以用下列语句查询全局和会话事务隔离级别：
```sql
SELECT @@global.tx_isolation;
SELECT @@session.tx_isolation;
SELECT @@tx_isolation;
```



相对于串行处理来说，并发事务处理能大大增加数据库资源的利用率，提高数据库系统的事务吞吐量，从而可以支持更多用户的并发操作，但与此同时，会带来一下问题：

**脏读**： 一个事务正在对一条记录做修改，在这个事务并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”的数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做“脏读” 

**不可重复读**：一个事务在读取某些数据已经发生了改变、或某些记录已经被删除了！这种现象叫做“不可重复读”。 

**幻读**： 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读” 

上述出现的问题都是数据库读一致性的问题，可以通过事务的隔离机制来进行保证。

数据库的事务隔离越严格，并发副作用就越小，但付出的代价也就越大，因为事务隔离本质上就是使事务在一定程度上串行化，需要根据具体的业务需求来决定使用哪种隔离级别

|                  | 脏读 | 不可重复读 | 幻读 |
| :--------------: | :--: | :--------: | :--: |
| read uncommitted |  √   |     √      |  √   |
|  read committed  |      |     √      |  √   |
| repeatable read  |      |            |  √   |
|   serializable   |      |            |      |

 可以通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况： 


| 隔离级别                     | 脏读（Dirty Read） | 不可重复读（NonRepeatable Read） | 幻读（Phantom Read）  |
| ---------------------------- | ------------------ | -------------------------------- | --------------------- |
| 未提交读（Read uncommitted） | 可能               | 可能                             | 可能                  |
| 已提交读（Read committed）   | 不可能             | 可能                             | 可能                  |
| 可重复读（Repeatable read）  | 不可能             | 不可能                           | 可能  （mysql不可能） |
| 可串行化（Serializable ）    | 不可能             | 不可能                           | 不可能                |

数据库使用锁是为了支持更好的并发，提供数据的完整性和一致性。InnoDB是一个支持行锁的存储引擎，锁的类型有：共享锁（S）、排他锁（X）、意向共享（IS）、意向排他（IX）。为了提供更好的并发，**InnoDB提供了非锁定读：不需要等待访问行上的锁释放，读取行的一个快照。该方法是通过InnoDB的一个特性：MVCC来实现的。**

MySQL InnoDB存储引擎，实现的是基于多版本的并发控制协议——**MVCC** (Multi-Version Concurrency Control) (注：与MVCC相对的，是基于锁的并发控制，Lock-Based Concurrency Control)。MVCC最大的好处，相信也是耳熟能详：**读不加锁，读写不冲突。**在读多写少的OLTP(On-Line Transaction Processing)应用中，**读写不冲突是非常重要的，极大的增加了系统的并发性能**，这也是为什么现阶段，几乎所有的RDBMS，都支持了MVCC。

 在MVCC并发控制中，读操作可以分成两类：**快照读 (snapshot read)与当前读 (current read)。**快照读，读取的是记录的可见版本 (有可能是历史版本)，不用加锁。**当前读，读取的是记录的最新版本，并且，当前读返回的记录，都会加上锁，保证其他事务不会再并发修改这条记录。**

#### 在一个支持MVCC并发控制的系统中，哪些读操作是快照读？哪些操作又是当前读呢？以MySQL InnoDB为例：
[MVVC机制](https://juejin.im/post/5c68a4056fb9a049e063e0ab#heading-8 "MVVC机制")
- 快照读：简单的select操作，属于快照读，不加锁。(当然，也有例外，下面会分析)
```sql
select * from table where ?;
```


- 当前读：特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。
```sql
select * from table where ? lock in share mode;
select * from table where ? for update;
insert into table values (…);
update table set ? where ?;
delete from table where ?;
```

所有以上的语句，都属于当前读，读取记录的最新版本。并且，读取之后，还需要保证其他并发事务不能修改当前记录，对读取记录加锁。其中，除了第一条语句，对读取记录加S锁 (共享锁)外，其他的操作，都加的是X锁 (排它锁)。

- 共享锁【S锁】
又称读锁，若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
- 排他锁【X锁】
又称写锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。这保证了其他事务在T释放A上的锁之前不能再读取和修改A。

#### 为什么将 插入/更新/删除 操作，都归为当前读？可以看看下面这个 更新 操作，在数据库中的执行流程：

![](https://oss.wyxxt.org.cn/images/2021/09/18/73bafc13d6faa298b578ffcad660bd8c.png)

从图中，可以看到，一个Update操作的具体流程。当Update SQL被发给MySQL后，MySQL Server会根据where条件，读取第一条满足条件的记录，然后InnoDB引擎会将第一条记录返回，并加锁 (current read)。待MySQL Server收到这条加锁的记录之后，会再发起一个Update请求，更新这条记录。一条记录操作完成，再读取下一条记录，直至没有满足条件的记录为止。因此，Update操作内部，就包含了一个当前读。同理，Delete操作也一样。Insert操作会稍微有些不同，简单来说，就是Insert操作可能会触发Unique Key的冲突检查，也会进行一个当前读。

MySQL/InnoDB定义的4种隔离级别：
**Read Uncommited**
可以读取未提交记录。此隔离级别，不会使用，忽略。
**Read Committed (RC)**
快照读：忽略，本文不考虑。
当前读：**RC隔离级别保证对读取到的记录加锁 (记录锁)**，存在幻读现象。
**Repeatable Read (RR)**
快照读：忽略，本文不考虑。
当前读：RR隔离级别保证对读取到的记录加锁 (记录锁)，同时保证对读取的范围加锁，新的满足查询条件的记录不能够插入 (间隙锁)，不存在幻读现象。**注意：这里的不存在幻读，是指使用select ... for update 在同一个事务中查询，不会出现两次不一样的结果**
```sql
-----session1------
begin;
select * from t1 for update;
+------+------+
| c1   | c2   |
+------+------+
|    1 |    1 |
|    5 |    5 |
+------+------+
2 rows in set (0.01 sec)

------session2------
select * from t1 for update;
/**waiting*/

------session1------
commit;

------session2------

+------+------+
| c1   | c2   |
+------+------+
|    1 |    1 |
|    5 |    5 |
+------+------+
2 rows in set (22.44 sec)
```
**Serializable**
从MVCC并发控制退化为基于锁的并发控制。不区别快照读与当前读，所有的读操作均为当前读，读加读锁 (S锁)，写加写锁 (X锁)。
Serializable隔离级别下，读写冲突，因此并发度急剧下降，在MySQL/InnoDB下不建议使用。
上面说的**当前读**就是上面列出来的　select .. for update, update , delete, insert 等语句