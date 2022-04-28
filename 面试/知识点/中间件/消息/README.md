kafka的使用场景（今日头条测开二面，百度一面）

> [典型应用场景](https://support.huaweicloud.com/productdesc-kafka/kafka-scenarios.html)
>
> - 异步通信
> - 错峰流控与流量削峰
> - 日志同步

为什么使用RabbitMQ，而不是采用其它消息队列？（深信服二面）

> RabbitMQ和Kafka的区别：
>
> - 通常会使用Kafka作为消息传输的数据管道，RabbitMQ作为交易数据作为数据传输管道，主要的取舍因素则是存在丢数据的可能
> - RabbitMQ在金融场景下经常使用，具有较高的严谨性，数据丢失的可能性更小，同时具备更高的实时性
> - Kafka优势主要体现在吞吐量上，虽然可以通过策略实现数据不丢失，但从严谨性角度来讲，大不如RabbitMQ
> - 而且由于Kafka保证每条数据最少送达一次，有较小的概率会出现数据重复发送的情况

MQ发送消息的模式？（腾讯CDG一面）

> - 点对点模式。（一对一，消费者主动拉取数据，消息收到后消息清除）
> - 发布/订阅模式。（一对多，消费者消费数据之后不会清除消息）

MQ丢包怎么处理？（腾讯CDG一面，百度一面）

> 一、生产者丢失消息的情况
>
> 生产者(Producer) 调用`send`方法发送消息之后，消息可能因为网络问题并没有发送过去。
>
> 所以，我们不能默认在调用`send`方法发送消息之后消息发送成功了。为了确定消息是发送成功，我们要判断消息发送的结果。但是要注意的是 Kafka 生产者(Producer) 使用 `send` 方法发送消息实际上是异步的操作，我们可以通过 `get()`方法获取调用结果，但是这样也让它变为了同步操作。如果消息发送失败的话，我们检查失败的原因之后重新发送即可！
>
> 二、消费者丢失消息的情况
>
> 我们知道消息在被追加到 Partition(分区)的时候都会分配一个特定的偏移量（offset）。偏移量（offset)表示 Consumer 当前消费到的 Partition(分区)的所在的位置。Kafka 通过偏移量（offset）可以保证消息在分区内的顺序性。
>
> 当消费者拉取到了分区的某个消息之后，消费者会自动提交了 offset。自动提交的话会有一个问题，试想一下，当消费者刚拿到这个消息准备进行真正消费的时候，突然挂掉了，消息实际上并没有被消费，但是 offset 却被自动提交了。
>
> 解决办法也比较粗暴，我们手动关闭自动提交 offset，每次在真正消费完消息之后再自己手动提交 offset 。 但是，细心的朋友一定会发现，这样会带来消息被重新消费的问题。比如你刚刚消费完消息之后，还没提交 offset，结果自己挂掉了，那么这个消息理论上就会被消费两次。
>
> 三、Kafka弄丢消息的情况
>
> 我们知道 Kafka 为分区（Partition）引入了多副本（Replica）机制。分区（Partition）中的多个副本之间会有一个叫做 leader 的家伙，其他副本称为 follower。我们发送的消息会被发送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步。生产者和消费者只与 leader 副本交互。可以理解为其他副本只是 leader 副本的拷贝，它们的存在只是为了保证消息存储的安全性。
>
> 试想一种情况：假如 leader 副本所在的 broker 突然挂掉，那么就要从 follower 副本重新选出一个 leader ，但是 leader 的数据还有一些没有被 follower 副本的同步的话，就会造成消息丢失。
>
> 1、**设置 acks = all**
>
> 解决办法就是我们设置 **acks = all**。acks 是 Kafka 生产者(Producer) 很重要的一个参数。
>
> acks 的默认值即为1，代表我们的消息被leader副本接收之后就算被成功发送。当我们配置 **acks = all** 代表则所有副本都要接收到该消息之后该消息才算真正成功被发送。
>
> 2、**设置 replication.factor >= 3**
>
> 为了保证 leader 副本能有 follower 副本能同步消息，我们一般会为 topic 设置 **replication.factor >= 3**。这样就可以保证每个 分区(partition) 至少有 3 个副本。虽然造成了数据冗余，但是带来了数据的安全性。
>
> 3、**设置 min.insync.replicas > 1**
>
> 一般情况下我们还需要设置 **min.insync.replicas> 1** ，这样配置代表消息至少要被写入到 2 个副本才算是被成功发送。**min.insync.replicas** 的默认值为 1 ，在实际生产中应尽量避免默认值 1。
>
> 但是，为了保证整个 Kafka 服务的高可用性，你需要确保 **replication.factor > min.insync.replicas** 。为什么呢？设想一下假如两者相等的话，只要是有一个副本挂掉，整个分区就无法正常工作了。这明显违反高可用性！一般推荐设置成 **replication.factor = min.insync.replicas + 1**。
>
> 4、**设置 unclean.leader.election.enable = false**
>
> > **Kafka 0.11.0.0版本开始 unclean.leader.election.enable 参数的默认值由原来的true 改为false**
>
> 我们最开始也说了我们发送的消息会被发送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步。多个 follower 副本之间的消息同步情况不一样，当我们配置了 **unclean.leader.election.enable = false** 的话，当 leader 副本发生故障时就不会从 follower 副本中和 leader 同步程度达不到要求的副本中选择出 leader ，这样降低了消息丢失的可能性。

Kafka如何搭建生产者和消费者（腾讯CDG事务开发一面）

Kafka的零拷贝（百度一面）

> [图解Kafka的零拷贝技术到底有多牛？](https://cloud.tencent.com/developer/article/1421266)
>
> - DMA，全称叫Direct Memory Access，一种可让某些硬件子系统去直接访问系统主内存，而不用依赖CPU的计算机系统的功能
>
> - 有了DMA后，就可以实现绝对的零拷贝了，因为网卡是直接去访问系统主内存的

消费者组（腾讯CDG事务开发一面）

> [怎么理解 Kafka 消费者与消费组之间的关系?](https://segmentfault.com/a/1190000039125247)
>
> - 如果所有的消费者都隶属于同一个消费组，那么所有的消息都会被均衡地投递给每一个消费者，即每条消息只会被一个消费者处理，这就相当于点对点模式的应用。
> - 如果所有的消费者都隶属于不同的消费组，那么所有的消息都会被广播给所有的消费者，即每条消息会被所有的消费者处理，这就相当于发布/订阅模式的应用。

Kafka的分布式（Lazada一面，百度一面）

> ![Kafka Topic Partition](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/KafkaTopicPartitioning.png)
>
> 比较重要的几个概念
>
> 1. **Producer（生产者）** : 产生消息的一方。
> 2. **Consumer（消费者）** : 消费消息的一方。
> 3. **Broker（代理）** : 可以看作是一个独立的 Kafka 实例。多个 Kafka Broker 组成一个 Kafka Cluster。
>
> 每个 Broker 中又包含了 Topic 以及 Partition 这两个重要的概念：
>
> - **Topic（主题）** : Producer 将消息发送到特定的主题，Consumer 通过订阅特定的 Topic(主题) 来消费消息。
> - **Partition（分区）** : Partition 属于 Topic 的一部分。一个 Topic 可以有多个 Partition ，并且同一 Topic 下的 Partition 可以分布在不同的 Broker 上，这也就表明一个 Topic 可以横跨多个 Broker 。
> - **Replica（副本）**：分区（Partition）中的多个副本之间会有一个叫做 leader 的家伙，其他副本称为 follower。我们发送的消息会被发送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步。
>
> **Kafka 的多分区（Partition）以及多副本（Replica）机制有什么好处呢？**
>
> 1. Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力（负载均衡）。
> 2. Partition 可以指定对应的 Replica 数, 这也极大地提高了消息存储的安全性, 提高了容灾能力，不过也相应的增加了所需要的存储空间。

