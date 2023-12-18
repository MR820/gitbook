## 创建topic

```shell
kafka-topics.sh --zookeeper node01:2181,node02:2181,node03:2181/kafka --create --topic ceshi-items --partitions 2 --replication-factor 3
```

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_f9444955d7e8b18b8df1521df690847b.jpg)

## 查看topic

```shell
kafka-topics.sh --zookeeper node01:2181,node02:2181,node03:2181/kafka --describe --topic ceshi-items
```

## 修改partition数量

```shell
kafka-topics.sh --zookeeper bigserver1:2181,bigserver2:2181,testing:2181 --alter --topic ceshi-items --partitions 3
```

## mmap


![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_844c862245085c1b7cf14144e4084993.jpg)


## 磁盘文件结构


![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_2a239ebd266eea9a23ffbb68ee3e8ff5.jpg)


## 跟踪打开文件

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_f09641f575036e69e9adae299f2a1513.jpg)

## 索引详解

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_67eb43da33ba593cb590ceee24374f8c.jpg)



![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_1aa461ae3d68202907673faacc00b46a.jpg)


## 自定义消费数据 本质 seek

### java代码

[java_kafka](https://github.com/MR820/java_kafka/tree/main "java_kafka")