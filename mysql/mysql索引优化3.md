### 索引的维护与优化
- 删除重复和冗余的索引
primary key(id) 主键索引,unique key(id) 唯一索引,index(id) 单列索引，index(a,b) 联合索引
index(a),index(a,b) 冗余
primary key(id), index(a), index(a,id) 冗余

percona公司提供的工具：
pt-duplicate-key-checker h=127.0.0.1

![](https://oss.wyxxt.org.cn/images/2021/09/18/ce5f440697d39c617d058d2f88e1da83.png)

- 查找未被使用过的索引

```sql
select object_schema, object_name, index_name, b.`TABLE_ROWS`
from performance_schema.table_io_waits_summary_by_index_usage a
join information_schema.tables b on
	a.`OBJECT_SCHEMA`=b.`TABLE_SCHEMA` and
	a.`OBJECT_NAME`=b.`TABLE_NAME`
where index_name is not null
and count_star=0
order by object_schema, object_name;
```
![](https://oss.wyxxt.org.cn/images/2021/09/18/4a5b7bbcb2d182e29d15e22d4282ea37.png)

- 更新索引统计信息及减少索引碎片

```sql
analyze table table_name

optimize table table_name #会锁表
```