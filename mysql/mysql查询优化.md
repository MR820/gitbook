> 所谓查询优化其实就是针对索引的优化所谓查询优化其实就是针对索引的优化

### 1. 模糊查询优化（like 模糊查询）
#### 1.1 业务允许，去掉前通配，改为后通配，可以走索引
```sql
#field已建立索引
select column from table where field like '%keyword%'
#优化  列左前缀查询 
select column from table where field like 'keyword%'

```
#### 1.2 通过函数达到模糊查询的效果
mysql内建函数无法使用索引，只有特殊情况下建立函数索引

keyword变化不大，如果是，考虑建立函数索引，否则对于全通配问题最好办法就是全文索引
##### 1.2.1 全文索引
```sql
CREATE FULLTEXT INDEX ft_name_t ON t(name)
#mysql只支持英文内容的全文索引
select * from t where match(name) against ('keyword')
```
LOCATE('substr',str,pos)
```sql
SELECT LOCATE('xbar',`foobar`);
###返回0

SELECT LOCATE('bar',`foobarbar`);
###返回4

SELECT LOCATE('bar',`foobarbar`,5);
###返回7
```
```sql
select column from table where locate('keyword', field)>0
```
POSITION('substr' IN `field`)
position可以看做是locate的别名，功能跟locate一样：
```sql
select column from table where position('keyword' in field)
```
INSTR(`str`,'substr')

```sql
create index idx_column_t on table(instr(field, 'keyword'))

SELECT `column` FROM `table` WHERE INSTR(`field`, 'keyword' )>0
```
FIND_IN_SET(str1,str2)
```sql
SELECT column FROM table WHERE FIND_IN_SET('keyword', field)
```
#### 1.3 只是前通配，可以使用reverse函数索引
```sql
select * from t where t.name like 'oradb1';
### 优化
create index idx_name_t on t(reverse(name));
select * from t where reverse(t.name) like '1bdaro%';

```

### 2. 不等于优化（!= &lt;&gt;）
```sql
select * from t where t.name <>'jack'
###优化
select * from t where t.name > 'jack' or t.name < 'jack'
```
### 3. not in 优化
```sql
select * from t where t.sort_id not in ('sortid001') limit 1;
###优化
select * from t left join (select sort_id from t where sort_id = 'sortid001') b on t.sort_id = b.sort_id 
where b.sort_id is null limit 1;
```
### 4. in 和 not in优化
```sql
select t.id,t.name from t where t.id not in (select u.tid from u where u.id=1)
###优化
select t.id,t.name from t left join u on u.tid = t.id where u.id=1
```
```sql
select t.id,t.name from t where t.id in (select u.tid from u)
###优化
select t.id,t.name from t left join u on u.id=t.id
```