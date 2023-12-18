![](https://oss.wyxxt.org.cn/images/2021/09/18/5b43c029e0efda093ac68bcb3f9ef0c8.png)


kafka的write操作使用了顺序写入和mmap，read操作使用了零拷贝


[顺序写入、MMAP、DMA计算、零拷贝](https://wyxxt.org.cn/archives/顺序写入、MMAP、DMA计算、零拷贝.html "顺序写入、MMAP、DMA计算、零拷贝")



1. kafka设计架构



分治是所有大数据领域的核心。微服务其实也是遵循分治原则，保证所有的服务都是无状态的，可以任意扩展。



![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_eb2201348434fd43e483981d4d5f2f2e.jpg)





![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_996763b1980bf6842e5f8537fce7c092.jpg)





![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_1db710c04f0995c27ba03112fdbfe503.jpg)






2. 数据传输的事务定义有哪三种？


最多一次：消息不会被重复发送，最多传输一次，但也可能一次不传输


最少一次：消息不会被漏发送，最少被传输一次，但也有可能被重复传输


精确的一次：不会漏传输也不会重复传输，每个消息都被传输一次而且仅传输一次




consumer：
所有的副本都有相同的日志文件和相同的offset，consumer维护自己消费的消息的offset，如果consumer不会崩溃当然可以在内存中保存这个值，当然谁也不能保证这点。如果consumer崩溃了，会有另外一个consumer接着消费消息，它需要从一个合适的offset继续处理。这种情况下可以有以下选择：




- consumer可以先读取消息，然后将offset写入日志文件中，然后再处理消息。这存在一种可能就是在存储offset后还没处理消息就crash了，新的consumer继续从这个offset处理，那么就会有些消息永远不会被处理，这就是上面说的“最多一次”。





- consumer可以先读取消息，处理消息，最后记录offset，当然如果在记录offset之前就crash了，新的consumer会重复的消费一些消息，这就是上面说的“最少一次”。





- “精确一次”可以通过将提交分为两个阶段来解决：保存了offset后提交一次，消息处理成功之后再提交一次。但是还有个更简单的做法：将消息的offset和消息被处理后的结果保存在一起。比如用Hadoop ETL处理消息时，将处理后的结果和offset同时保存在HDFS中，这样就能保证消息和offset同时被处理了。




producer：
当发布消息时，Kafka有一个“committed”的概念，一旦消息被提交确认了，数据就不会丢失。

- block.on.buffer.full = true 尽管该参数在0.9.0.0已经被标记为“deprecated”，但鉴于它的含义非常直观，所以这里还是显式设置它为true，使得producer将一直等待缓冲区直至其变为可用。否则如果producer生产速度过快耗尽了缓冲区，producer将抛出异常

- acks=all 很好理解，所有follower都响应了才认为消息提交成功，即"committed"

- retries = MAX 无限重试，直到你意识到出现了问题

- max.in.flight.requests.per.connection = 1 限制客户端在单个连接上能够发送的未响应请求的个数。设置此值是1表示kafka broker在响应请求之前client不能再向同一个broker发送请求。注意：设置此参数是为了避免消息乱序

- 使用KafkaProducer.send(record, callback)而不是send(record)方法 自定义回调逻辑处理消息发送失败

- callback逻辑中最好显式关闭producer：close(0) 注意：设置此参数是为了避免消息乱序

- unclean.leader.election.enable=false 关闭unclean leader选举，即不允许非ISR中的副本被选举为leader，以避免数据丢失

- replication.factor >= 3 这个完全是个人建议了，参考了Hadoop及业界通用的三备份原则

- min.insync.replicas > 1 消息至少要被写入到这么多副本才算成功，也是提升数据持久性的一个参数。与acks配合使用

- 保证replication.factor > min.insync.replicas 如果两者相等，当一个副本挂掉了分区也就没法正常工作了。通常设置replication.factor = min.insync.replicas + 1即可





3. kafka判断一个节点还活着有那两个条件


节点必须可以维护和ZooKeeper的连接，Zookeeper通过心跳机制检查每个节点的连接


如果节点是个follower,他必须能及时的同步leader的写操作，延时不能太久



4. producer是否直接将数据发送到broker的leader节点



producer直接将数据发送到broker的leader(主节点)，不需要在多个节点进行分发，为了帮助producer做到这点，所有的Kafka节点都可以及时的告知:哪些节点是活动的，目标topic目标分区的leader在哪。这样producer就可以直接将消息发送到目的地了




5. consumer是否可以消费指定分区消息


Kafa consumer消费消息时，向broker发出"fetch"请求去消费特定分区的消息，consumer指定消息在日志中的偏移量（offset），就可以消费从这个位置开始的消息，customer拥有了offset的控制权，可以向后回滚去重新消费之前的消息，这是很有意义的







![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_9391f0d13ec47bafae68e51d32e99273.jpg)





![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_fc9d491a7bd1ee7ad8d3a543aa94bf7c.jpg)




6. kafka消费时采用Pull模式，还是Push模式



Kafka最初考虑的问题是，customer应该从brokes拉取消息还是brokers将消息推送到consumer，也就是pull还push。在这方面，Kafka遵循了一种大部分消息系统共同的传统的设计：producer将消息推送到broker，consumer从broker拉取消息


一些消息系统比如Scribe和Apache Flume采用了push模式，将消息推送到下游的consumer。这样做有好处也有坏处：由broker决定消息推送的速率，对于不同消费速率的consumer就不太好处理了。消息系统都致力于让consumer以最大的速率最快速的消费消息，但不幸的是，push模式下，当broker推送的速率远大于consumer消费的速率时，consumer恐怕就要崩溃了。最终Kafka还是选取了传统的pull模式


Pull模式的另外一个好处是consumer可以自主决定是否批量的从broker拉取数据。Push模式必须在不知道下游consumer消费能力和消费策略的情况下决定是立即推送每条消息还是缓存之后批量推送。如果为了避免consumer崩溃而采用较低的推送速率，将可能导致一次只推送较少的消息而造成浪费。Pull模式下，consumer就可以根据自己的消费能力去决定这些策略


Pull有个缺点是，如果broker没有可供消费的消息，将导致consumer不断在循环中轮询，直到新消息到t达。为了避免这点，Kafka有个参数可以让consumer阻塞直到新消息到达（当然也可以阻塞直到消息的数量达到某个特定的量这样就可以批量发）





7. kafka存储在硬盘上的消息格式是什么



消息由一个固定长度的头部和可变长度的字节数组组成。头部包含了一个版本号和CRC32校验码。

- 消息长度: 4 bytes (value: 1+4+n)
- 版本号: 1 byte
- CRC校验码: 4 bytes
- 具体的消息: n bytes


8. kafka高效文件存储设计特点


- Kafka把topic中一个parition大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除已经消费完的文件，减少磁盘占用。

- 通过索引信息可以快速定位message和确定response的最大大小。

- 通过index元数据全部映射到memory，可以避免segment file的IO磁盘操作。

- 通过索引文件稀疏存储，可以大幅降低index文件元数据占用空间大小。



9. kafka与传统的消息系统之间的三个关键区别

- Kafka持久化日志，这些日志可以被重复读取和无限期保存

- kafka是一个分布式系统：它以集群的方式运行，可以灵活伸缩，在内部通过复制数据提升容错能力和高可用性

- Kafka支持实时的流试处理


10.  kafka创建topic时如何将分区放置到不同的broker中


- 副本因子不能大于 Broker 的个数；

- 第一个分区（编号为0）的第一个副本放置位置是随机从 brokerList 选择的；

- 其他分区的第一个副本放置位置相对于第0个分区依次往后移。也就是如果我们有5个 Broker，5个分区，假设第一个分区放在第四个 Broker 上，那么第二个分区将会放在第五个 Broker 上；第三个分区将会放在第一个 Broker 上；第四个分区将会放在第二个 Broker 上，依次类推；

- 剩余的副本相对于第一个副本放置位置其实是由 nextReplicaShift 决定的，而这个数也是随机产生的


12. partition的数据如何保存到硬盘


topic中的多个partition以文件夹的形式保存到broker，每个分区序号从0递增，

且消息有序

Partition文件下有多个segment（xxx.index，xxx.log）

segment 文件里的 大小和配置文件大小一致可以根据要求修改 默认为1g

如果大小大于1g时，会滚动一个新的segment并且以上一个segment最后一条消息的偏移量命名

13. kafka的ack机制

request.required.acks有三个值 0 1 -1

0:生产者不会等待broker的ack，这个延迟最低但是存储的保证最弱当server挂掉的时候就会丢数据

1:服务端会等待ack值leader副本确认接收到消息后发送ack但是如果leader挂掉后他不缺包是否复制完成新leader也会导致数据丢失

-1:同样在1的基础上服务端会等所有的follower的副本收到数据后才会收到来自leader发出的ack，确保数据的强一致性


14. kafka的消费者如何消费数据

消费者记录物理偏移量offset的位置

15. 消费者负载均衡策略


一个partition对应一个消费者成员，消费组中每个消费者成员都能访问，如果组中成员太多会有空闲的成员



![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_fc9d491a7bd1ee7ad8d3a543aa94bf7c.jpg)




16. 数据有序

分区可以打破单机的容量
分区数越大，写入并发越大。提升写入性能

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_166b949251a89b1623a1d87f4f928e4a.jpg)


17. kafka生产数据时数据的分组策略

hash(key) % 分区数





[ThinkPHP 5.0 整合kafka2.4.0](https://wyxxt.org.cn/archives/thinkphp5-0%e6%95%b4%e5%90%88kafka2-4-0.html)