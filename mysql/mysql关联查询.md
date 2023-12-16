相信很多人在刚开始使用数据库的INNER JOIN、LEFT JOIN和RIGHT JOIN时，都不太能明确区分和正确使用这三种JOIN操作，本文通过一个简单的例子通俗易懂的讲解这三者的区别，希望对大家能带来帮助。

首先，我们创建示例数据库和表。同时也要明确一个概念：A INNER/LEFT/RIGHT JOIN B操作中，A表被称为左表，B表被称为右表。

创建示例数据库school，在数据库school下创建两张示例表：STUDENT、PUNISHMENT。

创建学生基本信息表STUDENT，如下:
![](https://oss.wyxxt.org.cn/images/2021/09/18/0d4ed38a8c134ad3bc1f017e2f01cd33.png)


创建学生违纪处罚记录表PUNISHMENT，如下：
![](https://oss.wyxxt.org.cn/images/2021/09/18/ef83889415028d070d7191e15657dd79.png)


注意，为了测试这三种JOIN操作的不同，PUNISHMENT表中（上图黄色标识）2014000009这个学生ID在学生基本信息表中是不存在的，这个相当于异常数据。

示例信息已经创建完毕，那么我们来看看具体的操作有什么区别。

### INNER JOIN操作
首先，我们看看INNER JOIN操作，我们写个SQL语句，查询学生表中哪些学生受过处分：
![](https://oss.wyxxt.org.cn/images/2021/09/18/23cad0c0d27127e4c06117034b1aa46c.png)
分析一下上面SQL语句的执行结果，我们的查询条件是“STU.STUDENT_ID=P.STUDENT_ID”，即学生表和处分表都有的STUDENT_ID的结果集，很明显，2014000002、2014000006在两表中都有，所以我们可以得出INNER JOIN操作的作用是：
**INNER JOIN：根据ON字段标识出来的条件，查出关联的几张表中，符合该条件的记录，合并成一个查询结果集。**
### LEFT JOIN操作
我们写个分析LEFT JOIN操作的SQL：
![](https://oss.wyxxt.org.cn/images/2021/09/18/d8131783c2ec1763d3a28e5786e9584d.png)
分析一下执行结果，LEFT JOIN操作中，比如A LEFT JOIN B，会输出左表A中所有的数据，同时将符合ON条件的右表B中搜索出来的结果合并到左表A表中，如果A表中存在而在B表中不存在，则结果集中会将查询的B表字段值（如此处的P.PUNISHMENT字段）设置为NULL。所以，LEFT JOIN的作用是：
**LEFT JOIN：从右表B中将符合ON条件的结果查询出来，合并到A表中，再作为一个结果集输出。**
### RIGHT JOIN操作
分析过LEFT JOIN了，RIGHT JOIN相信你也已经明白了，“A LEFT JOIN B ON ……”是将符合ON条件的B表搜索结果合并到A表中，作为一个结果集输出。而RIGHT JOIN刚好相反，“A RIGHT JOIN B ON ……”是将符合ON条件的A表搜索结果合并到B表中，作为一个结果集输出：
![](https://oss.wyxxt.org.cn/images/2021/09/18/925d2c892796d5467108f229e437afa2.png)

*总结一下：*
A INNER JOIN B ON……：内联操作，将符合ON条件的A表和B表结果均搜索出来，然后合并为一个结果集。
A LEFT JOIN B ON……：左联操作，左联顾名思义是，将符合ON条件的B表结果搜索出来，
然后左联到A表上，然后将合并后的A表输出。
A RIGHT JOIN B ON……：右联操作，右联顾名思义是，将符合ON条件的A表结果搜索出来，
然后右联到B表上，然后将合并后的B表输出。