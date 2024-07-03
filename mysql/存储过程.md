## 使用背景

由于为了适应业务的灵活性，以前一直没有使用存储过程。最近接触了一个成熟的业务系统，需要做数据仓，并生成可视化报表。

## 优点

快：存储过程因为SQL语句已经预编译过了，因此运行的速度比较快
抽象封装：存储过程可以在单个存储过程中执行一系列SQL语句，使用时只需要存储过程的名字即可以重复使用。不需要再写一系列复杂的sql

## 创建存储过程

```sql
create procedure proc1(out s int)
begin
select count(*) from ob_article;
end;
```

## 调用存储过程

```sql
call proc1;
```

## 查看存储过程状态

```sql
show procedure status like 'proc1';
```

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_a7d37c4a6427c129a330a4117642efe0.jpg)

## 查看存储过程创建语句

```sql
show create procedure proc1;
```

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_2ff1dd3d054de6e8c24688f7ea0d3ece.jpg)

## 删除存储过程

```sql
drop procedure if exists proc1;
```