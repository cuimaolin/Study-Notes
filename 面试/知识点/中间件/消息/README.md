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

> | 场景         | 措施                                                         |
> | ------------ | ------------------------------------------------------------ |
> | 生产者       | 将发送消息的异步操作转换为同步操作，消息发送失败时检查原因，并重新发送 |
> | 消费者       | 把自动提交转换为手动提交                                     |
> | Kafka 服务端 | 1. 设置acks = all，即让消息被所有副本接受成功之后才算真正被发送成功<br />2. 增大副本的数量 |

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

