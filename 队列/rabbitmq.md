[RabbitMQ官网](https://www.rabbitmq.com/getstarted.html "RabbitMQ官网")

### RabbitMQ 中，出现了六种消息传播模式：
1. Simple Work Queue （简单工作队列）：也就是常说的点对点模式，一条消息由一个消费者进行消费。（当有多个消费者时，默认使用轮训机制把消息分配给消费者）。
2. Work Queues （工作队列）：也叫公平队列，能者多劳的消息队列模型。队列必须接收到来自消费者的   手动ack 才可以继续往消费者发送消息。
3. Publish/Subscribe （发布订阅模式）：一条消息被多个消费者消费。
4. Routing（路由模式）：有选择的接收消息。
5. Topics （主题模式）：通过一定的规则来选择性的接收消息
6. RPC 模式：发布者发布消息，并且通过 RPC 方式等待结果。目前这个应该场景少，而且代码也较为复杂，本章不做细讲。

注意：官网最后有 Publisher Confirms 为消息确认机制。指的是生产者如何发送可靠的消息。

### RabbitMQ的四种Exchange

在了解这些消息模式的时候，引入了一个概念 Exchange（交换机）：
在发布订阅里面有对这个概念做解释：

> RabbitMQ消息传递模型中的核心思想是生产者从不将任何消息直接发送到队列。实际上，生产者经常甚至根本不知道是否将消息传递到任何队列。
> 相反，生产者只能将消息发送到交换机。交流是一件非常简单的事情。一方面，它接收来自生产者的消息，另一方面，将它们推入队列。交易所必须确切知道如何处理收到的消息。是否应将其附加到特定队列？是否应该将其附加到许多队列中？还是应该丢弃它。规则由交换机类型定义 。

#### 而 Exchange 的类型有下面四种：

- direct（直连交换机）：将队列绑定到交换机，消息的 routeKey 需要与队列绑定的 routeKey 相同。
- fanout （扇形交换机）：不处理 routeKey ，直接把消息转发到与其绑定的所有队列中。
- topic（主题交换机）：根据一定的规则，根据 routeKey 把消息转发到符合规则的队列中，其中 # 用于匹配符合一个或者多个词（范围更广）， * 用于匹配一个词。
- headers （头部交换机）：根据消息的 headers 转发消息而不是根据 routeKey 来转发消息, 其中 header 是一个 Map，也就意味着不仅可以匹配字符串类型，也可以匹配其他类型数据。 规则可以分为所有键值对匹配或者单一键值对匹配。


| 消息模式                                                     | 交换机                |
| ------------------------------------------------------------ | --------------------- |
| Simple Work Queue （简单工作队列），Work Queues （工作队列） | 空交换机              |
| Publish/Subscribe （发布订阅模式）                           | fanout （扇形交换机） |
| Routing（路由模式）                                          | direct （直连交换机） |
| Topics（主题模式）                                           | topic（主题交换机）   |

### 一、 Simple Work Queue （简单工作队列）
![](https://oss.wyxxt.org.cn/images/2021/09/18/1d6f1f27fabfc9f7f3b6279d887fbe32.png)
```json
{
    "require": {
        "php-amqplib/php-amqplib": ">=2.9.0"
    }
}
```
```shell
composer install
```

#### 1. Sending
![file](https://oss.wyxxt.org.cn/images/2021/09/18/image-1585042220653.png)

```php
//send.php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
//1、创建连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//2、创建通道
$channel = $connection->channel();
//3、队列声明
$channel->queue_declare('hello', false, false, false, false);
//4、格式化数据
$msg = new AMQPMessage('Hello World!');
//5、基础发布
$channel->basic_publish($msg, '', 'hello');

echo " [x] Sent 'Hello World!'\n";
//6、关闭通道
$channel->close();
//7、关闭连接
$connection->close();

```
#### 2. Receiving

![](https://oss.wyxxt.org.cn/images/2021/09/18/db6b3f5ad8ad77401b91630047503a30.png)

```php
//receive.php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
//1、建立连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//2、创建通道
$channel = $connection->channel();


//3、队列声明
$channel->queue_declare('hello', false, false, false, false);

echo " [*] Waiting for messages. To exit press CTRL+C\n";

$callback = function ($msg) {
  echo ' [x] Received ', $msg->body, "\n";
};
//4、基础消费
$channel->basic_consume('hello', '', false, true, false, false, false, $callback);

//5、循环
while ($channel->is_consuming()) {
    $channel->wait();
}
```

### 二、Work Queue （工作队列）
![](https://oss.wyxxt.org.cn/images/2021/09/18/474c6a254c999cd9d6f1d3c7f220b9dc.png)

```php
//new_task.php
$data = implode(' ', array_slice($argv, 1));
if (empty($data)) {
    $data = "Hello World!";
}
$msg = new AMQPMessage($data);

$channel->basic_publish($msg, '', 'hello');

echo ' [x] Sent ', $data, "\n";
```

```php
//worker.php
$callback = function ($msg) {
  echo ' [x] Received ', $msg->body, "\n";
  sleep(substr_count($msg->body, '.'));
  echo " [x] Done\n";
};

$channel->basic_consume('hello', '', false, true, false, false, $callback);
```

```sh
# shell 1
php worker.php
# shell 2
php worker.php
# shell 3
php new_task.php First message.
php new_task.php Second message..
php new_task.php Third message...
php new_task.php Fourth message....
php new_task.php Fifth message.....
```
![](https://oss.wyxxt.org.cn/images/2021/09/18/10c6b2626fc17f7c51edb293b686d386.png)

### 三、Publish/Subscribe （发布订阅模式）
**生产者生产消息到交换机，生产者并不与队列交流。队列由消费者根据交换机创建出来并绑定到交换机。**
![](https://oss.wyxxt.org.cn/images/2021/09/18/e3271af4664d0d6afe193e11891e4913.png)

```php
//emit_log.php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
// 交换机声明
$channel->exchange_declare('logs', 'fanout', false, false, false);

$data = implode(' ', array_slice($argv, 1));
if (empty($data)) {
    $data = "info: Hello World!";
}
$msg = new AMQPMessage($data);

$channel->basic_publish($msg, 'logs');

echo ' [x] Sent ', $data, "\n";

$channel->close();
$connection->close();
```

```php
//receive_logs.php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
// 交换机声明（扇形交换机）
$channel->exchange_declare('logs', 'fanout', false, false, false);
// 队列声明
list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);
// 队列绑定到交换机
$channel->queue_bind($queue_name, 'logs');

echo " [*] Waiting for logs. To exit press CTRL+C\n";

$callback = function ($msg) {
    echo ' [x] ', $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

### 四、Routing（路由模式）
![](https://oss.wyxxt.org.cn/images/2021/09/18/aa6515a3e9de84af63deef6681ea7dba.png)

![](https://oss.wyxxt.org.cn/images/2021/09/18/7c2ec9931c7253acc547371d01a3e5cc.png)

![](https://oss.wyxxt.org.cn/images/2021/09/18/3653675409bff37069f88d0cdf4dc8d1.png)

```php
//emit_log_direct.php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
//交换机声明（直连交换机）
$channel->exchange_declare('direct_logs', 'direct', false, false, false);

$severity = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'info';

$data = implode(' ', array_slice($argv, 2));
if (empty($data)) {
    $data = "Hello World!";
}

$msg = new AMQPMessage($data);
//发布消息到direct_logs交换机，密钥为$severity
$channel->basic_publish($msg, 'direct_logs', $severity);

echo ' [x] Sent ', $severity, ':', $data, "\n";

$channel->close();
$connection->close();
```

```php
//receive_logs_direct.php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
//声明交换机（直连交换机）
$channel->exchange_declare('direct_logs', 'direct', false, false, false);

list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$severities = array_slice($argv, 1);
if (empty($severities)) {
    file_put_contents('php://stderr', "Usage: $argv[0] [info] [warning] [error]\n");
    exit(1);
}

foreach ($severities as $severity) {
	//队列绑定到交换机，密钥为$severity
    $channel->queue_bind($queue_name, 'direct_logs', $severity);
}

echo " [*] Waiting for logs. To exit press CTRL+C\n";

$callback = function ($msg) {
    echo ' [x] ', $msg->delivery_info['routing_key'], ':', $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

### 五、Topics （主题模式）

- *可以代替一个单词
- #可以代替零个或多个单词

![](https://oss.wyxxt.org.cn/images/2021/09/18/0af2d9df0b892368f9a9ef35421138e7.png)


在此示例中，我们将发送所有描述动物的消息。将使用包含三个词的路由密钥发送消息。路由键中的第一个单词将描述速度，第二个描述颜色，第三个描述物种:"<spead>.<colour>.<species>"
Q1与"*.orange.*"绑定，Q2与"*.*.rabbit"和"lazy.#"绑定。
- Q1对所有橙色动物都感兴趣
- Q2关注兔子的一切和速度慢的动物的一切

```php
//emit_log_topic.php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
//声明交换机（主题交换机）
$channel->exchange_declare('topic_logs', 'topic', false, false, false);

$routing_key = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'anonymous.info';
$data = implode(' ', array_slice($argv, 2));
if (empty($data)) {
    $data = "Hello World!";
}

$msg = new AMQPMessage($data);

$channel->basic_publish($msg, 'topic_logs', $routing_key);

echo ' [x] Sent ', $routing_key, ':', $data, "\n";

$channel->close();
$connection->close();
```

```php
//receive_logs_topic.php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
//声明交换机（主题交换机）
$channel->exchange_declare('topic_logs', 'topic', false, false, false);

list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$binding_keys = array_slice($argv, 1);
if (empty($binding_keys)) {
    file_put_contents('php://stderr', "Usage: $argv[0] [binding_key]\n");
    exit(1);
}

foreach ($binding_keys as $binding_key) {
	//队列绑定到交换机，密钥为$binding_key
    $channel->queue_bind($queue_name, 'topic_logs', $binding_key);
}

echo " [*] Waiting for logs. To exit press CTRL+C\n";

$callback = function ($msg) {
    echo ' [x] ', $msg->delivery_info['routing_key'], ':', $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```
![](https://oss.wyxxt.org.cn/images/2021/09/18/faa747e572f4a34c178ba6198f863cbf.png)

### 远程过程调用Remote procedure call (RPC)
![](https://oss.wyxxt.org.cn/images/2021/09/18/d0778172c87d8a900a8c9f4f8c758472.png)

我们的RPC将像这样工作：

- 客户端启动时，它将创建一个匿名排他回调队列。
- 对于RPC请求，客户端发送一条消息，该消息具有两个属性： reply_to（设置为回调队列）和correlation_id（设置为每个请求的唯一值）。
- 该请求被发送到rpc_queue队列。
- RPC工作程序（又名：服务器）正在等待该队列上的请求。出现请求时，它将使用reply_to字段中的队列来完成工作，并将消息和结果发送回客户端。
- 客户端等待回调队列上的数据。出现消息时，它将检查correlation_id属性。如果它与请求中的值匹配，则将响应返回给应用程序。

```php
//rpc_server.php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

//队列声明
$channel->queue_declare('rpc_queue', false, false, false, false);

function fib($n)
{
    if ($n == 0) {
        return 0;
    }
    if ($n == 1) {
        return 1;
    }
    return fib($n-1) + fib($n-2);
}

echo " [x] Awaiting RPC requests\n";
$callback = function ($req) {
    $n = intval($req->body);
    echo ' [.] fib(', $n, ")\n";

    $msg = new AMQPMessage(
        (string) fib($n),
        array('correlation_id' => $req->get('correlation_id')) //相关id（唯一值）
    );

	//服务端将消息根据reply_to发回客户端
    $req->delivery_info['channel']->basic_publish(
        $msg,
        '',
        $req->get('reply_to')
    );
	//ack确认
    $req->delivery_info['channel']->basic_ack(
        $req->delivery_info['delivery_tag']
    );
};

$channel->basic_qos(null, 1, null);
$channel->basic_consume('rpc_queue', '', false, false, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

```php
//rpc_client.php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

class FibonacciRpcClient
{
    private $connection;
    private $channel;
    private $callback_queue;
    private $response;
    private $corr_id;

    public function __construct()
    {
        $this->connection = new AMQPStreamConnection(
            'localhost',
            5672,
            'guest',
            'guest'
        );
        $this->channel = $this->connection->channel();
        list($this->callback_queue, ,) = $this->channel->queue_declare(
            "",
            false,
            false,
            true,
            false
        );
        $this->channel->basic_consume(
            $this->callback_queue,
            '',
            false,
            true,
            false,
            false,
            array(
                $this,
                'onResponse'
            )
        );
    }

    public function onResponse($rep)
    {
        if ($rep->get('correlation_id') == $this->corr_id) {
            $this->response = $rep->body;
        }
    }

    public function call($n)
    {
        $this->response = null;
        $this->corr_id = uniqid();

        $msg = new AMQPMessage(
            (string) $n,
            array(
                'correlation_id' => $this->corr_id,
                'reply_to' => $this->callback_queue
            )
        );
        $this->channel->basic_publish($msg, '', 'rpc_queue');
        while (!$this->response) {
            $this->channel->wait();
        }
        return intval($this->response);
    }
}

$fibonacci_rpc = new FibonacciRpcClient();
$response = $fibonacci_rpc->call(30);
echo ' [.] Got ', $response, "\n";
```