- 索引列上不能使用表达式或函数
- 前缀索引和索引列的选择
  create index index_name on table(col_name(n));
  - 索引的选择性是不重复的索引值和表的记录数的比值
  ![](https://oss.wyxxt.org.cn/images/2021/09/18/54a39a5275efcf35bce861e187f77d4b.png)
- 联合索引
  如何选择索引列的顺序
  - 经常会被使用到的列优先
  - 选择性高的列优先
  - 宽度小的列优先
- 覆盖索引
  优点
  - 可以优化缓存，减少磁盘IO操作
  - 可以减少随机IO，变随机IO操作变为顺序IO操作
  - 可以避免对Innodb主键索引的二次查询
  - 可以避免MyISAM表进行系统调用

  无法使用覆盖索引的情况
  - 存储引擎可能不支持覆盖索引
  - 查询中使用了太多的列
  - 使用了双%号的like查询

![](https://oss.wyxxt.org.cn/images/2021/09/18/82012805d598cb53e819a24fc83f2a6c.png)

![](https://oss.wyxxt.org.cn/images/2021/09/18/6d642421f1c64b3692bac2d831a3620d.png)

![](https://oss.wyxxt.org.cn/images/2021/09/18/f3acf30cc250f96a956ab1aea4f72c53.png)

![](https://oss.wyxxt.org.cn/images/2021/09/18/654f88e553e114093486b47844fa3d93.png)
**Innodb二级索引自动加入主键信息**