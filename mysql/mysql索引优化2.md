### 使用索引来优化查询
- 使用索引扫描来优化排序(B-tree)
  通过排序操作
  按照索引顺序扫描数据
  - 索引的列的顺序和order by子句的顺序完全一致
  - 索引中所有列的方向（升序，降序）和order by子句完全一致
  - order by中的字段全部在关联表中的第一张表中
  Innodb表结构
  ![](https://oss.wyxxt.org.cn/images/2021/09/18/44b541372913701e3db2f86bd1eabe6a.png)
  执行计划
  ![](https://oss.wyxxt.org.cn/images/2021/09/18/f2b9d2cce1dae858a89e544b777aad75.png)
  MyISAM表结构
  ![](https://oss.wyxxt.org.cn/images/2021/09/18/41871e8e97faa2f4fa4c77c28ca5cdca.png)
  执行计划
  ![](https://oss.wyxxt.org.cn/images/2021/09/18/2b69d9e23534e993189d7c7288eb95b7.png)
  Innodb二级索引排序
  ![](https://oss.wyxxt.org.cn/images/2021/09/18/11684b923dc2184d1c5d8f0ed2ca9969.png)
  MyISAM二级索引排序
  ![](https://oss.wyxxt.org.cn/images/2021/09/18/2dab7505327fc69bcdbed25f8c9b2a0a.png)
  索引中所有列的方向（升序，降序）和order by子句完全一致
  ![](https://oss.wyxxt.org.cn/images/2021/09/18/474b03bf98321e03b2d40d94e734c432.png)

### 模拟Hash索引优化查询
表结构
![](https://oss.wyxxt.org.cn/images/2021/09/18/db1a296f36e0849b882166d816fdbd08.png)
```sql
alert table file add title_md5 varchar(32);
update film set title_md5=md5(title);
create index idx_md5 on film(title_md5);
```
![](https://oss.wyxxt.org.cn/images/2021/09/18/9a5c015478550dc69152bc0e58736233.png)
- 只能处理键值的全值匹配查找
- 所使用的Hash函数决定着索引键的大小

### 利用索引优化锁
  - 索引可以减少锁定的行数
  - 索引可以加快处理速度，同时也加快了锁的释放