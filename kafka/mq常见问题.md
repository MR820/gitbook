![](https://oss.wyxxt.org.cn/images/2021/09/18/86f26c15541e807411399ccf515cd9eb.png)

### 一、mq如何保证消息不丢失
**以rabbitmq php为例**

1. 生产者必须确认我们的消息一定投递到mq中
>如何确认消息一定投递到mq成功?
>confirm机制 消息确认机制
>生产者投递消息被mq拒绝，放入私信队列或者增加消费者
>建议最好将每次向mq投递消息采用日志的形式记录下来，保证后期查数据
```php
//一旦消息被设为confirm模式，就不能设置事务模式，反之亦然
$channel->confirm_select();
//阻塞等待消息确认
$channel->wait_for_pending_acks();
//异步回调消息确认
$channel->set_ack_handle();
$channel->set_nack_handler();
PHP使用 Rabbitmq
```
![](https://oss.wyxxt.org.cn/images/2021/09/18/73b8ff559a7513534fc3b0c695648178.png)

2. mq服务器将消息持久化到硬盘
>- mq宕机的情况下，消息是否会丢失?
>不丢失，持久化到硬盘
>![](https://oss.wyxxt.org.cn/images/2021/09/18/975f22ffe769915f8b9d7dc73c67f97e.png)
>在声明交换机和队列时，durable（耐久）设置为true
```php
//交换机持久化
$channel->exchange_declare('exchange', 'fanout', false, true, false);
//队列持久化
$channel->queue_declare('task_queue', false, true, false, false);
//消息持久化
$msg = new AMQPMessage($body, ['delivery_mode' => 2]);
```

3. 消费者一定要确保消息消费成功,手动ack
>消费者在业务逻辑成功的情况下，将消息通知给mq服务器端删除
```php
$callback = function ($msg) {
//    echo " [x] Received ", $msg->body, "\n";
    // 手动确认ack，确保消息已经处理
    $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
};
//公平调度
$channel->basic_qos(null,1,null);
$channel->basic_consume('hello',false,false,false,false,false,$callback);
```
![](https://oss.wyxxt.org.cn/images/2021/09/18/d5954584c36aeffcc38eaeb8e02a8ad7.png)


### 二、mq数据有问题，导致消费端报错，怎么解决？
判断错误原因，如果是调用第三方错误，进行重试。
如果是代码问题，需要消费者发布版本才能解决的问题，mq不能一直重试，应该放入消息日志表后期处理。

### 三、mq如何保证消息的幂等性？
1、消费者没有在规定的时间内或者抛出异常的情况下，消费的结果没有给mq服务器端的情况下，mq服务器端会默认为消费者消费失败，自动触发重试机制。
2、mq在重试的过程中，会造成我们的消费者重复消费的问题。
如何解决：
1、生产者在消息头中设置全局的业务id，比如订单号作为全局的id
2、消费者根据全局id判断是否处理过此业务逻辑，如果已经处理过，手动发ack给mq服务器端
重试的请求和第一次请求是否会产生并发冲突？
1、重试一般的情况下都是间隔的，一起执行到相同的代码概率极低
2、就算有情况1，全局id作为唯一id存入数据库即可，保证多线程安全