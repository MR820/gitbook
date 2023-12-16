mysql在海量数据处理的时候效率会显著变慢。

mongodb快速！在适量级的内存的Mongodb的性能是非常迅速的，它将热数据存储在物理内存中，使得热数据的读写变得十分快。高扩展性，存储的数据格式是json格式！

![](https://oss.wyxxt.org.cn/images/2021/09/18/cb63beb90d59e49e6257994b962988f8.png)

![](https://oss.wyxxt.org.cn/images/2021/09/18/153a310f09f7b15fe0cd2d607ccfa3f4.png)

![](https://oss.wyxxt.org.cn/images/2021/09/18/baee8c0bc8441a4729e1666514cc15a5.png)

Mysql和Mongodb主要应用场景
如果需要将mongodb作为后端db来代替mysql使用，即这里mysql与mongodb 属于平行级别，那么，这样的使用可能有以下几种情况的考量：

- (1)mongodb所负责部分以文档形式存储，能够有较好的代码亲和性，json格式的直接写入方便。(如日志之类) 

- (2)从datamodels设计阶段就将原子性考虑于其中，无需事务之类的辅助。开发用如nodejs之类的语言来进行开发，对开发比较方便。

- (3)mongodb本身的failover(故障转移)机制，无需使用如MHA之类的方式实现。

- 