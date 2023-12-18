## 物理结构图

kafka:node01-node03
zookeeper:node01-node04(node04观察者，不参与选主)

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_f69f0c4635b99f0f6f4a82a00c26ff61.jpg)

## 配置

kafka的配置文件 server.properties

```vi
broker.id=0
listeners=PLANTEXT://node01:9092
log.dirs=/var/bigdata/kafka-logs
zookeeper.connect=node01:2181,node02:2181,node03:2181/kafka
```
/etc/profile 配置

## 节点同步

scp 个性化

## 结果图

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_2976a9d58782eefed4b88005ea945248.jpg)

## 消费者逻辑

是一条一条处理还是批次处理

解决方案是啥，
使用多线程
1，间接的聊大数据，比如spark
2，流式计算编程上去

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_1468a773882a456cba9d0bbf5f9a70e3.jpg)

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_1ce4522c255e48a24ba49d622432191a.jpg)


![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_1ec6ccd597b57f608345b1a57c08dc8f.jpg)


![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_961e0d1f89bb5e9ec1bfcd264d59055e.jpg)


1，单线程，按顺序，单条处理，offset就是递增的，无论对db，offset频率，成本有点高，CPU，网卡，资源浪费
进度控制，精度

2，流式的多线程，能多线程的多线程，但是，将整个批次的事务环节交给一个线程，做到这个批次，要么成功，要么失败，减少对DB的压力，和offset频率压力，更多的去利用CPU，网卡硬件资源
粒度比较粗了

通过redis获取数据，完善每一条记录的内容

## 代码实例

[java_kafka](https://github.com/MR820/java_kafka/tree/main "java_kafka")