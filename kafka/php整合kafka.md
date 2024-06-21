### 一 通过php原生扩展实现

1.  安装方式

使用 Composer 安装:
添加 composer 依赖 nmred/kafka-php 到项目的 composer.json 文件中即可，如：

```json
{ "require": { "nmred/kafka-php": "0.2.*" } }
```

2.  详细配置


![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_525cc7a808fd7f975bff4d5118858c40.jpg)



3.  TP5使用配置


```php

//php-kafka配置
'phpkafka'=>[
 "metadataBrokerList"=>'10.19.185.17:4400', //kafka连接地址,集群用逗号隔开
 "brokerVersion"=>'2.4.0', //broker版本
 "requiredAck"=> 1, //该字段指示领导者经纪人在响应请求之前必须从ISR brokers收到多少确认：0 =brokers不向客户端发送任何响应/确认，1 =只有 leader broker需要确认消息，-1或all=代理将阻塞，直到所有同步副本（ISR）或代理的in.sync.replicas设置提交了消息，然后再发送响应
 "isAsyn"=>false, //是否使用异步生产消息
 "produceInterval"=>500, //异步生成消息时执行生产消息请求的时间间隔
 "metadataRefreshIntervalMs" =>10000//主题元数据刷新间隔（以毫秒为单位）。错误时元数据会自动刷新并连接。使用-1禁用间隔刷新。
]

```

4. producer代码:

```php

<?php
namespace app\home\command;

use think\Config;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class ProducerTest extends Command
{
 protected function configure()
 {
 $this->setName('producer')->setDescription('this command is to product data use kafka');
 }

 //异步生产
 public function producer_php(){
 $kafka_config=Config::get('phpkafka');
 $config = \Kafka\ProducerConfig::getInstance();
 $config->setMetadataRefreshIntervalMs($kafka_config['metadataRefreshIntervalMs']);
 $config->setMetadataBrokerList($kafka_config['metadataBrokerList']);
 $config->setBrokerVersion($kafka_config['brokerVersion']);
 $config->setRequiredAck($kafka_config['requiredAck']);
 $config->setIsAsyn(true);
 $config->setProduceInterval($kafka_config['produceInterval']);
 $producer = new \Kafka\Producer(function() {
 return array(
 array(
 'topic' => 'test',
 'value' => 'test....message.',
 'key' => 'testkey',
 ),
 );
 });
// $producer->setLogger($logger);
 $producer->success(function($result) {
 echo 'success---';
 var_dump($result);
 });
 $producer->error(function($errorCode) {
 echo 'error---';
 var_dump($errorCode);
 });
 $producer->send(true);
 }

 //同步生产
 public function producer_php2(){
 $kafka_config=Config::get('phpkafka');
 $config = \Kafka\ProducerConfig::getInstance();
 $config->setMetadataRefreshIntervalMs($kafka_config['metadataRefreshIntervalMs']);
 $config->setMetadataBrokerList($kafka_config['metadataBrokerList']);
 $config->setBrokerVersion($kafka_config['brokerVersion']);
 $config->setRequiredAck($kafka_config['requiredAck']);
 $config->setIsAsyn($kafka_config['isAsyn']);
 $config->setProduceInterval($kafka_config['produceInterval']);
 $producer = new \Kafka\Producer();
 for($i = 0; $i < 10000; $i++) {
 $result = $producer->send(array(
 array(
 'topic' => 'test',
 'value' => 'test....message.'.$i,
 'key' => '',
 ),
 ));
 echo '.';
// var_dump($result);
 }
 }


 protected function execute(Input $input, Output $output)
 {
 //单进程脚本测试
 $output->writeln("TestCommand:".$input);
// $this->producer_php();
 $this->producer_php2();
 }
}

```

脚本测试执行 php think producer


5. consumer代码:

```php

<?php
namespace app\home\command;

use think\console\Command;
use think\console\Input;
use think\console\Output;

class ConsumerTest extends Command
{
 protected function configure()
 {
 $this->setName('consumer')->setDescription('this command is to accept data from kafka ');
 }


 
 //消费
 public function consumerphp(){
 $kafka_config=Config::get('phpkafka');
 $config = \Kafka\ConsumerConfig::getInstance();
 $config->setMetadataRefreshIntervalMs($kafka_config['metadataRefreshIntervalMs']);
 $config->setMetadataBrokerList($kafka_config['metadataBrokerList']);
 $config->setGroupId('test');
 $config->setBrokerVersion($kafka_config['brokerVersion']);
 $config->setTopics(array('test'));
 // $config->setOffsetReset('earliest');
 $consumer = new \Kafka\Consumer();
 $consumer->start(function($topic, $part, $message) {
 // var_dump($message);
 echo $message['offset'];
 });
 }
 


 protected function execute(Input $input, Output $output)
 {
 $output->writeln("TestCommand:".$input);
 $this->consumerphp();
 }
}

```

脚本测试执行 php think consumer



参考github地址:https://github.com/weiboad/kafka-php



### 二 通过C扩展实现(推荐使用)


1.  安装方式

安装kafka扩展

#### 安装librdkafka

```shell
git clone https://github.com/edenhill/librdkafka.git
cd librdkafka
./configure
make
make install
```

以上方式不行可以直接通过yum源安装:

```shell
yum install librdkafka-devel
```

#### 安装php-rdkafka

```shell
git clone https://github.com/arnaud-lb/php-rdkafka.git
cd php-rdkafka
phpize
./configure
make all -j 5
make install
```

#### 在php.ini加入如下信息

```shell
vim /usr/local/php/etc/php.ini
extension=rdkafka.so
```

通过php -m查看到扩展rdkafka表示安装成功

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_eba6746c0074a73db1369c6c8362e02e.jpg)

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_33022c2581d301673218213db382bd1f.jpg)



2. 常用配置项

生产者配置:


![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_d6686936a0a4d99a0a19e389604323d1.jpg)




生产者topic配置属性：

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_3ecb9be37c9e673b7b04478d21d914b9.jpg)



消费者配置：

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_7ce466a56a1a189f141fa2abe066437d.jpg)


消费者topic配置属性：

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_5c11c50bdb80ea17e7cc6a2724685b45.jpg)

3. 生产者代码:

```php
<?php
namespace app\home\command;

use think\Config;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class RdkafkaProducerTest extends Command
{
 protected function configure()
 {
 $this->setName('rdkafka-producer')->setDescription('this command is to product data use kafka');
 }



 public function producer(){
 $conf = new \RdKafka\Conf();
 $conf->set('metadata.broker.list', '10.19.185.17:4400');

 //如果只需要生产一次，并且希望保留原始的生产消息，取消注释下面的行
 //$conf->set('enable.idempotence', 'true');

 $producer = new \RdKafka\Producer($conf);

 //$conf = new RdKafka\TopicConf();
 //$conf->set("...", "...");
 //$topic = $kafka->newTopic("myTopic", $conf);
 $topic = $producer->newTopic("rdkafka-test3");

 for ($i = 0; $i < 10; $i++) {
 $topic->produce(RD_KAFKA_PARTITION_UA, 0, "Message $i");
 //非阻塞轮询,来提供排队的交付报告回调
 $producer->poll(0);
 }

 for ($flushRetries = 0; $flushRetries < 10; $flushRetries++) {
 //发送完消息清除消息
 //$producer->purge(RD_KAFKA_PURGE_F_QUEUE);

 //确保在终止之前已完成所有排队和进行中的生产请求
 $result = $producer->flush(10000);
 if (RD_KAFKA_RESP_ERR_NO_ERROR === $result) {
 echo '.';
 break;
 }
 }

 if (RD_KAFKA_RESP_ERR_NO_ERROR !== $result) {
 throw new \RuntimeException('Was unable to flush, messages might be lost!');
 }
 }


 protected function execute(Input $input, Output $output)
 {
 //单进程脚本测试
 $output->writeln("TestCommand:".$input);
// $this->producer_php();
 $this->producer();
 }
}

```

脚本测试执行 php think rdkafka-producer

4. 消费者代码:

```php

<?php
namespace app\home\command;

use think\Config;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class RdkafkaConsumerTest extends Command
{
 protected function configure()
 {
 $this->setName('rdkafka-consumer')->setDescription('this command is to accept data from kafka ');
 }


 //消费
 public function consumerphp(){
 $conf = new \RdKafka\Conf();

// Set a rebalance callback to log partition assignments (optional)
 $conf->setRebalanceCb(function (\RdKafka\KafkaConsumer $kafka, $err, array $partitions = null) {
 switch ($err) {
 case RD_KAFKA_RESP_ERR__ASSIGN_PARTITIONS:
 echo "Assign: ";
 var_dump($partitions);
 $kafka->assign($partitions);
 break;

 case RD_KAFKA_RESP_ERR__REVOKE_PARTITIONS:
 echo "Revoke: ";
 var_dump($partitions);
 $kafka->assign(NULL);
 break;

 default:
 throw new \Exception($err);
 }
 });

// Configure the group.id. All consumer with the same group.id will consume
// different partitions.
 $conf->set('group.id', 'myConsumerGroup');

 // kafka的broker连接地址
 $conf->set('metadata.broker.list', '10.19.185.17:4400');

 //smallest,earliest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
 //largest,latest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据
 $conf->set('auto.offset.reset', 'smallest');

 $consumer = new \RdKafka\KafkaConsumer($conf);

 // topic列表
 $consumer->subscribe(['rdkafka-test3']);

 echo "Waiting for partition assignment... (make take some time when\n";
 echo "quickly re-joining the group after leaving it.)\n";

 while (true) {
 $message = $consumer->consume(120*1000);
 switch ($message->err) {
 case RD_KAFKA_RESP_ERR_NO_ERROR:
 var_dump($message);
 break;
 case RD_KAFKA_RESP_ERR__PARTITION_EOF:
 echo "No more messages; will wait for more\n";
 break;
 case RD_KAFKA_RESP_ERR__TIMED_OUT:
 echo "Timed out\n";
 break;
 default:
 throw new \Exception($message->errstr(), $message->err);
 break;
 }
 }
 }


 protected function execute(Input $input, Output $output)
 {
 $output->writeln("TestCommand:".$input);
 $this->consumerphp();
 }
}

```

脚本测试执行 php think rdkafka-consumer

5. 消费端多数据源配置:

```php
// topic列表,配置多个逗号隔开即可
$consumer->subscribe(['rdkafka-test1','rdkafka-test2']);
```

6. 设置手动提交offset


```php
<?php
namespace app\home\command;

use think\Config;
use think\console\Command;
use think\console\Input;
use think\console\Output;

class RdkafkaConsumerTest extends Command
{
 protected function configure()
 {
 $this->setName('rdkafka-consumer')->setDescription('this command is to accept data from kafka ');
 }


 //消费
 public function consumerphp(){
 $conf = new \RdKafka\Conf();

// Set a rebalance callback to log partition assignments (optional)
 $conf->setRebalanceCb(function (\RdKafka\KafkaConsumer $kafka, $err, array $partitions = null) {
 switch ($err) {
 case RD_KAFKA_RESP_ERR__ASSIGN_PARTITIONS:
 echo "Assign: ";
 var_dump($partitions);
 $kafka->assign($partitions);
 break;

 case RD_KAFKA_RESP_ERR__REVOKE_PARTITIONS:
 echo "Revoke: ";
 var_dump($partitions);
 $kafka->assign(NULL);
 break;

 default:
 throw new \Exception($err);
 }
 });

// Configure the group.id. All consumer with the same group.id will consume
// different partitions.
 $conf->set('group.id', 'myConsumerGroup');

 // kafka的broker连接地址
 $conf->set('metadata.broker.list', '10.19.185.17:4400');

 //设置使用手动提交offset
 $conf->set('enable.auto.commit', 'false');

 //smallest,earliest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
 //largest,latest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据
 $conf->set('auto.offset.reset', 'smallest');
 $consumer = new \RdKafka\KafkaConsumer($conf);
 // topic列表,可以为多个,逗号隔开
 $consumer->subscribe(['rdkafka-test']);

 echo "Waiting for partition assignment... (make take some time when\n";
 echo "quickly re-joining the group after leaving it.)\n";

 while (true) {
 $message = $consumer->consume(120*1000);
 switch ($message->err) {
 case RD_KAFKA_RESP_ERR_NO_ERROR:
 var_dump($message);
 //消费完成后手动提交offset
 $consumer->commit($message);
 break;
 case RD_KAFKA_RESP_ERR__PARTITION_EOF:
 echo "No more messages; will wait for more\n";
 break;
 case RD_KAFKA_RESP_ERR__TIMED_OUT:
 echo "Timed out\n";
 break;
 default:
 throw new \Exception($message->errstr(), $message->err);
 break;
 }
 }
 }

 protected function execute(Input $input, Output $output)
 {
 $output->writeln("TestCommand:".$input);
 $this->consumerphp();
 }
}
```

详细配置项地址:https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md

函数配置使用参考文档:https://arnaud.le-blanc.net/php-rdkafka-doc/phpdoc/book.rdkafka.html

github源码地址:https://github.com/arnaud-lb/php-rdkafka