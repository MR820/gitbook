### 一、如何保证消息消费时的幂等性
　　其实消息重复消费的主要原因在于回馈机制（RabbitMQ是ack，Kafka是offset)，在某些场景中我们采用的回馈机制不同，原因也不同，例如消费者消费完消息后回复ack, 但是刚消费完还没来得及提交系统就重启了，这时候上来就pull消息的时候由于没有提交ack或者offset，消费的还是上条消息。

　　那么如何怎么来保证消息消费的幂等性呢？实际上我们只要保证多条相同的数据过来的时候只处理一条或者说多条处理和处理一条造成的结果相同即可，但是具体怎么做要根据业务需求来定，例如入库消息，先查一下消息是否已经入库啊或者说搞个唯一约束啊什么的，还有一些是天生保证幂等性就根本不用去管，例如redis就是天然幂等性。

　　还有一个问题，消费者消费消息的时候在某些场景下要放过消费不了的消息，遇到消费不了的消息通过日志记录一下或者搞个什么措施以后再来处理，但是一定要放过消息，因为在某些场景下例如spring-rabbitmq的默认回馈策略是出现异常就没有提交ack，导致了一直在重发那条消费异常的消息，而且一直还消费不了，这就尴尬了，后果你会懂的。
### 二、如何保证消息的可靠性传输
#### 1.生产者弄丢了数据
　　生产者将数据发送到RabbitMQ的时候，可能数据就在半路给搞丢了，因为网络啥的问题，都有可能。此时可以选择用RabbitMQ提供的事务功能，就是生产者发送数据之前开启RabbitMQ事务（channel.txSelect），然后发送消息，如果消息没有成功被RabbitMQ接收到，那么生产者会收到异常报错，此时就可以回滚事务（channel.txRollback），然后重试发送消息；如果收到了消息，那么可以提交事务（channel.txCommit）。但是问题是，RabbitMQ事务机制一搞，基本上吞吐量会下来，因为太耗性能。

　　所以一般来说，如果你要确保说写RabbitMQ的消息别丢，可以开启confirm模式，在生产者那里设置开启confirm模式之后，你每次写的消息都会分配一个唯一的id，然后如果写入了RabbitMQ中，RabbitMQ会给你回传一个ack消息，告诉你说这个消息ok了。如果RabbitMQ没能处理这个消息，会回调你一个nack接口，告诉你这个消息接收失败，你可以重试。而且你可以结合这个机制自己在内存里维护每个消息id的状态，如果超过一定时间还没接收到这个消息的回调，那么你可以重发。

　　事务机制和cnofirm机制最大的不同在于，事务机制是同步的，你提交一个事务之后会阻塞在那儿，但是confirm机制是异步的，你发送个消息之后就可以发送下一个消息，然后那个消息RabbitMQ接收了之后会异步回调你一个接口通知你这个消息接收到了。

　　所以一般在生产者这块避免数据丢失，都是用confirm机制的。

#### 2.RabbitMQ弄丢了数据

　　就是RabbitMQ自己弄丢了数据，这个你必须开启RabbitMQ的持久化，就是消息写入之后会持久化到磁盘，哪怕是RabbitMQ自己挂了，恢复之后会自动读取之前存储的数据，一般数据不会丢。除非极其罕见的是，RabbitMQ还没持久化，自己就挂了，可能导致少量数据会丢失的，但是这个概率较小。

　　设置持久化有两个步骤，第一个是创建queue的时候将其设置为持久化的，这样就可以保证RabbitMQ持久化queue的元数据，但是不会持久化queue里的数据；第二个是发送消息的时候将消息的deliveryMode设置为2，就是将消息设置为持久化的，此时RabbitMQ就会将消息持久化到磁盘上去。必须要同时设置这两个持久化才行，RabbitMQ哪怕是挂了，再次重启，也会从磁盘上重启恢复queue，恢复这个queue里的数据。

　　而且持久化可以跟生产者那边的confirm机制配合起来，只有消息被持久化到磁盘之后，才会通知生产者ack了，所以哪怕是在持久化到磁盘之前，RabbitMQ挂了，数据丢了，生产者收不到ack，你也是可以自己重发的。

　　哪怕是你给RabbitMQ开启了持久化机制，也有一种可能，就是这个消息写到了RabbitMQ中，但是还没来得及持久化到磁盘上，结果不巧，此时RabbitMQ挂了，就会导致内存里的一点点数据会丢失。

#### 3.消费端弄丢了数据

　　RabbitMQ如果丢失了数据，主要是因为你消费的时候，刚消费到，还没处理，结果进程挂了，比如重启了，那么就尴尬了，RabbitMQ认为你都消费了，这数据就丢了。

　　这个时候得用RabbitMQ提供的ack机制，简单来说，就是你关闭RabbitMQ自动ack，可以通过一个api来调用就行，然后每次你自己代码里确保处理完的时候，再程序里ack一把。这样的话，如果你还没处理完，不就没有ack？那RabbitMQ就认为你还没处理完，这个时候RabbitMQ会把这个消费分配给别的consumer去处理，消息是不会丢的。

### 三、如何保证消息的顺序性
　　因为在某些情况下我们扔进MQ中的消息是要严格保证顺序的，尤其涉及到订单什么的业务需求，消费的时候也是要严格保证顺序，不然会出大问题的。

#### 1.先看看顺序会错乱的俩场景
1. rabbitmq：一个queue，多个consumer，这不明显乱了
2. kafka：一个topic，一个partition，一个consumer，内部多线程，这不也明显乱了

#### 2.如何来保证消息的顺序性呢？

1. rabbitmq：拆分多个queue，每个queue一个consumer，就是多一些queue而已，确实是麻烦点；或者就一个queue但是对应一个consumer，然后这个consumer内部用内存队列做排队，然后分发给底层不同的worker来处理。
2. kafka：一个topic，一个partition，一个consumer，内部单线程消费，写N个内存queue，然后N个线程分别消费一个内存queue即可。

### 四、如何解决消息队列的延时以及过期失效问题？消息队列满了以后该怎么处理？有几百万消息持续积压几小时怎么解决？

#### 1.大量消息在mq里积压了几个小时了还没解决
　　几千万条数据在MQ里积压了七八个小时，从下午4点多，积压到了晚上很晚，10点多，11点多
这个是我们真实遇到过的一个场景，确实是线上故障了，这个时候要不然就是修复consumer的问题，让他恢复消费速度，然后傻傻的等待几个小时消费完毕。这个肯定不能在面试的时候说吧。
　　一个消费者一秒是1000条，一秒3个消费者是3000条，一分钟是18万条，1000多万条，所以如果你积压了几百万到上千万的数据，即使消费者恢复了，也需要大概1小时的时间才能恢复过来。
　　一般这个时候，只能操作临时紧急扩容了，具体操作步骤和思路如下：
1. 先修复consumer的问题，确保其恢复消费速度，然后将现有cnosumer都停掉。
2. 新建一个topic，partition是原来的10倍，临时建立好原先10倍或者20倍的queue数量。
3. 然后写一个临时的分发数据的consumer程序，这个程序部署上去消费积压的数据，消费之后不做耗时的处理，直接均匀轮询写入临时建立好的10倍数量的queue。
4. 接着临时征用10倍的机器来部署consumer，每一批consumer消费一个临时queue的数据。
5. 这种做法相当于是临时将queue资源和consumer资源扩大10倍，以正常的10倍速度来消费数据。等快速消费完积压数据之后，得恢复原先部署架构，重新用原先的consumer机器来消费消息。
#### 2.消息队列过期失效问题
　　假设你用的是rabbitmq，rabbitmq是可以设置过期时间的，就是TTL，如果消息在queue中积压超过一定的时间就会被rabbitmq给清理掉，这个数据就没了。那这就是第二个坑了。这就不是说数据会大量积压在mq里，而是大量的数据会直接搞丢。

　　这个情况下，就不是说要增加consumer消费积压的消息，因为实际上没啥积压，而是丢了大量的消息。我们可以采取一个方案，就是批量重导，这个我们之前线上也有类似的场景干过。就是大量积压的时候，我们当时就直接丢弃数据了，然后等过了高峰期以后，比如大家一起喝咖啡熬夜到晚上12点以后，用户都睡觉了。

　　这个时候我们就开始写程序，将丢失的那批数据，写个临时程序，一点一点的查出来，然后重新灌入mq里面去，把白天丢的数据给他补回来。也只能是这样了。

　　假设1万个订单积压在mq里面，没有处理，其中1000个订单都丢了，你只能手动写程序把那1000个订单给查出来，手动发到mq里去再补一次。

#### 3.消息队列满了怎么搞？
　　如果走的方式是消息积压在mq里，那么如果你很长时间都没处理掉，此时导致mq都快写满了，咋办？这个还有别的办法吗？没有，谁让你第一个方案执行的太慢了，你临时写程序，接入数据来消费，消费一个丢弃一个，都不要了，快速消费掉所有的消息。然后走第二个方案，到了晚上再补数据吧。