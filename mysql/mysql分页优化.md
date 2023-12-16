## MYSQL索引详解
索引的定义（索引别称index，key，键）

在关系数据库中，索引是对表中一列或多列的值进行排序的一种存储结构，它是表中一列或多列的值的集合，而且其中包含了对应表中记录的引用指针。索引的作用相当于图书的目录，可以根据目录中的页码快速找到所需的内容。
要注意的是，索引也是表的组成部分，建立太多的索引将会影响更新和插入的速度，因为它需要同样更新每个索引文件。对于一个经常需要更新和插入的表格，就没有必要为一个很少使用的where字句单独建立索引了，对于比较小的表，排序的开销不会很大，也没有必要建立索引。

举个例子：首先，先假设有一张表，表有10W个记录，其中有一条记录我们已知a='1'，如果想要拿到对应记录的话，需要的sql语句是 SELECT * FROM xxx WHERE a='1'.

一般情况下，对于查询语句，在没有建立索引的时候，mysql会进行全表扫描，而且不扫描完10W个记录不会停止，如果我在a上建立索引，那么mysql相当于只扫描a这一列即可，而且因为这一列已排好序，找到对应结果或结果集可以直接返回。

mysql的索引分为单列索引(全文索引，主键索引，唯一索引，普通索引)和组合索引。
- 单列索引:一个索引只包含一个列，一个表可以有多个单列索引
- 组合索引:一个组合索引包含两个或两个以上的列

### (一)索引的创建

#### 1.单列索引
##### 1-1)    普通索引（这个是最基本的索引）

建表时：INDEX IndexName(`字段名`(length))

建表后：CREATE INDEX IndexName ON `TableName`(`字段名`(length))

或ALTER TABLE TableName ADD INDEX IndexName(`字段名`(length)

注意：如果字段数据是CHAR，VARCHAR类型，可以指定length，其值小于字段的实际长度，如果是BLOB和TEXT类型就必须指定length。

这个length的用处是什么?

有时候需要在长文本字段上建立索引，但这种索引会增加索引的存储空间以及降低索引的效率，这时就可以用到length，创建索引时用到length的索引，我们叫做前缀索引，前缀索引是选择字段数据的前n个字符作为索引，这样可以大大节约索引空间，从而提高索引效率。

此处展示的语句用于创建一个索引，索引使用字段数据的前10个字符。
CREATE INDEX part_of_name ON customer (name(10));

使用字段数据的一部分创建索引可以使索引文件大大减小，从而节省了大量的磁盘空间，有可能提高INSERT操作的速度。

前缀索引是一种能使索引更小，更快的有效办法，但是MySql无法使用前缀索引做ORDER BY 和 GROUP BY以及使用前缀索引做覆盖扫描。

这里又引出了一个新概念，覆盖扫描！

如果一个索引（如：组合索引）中包含所有要查询的字段的值，那么就称之为覆盖索引，如：

SELECT user_name, city, age FROM user_test WHERE user_name = 'feinik' AND age > 25;
因为要查询的字段（user_name, city, age）都包含在组合索引的索引列中，所以就使用了覆盖索引查询，查看是否使用了覆盖索引可以通过执行计划中的Extra中的值为Using index则证明使用了覆盖索引，覆盖索引可以极大的提高访问性能。


##### 1-2)    唯一索引，要求字段所有的值是唯一的，这一点和主键索引一样，但是允许有空值。


建表时：UNIQUE INDEX IndexName(`字段名`(length))

建表后：CREATE UNIQUE  INDEX IndexName ON `TableName`(`字段名`(length))

或ALTER TABLE TableName ADD UNIQUE  INDEX IndexName(`字段名`(length)）


##### 1-3)    主键索引，不允许有空值
一般在建表的时候自动创建，主键一般会设为 int 而且是 AUTO_INCREMENT自增类型的



##### 1-4)全文索引
假设字段的数据类型是长文本，文本字段上(text等)建立了普通索引，我们需要查找关键字的话，那么其条件只能是where column like '%xxxx%' ，但是，这样做就会让索引失效，这时就需要全文索引了。


建表时：FULLTEXT INDEX IndexName(`字段名`(length))

建表后：CREATE FULLTEXT  INDEX IndexName ON `TableName`(`字段名`(length))

或ALTER TABLE TableName ADD FULLTEXT  INDEX IndexName(`字段名`(length)）

使用：
SELECT * FROM TableName
WHERE MATCH(column1， column2) AGAINST(‘xxx′， ‘sss′， ‘ddd′)
这条命令将把column1和column2字段里有xxx、sss和ddd的数据记录全部查询出来。

下面我们来举个例子：

假设有一个书籍表，结构如下，文章内容字段的数据类型是text
![](https://oss.wyxxt.org.cn/images/2021/09/18/ef09a7704eea78392a59147a6944ff39.png)

我想在茫茫多书籍的内容里搜索关键词，如果用%xxx%搜索，那效率就太低了。

我们在文章内容字段上建立全文索引，下面是索引文件
![](https://oss.wyxxt.org.cn/images/2021/09/18/af2006a73ac10ad26374bf7361440815.png)

那么当我想搜索  “塞亚人”的时候，这个索引文件直接告诉我在文章id为1和4的文章里有这个词。

可是这些关键词是如何提取出来的呢？这就是要提到一个新概念，“分词”！分词就是提取关键词，但是MYSQL的FULLTEXT对分词不够智能，对中文也不是很支持，所以我们一般不用全文索引。取而代之的是：

coreseek=sphinx+mmesg 这个程序就可以解决这个问题的啦。

sphinx就是索引程序。

mmseg就是分词程序。

国内有人修改了sphinx源码，内建和mmseg配合，整合到一起就是coreseek啦（中文版sphinx）！



2.组合索引

假设字段a，b都有索引，我们的查询条件是a=1，b=2查询过程是mysql会先挑选出符合a=1的结果集，再在这些结果集中挑选b=2的结果集，但是mysql并不会在查询a，b时都用到索引，只会用其中一个，这和我们的预期不一样，所以，我们要使用组合索引

建表时：INDEX IndexName(`字段名`(length)，`字段名`(length)，........) 

建表后：CREATE INDEX IndexName ON `TableName`(`字段名`(length)，`字段名`(length)，........) 

或ALTER TABLE TableName ADD INDEX IndexName(`字段名`(length)，`字段名`(length)，........) 


### (二)索引的删除
```sql
DORP INDEX IndexName ON `TableName`
```

### （三）索引失效的情况
1. 如果条件中有or，即使其中有条件带索引也不会使用(这也是为什么尽量少用or的原因)要想使用or，又想让索引生效，只能将or条件中的每个列都加上索引

2. 使用查询的时候遵循mysql组合索引的&quot;最左前缀&quot;规则，假设现在有组合索引（a，b，c），查询语句就只能是a=1或a=1and b=1或a=1 and b=1 and c=1。这里有两点需要注意①a=1 and b=1和b=1 and a=1一样，没有区别，都会使用索引②组合索引（a，b，c）的最左前缀是a；组合索引（c，b，a）的最左前缀是c，最左前缀和表字段顺序无关在组合索引中，如果where查询条件中某个列使用了范围查询（不管%在哪），则其右边的所有列都无法使用索引优化查询

3. like查询以%开头

4. 如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引

5. 如果mysql估计使用全表扫描要比使用索引快,则不使用索引

6. 索引列不能是表达式的一部分，也不能作为函数的参数，否则无法使用索引查询。下面是例子：
```sql
SELECT * FROM user_test WHERE user_name = concat(user_name, 'fei');
```

## MYSQL explain详解
explain显示了mysql如何使用索引来处理select语句以及连接表，可以帮助选择更好的索引和写出更优化的查询语句。

先解析一条sql语句：
```sql
EXPLAIN SELECT s.uid,s.username,s.name,f.email,f.mobile,f.phone,f.postalcode,f.address
FROM uchome_space AS s,uchome_spacefield AS f
WHERE 1
AND s.groupid=0
AND s.uid=f.uid
```
![](https://oss.wyxxt.org.cn/images/2021/09/18/4b583661e2d3912709c4d0569b6e6cd1.png)
### 1、id
select 识别标识符。这是select 查询序列号，这个不重要，查询序列号即为sql语句执行的顺序，继续看下边这条sql语句
```sql
explain select * from（Select * from uchome_space limit 10）as s
```
他的执行结果：
![](https://oss.wyxxt.org.cn/images/2021/09/18/e749c505359c544c92d2b0a4cbf5c23c.png)

### 2、select_type
select 类型，他有以下几种值
#### 2.1 simple 他表示简单的select ，没有union和子查询

#### 2.2 primary 最外面的select ，在有子查询的语句中，最外面的select 查询就是primary

#### 2.3 union union语句的第二个或者说是后面那一个，现执行一条语句，explain select * from uchome_space limit 10 union select *from uchome_space limit 10,10结果为：
![](https://oss.wyxxt.org.cn/images/2021/09/18/24aa65e7924383da9acbdf1f41522a7b.png)
#### 2.4dependent union union 中的第二个或者后面的select语句，取决于外面的查询

#### 2.5union result union 的结果如上所示

### 3、table
输出的行所用的表，这个参数显而易见，容易理解

### 4、type
连接类型，有多个参数，先从最佳类型到最差类型介绍

#### 4.1system
表示仅有一行，这是const类型的特例，平时不会出现，和这个也可以忽略不计

#### 4.2const
表最多有一行匹配，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快，记住一定是用到primary key或者unique，并且只检索出两条数据的情况下才会是const，看下面这条语句
```sql
explain select * from 'asj_admin_log' limit 1
```
结果是：
![](https://oss.wyxxt.org.cn/images/2021/09/18/069e8847374a327bfb07c94b08210b6f.png)
虽然只搜索一条数据，但是因为没有用到指定的索引，所以不会使用const，继续看看下面这个
```sql
explain select * from 'asj_admin_log' where log_id=111
```
![](https://oss.wyxxt.org.cn/images/2021/09/18/02bb9af28fe1c6e9e85112bbc93e0251.png)
log_id是主键，所以使用了const，所以说可以理解为copnst是最优化的；

#### 4.3eq_ref

对于er_ref的解释，mysql手册是这样说的：&quot;对于每个来自前面的表的行组合，从该表中读取一行。这可能是最好的连接类型，除了const类型。他用在一个索引的所有部分被联接使用并且索引是unique或者primary key&quot;. er_ref可以使用=比较呆索引的列，看下面语句
```sql
explain select *from uchome_spacefield ,uchome_space 
where uchome_spacefield.uid=uchome_space.uid
```
得到结果：
![](https://oss.wyxxt.org.cn/images/2021/09/18/94e886ed235e24b9a89638f9bf0e9b83.png)
#### 4.4ref

对于每个来自于前面的表的行组合，所有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是unique或primary key ，则使用ref。如果使用的键仅仅匹配少量行，该联接类型是不错的。
```sql
EXPLAIN select * from uchome_space where uchome_space.friendnum=0
```
得到结果如下，这条语句能搜出1w条数据。
![](https://oss.wyxxt.org.cn/images/2021/09/18/f0e9d491d89b9541c82d3c7c98c1cc4e.png)

#### 4.5ref_or_null

该联接类型如同ref，但是添加了mysql可以专门搜索包含NULL值得行。在解决子查询中经常使用该联接类型的优化。

#### 4.6index_merge

该联接类型表示使用了索引合并优化方法，在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素。

#### 4.7unique_subquery

#### 4.8index_subquery

#### 4.9range

给定范围内的检索，使用一个索引来检查行。看下面两条语句
```sql
explain select * from uchome_space where uid in(1,2)

explain select * from uchome_space where groupid in(1,2)
```
uid有索引，groupid没有索引，结果是第一条语句的联接类型是range，第二个是ALL，以为是一定范围，所以像between也可以这样联接，很明显

explain select *from uchome_space where friendnum=17

这样的语句是不会使用range的，它会使用更好的联接类型就是上面介绍的ref。

#### 4.10index

该联接类型与ALL相同，除了只有索引树被扫描，这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和index都是读全表，但是index是从索引中读取的，而all是从硬盘中读取的）

当查询只使用作为单索引部分的列时，Mysql可以使用该联接类型。

#### 4.11ALL

对于每个来自于优先前的表的行组合，进程完整的表扫描。如果标识第一个没标记const的表，这通常不好，并且通常在它情况下很差，通常可以增加更多的索引而不要使用ALL，使得行能基于前面的表中的常数值或值被检索出。

### 5、possible_keys 提示使用哪个索引会在该列表中找到行，不太重要

### 6、keys mysql 使用的索引，简单且重要

### 7、key_len mysql使用的索引长度

### 8、ref ref列显示使用哪列或常数与key一起从表中选择行。

### 9、rows 显示mysql 执行查询的行数，简单且重要，数值越大越不好，说明没有用好索引。

### 10、extra 该列包含mysql解决查询的详细信息

#### 10.1distinct mysql发现第一个匹配行后，停止为当前的行组合搜索更多的行

#### 10.2Not exists

#### 10.3range checked each record没有找到适合的索引。

#### 10.4using filesort

mysql手册是这么解释”mysql 需要额外的一次传递，以找出如何安排顺序检索行，通过根绝联接类型浏览所有行并为所有匹配where子句的行保存排序关键字和行的指针来完成排序。然后关键字被排序，并按排序顺序检索行“

#### 10.5using index

只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的信息。

explain select * from uspace_uchome where uid=1的extra为using index(uid建有索引)

explain select * from uspace_uchome where uid=1的extra为using index(groupid 未建立索引)

#### 10.6using temporary

为了解决查询，mysql需要创建一个临时表来容纳结果，典型情况如查询包含客户以按不同情况列出列的group by和order by子句时.出现using temporary就说明语句需要优化。

#### 10.7using where

where 子句用于限制哪一个行匹配下一个表或发送到客户。除非你专门从表中索取或检查所有行，如果extra值不为using where 并且表示联接类型为ALL或index，查询可能会有一些错误。

#### 10.8using sort_union(), using union（...）,using intersect(....)

这些函数说明如何为index_merge联接类型合并索引扫描

#### 10.9using index fro group-by

类似于访问表的using index方式，using index for group-by 表示mysql发现了一个索引，可以用查询group by或distinct查询的所有列，而不要额外搜索硬盘访问实际的表。并且，按最有效的方式使用索引，以便于对每个组，只读取少量索引条目。

## mysql千万级数据分页查询性能优化

mysql数据量大时使用limit分页，随着页码的增大，查询效率越低下。

实验

### 1.直接使用用limit start, count分页语句：
```sql
select * from order limit start, count
```
当起始页较小时，查询没有性能问题，我们分别看下从10， 100， 1000， 10000开始分页的执行时间（每页取20条）， 如下：
```sql
select * from order limit 10, 20 #0.016秒
 
select * from order limit 100, 20 #0.016秒
 
select * from order limit 1000, 20 #0.047秒
 
select * from order limit 10000, 20 #0.094秒
```
我们已经看出随着起始记录的增加，时间也随着增大， 这说明分页语句limit跟起始页码是有很大关系的，那么我们把起始记录改为40w看下
```sql
select * from order limit 400000, 20 #3.229秒
```
再看我们取最后一页记录的时间
```sql
select * from order limit 800000, #20 37.44秒
```
显然这种时间是无法忍受的。

从中我们也能总结出两件事情：
- 1）limit语句的查询时间与起始记录的位置成正比
- 2）mysql的limit语句是很方便，但是对记录很多的表并不适合直接使用。

### 2.对limit分页问题的性能优化方法
**利用表的覆盖索引来加速分页查询**

我们都知道，利用了索引查询的语句中如果只包含了那个索引列（覆盖索引），那么这种情况会查询很快。

因为利用索引查找有优化算法，且数据就在查询索引上面，不用再去找相关的数据地址了，这样节省了很多时间。另外Mysql中也有相关的索引缓存，在并发高的时候利用缓存就效果更好了。

在我们的例子中，我们知道id字段是主键，自然就包含了默认的主键索引。现在让我们看看利用覆盖索引的查询效果如何：

这次我们之间查询最后一页的数据（利用覆盖索引，只包含id列），如下：
```sql
select id from order limit 800000, 20 0.2秒
```
相对于查询了所有列的37.44秒，提升了大概100多倍的速度

那么如果我们也要查询所有列，有两种方法，一种是id&gt;=的形式，另一种就是利用join，看下实际情况：
```sql
SELECT * FROM order WHERE ID > =(select id from order limit 800000, 1) limit 20
```
查询时间为0.2秒，简直是一个质的飞跃啊，哈哈

另一种写法
```sql
SELECT * FROM order a JOIN (select id from order limit 800000, 20) b ON a.ID = b.id
```