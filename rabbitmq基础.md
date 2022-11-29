# 一、MQ相关概念

## **1.什么是 MQ** 

MQ(message queue)，从字面意思上看，本质是个队列，FIFO 先入先出，只不过队列中存放的内容是message 而已，还是一种跨进程的通信机制，用于上下游传递消息。在互联网架构中，MQ 是一种非常常见的上下游“逻辑解耦+物理解耦”的消息通信服务。使用了 MQ 之后，消息发送上游只需要依赖 MQ，不用依赖其他服务

## 2.**为什么要用 MQ** （重点）

### I.流量消峰

流量削峰也是**消息队列的常用场景**。我们做秒杀实现的时候，需要避免流量暴涨，打垮应用系统的风险。可以在应用前面加入消息队列。

![img](https://pic2.zhimg.com/80/v2-f1ab1f75f654f3aa1cfcd0be564cccf5_720w.webp)


假设秒杀系统每秒最多可以处理`2k`个请求，每秒却有`5k`的请求过来，可以引入消息队列，秒杀系统每秒从消息队列拉2k请求处理得了。
有些伙伴担心这样会出现**消息积压**的问题，

- 首先秒杀活动不会每时每刻都那么多请求过来，高峰期过去后，积压的请求可以慢慢处理；
- 其次，如果消息队列长度超过最大数量，可以直接抛弃用户请求或跳转到错误页面；

### II.应用解耦

以电商应用为例，应用中有订单系统、库存系统、物流系统、支付系统。用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障，都会造成下单操作异常。当转变成基于消息队列的方式后，系统间调用的问题会减少很多，比如物流系统因为发生故障，需要几分钟来修复。在这几分钟的时间里，物流系统要处理的内存被缓存在消息队列中，用户的下单操作可以正常完成。当物流系统恢复后，继续处理订单信息即可，中单用户感受不到物流系统的故障，提升系统的可用性。

![image-20221118153636830](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118153636830.png)

### III.异步处理

有些服务间调用是异步的，例如 A 调用 B，B 需要花费很长时间执行，但是 A 需要知道 B 什么时候可以执行完，以前一般有两种方式，A 过一段时间去调用 B 的查询 api 查询。或者 A 提供一个 callback api， B 执行完之后调用 api 通知 A 服务。这两种方式都不是很优雅，使用消息总线，可以很方便解决这个问题，**A 调用 B 服务后，只需要监听 B 处理完成的消息，当 B 处理完成后，会发送一条消息给 MQ，MQ 会将此消息转发给 A 服务。**这样 A 服务既不用循环调用 B 的查询 api，也不用提供 callback api。同样 B 服务也不用做这些操作。A 服务还能及时的得到异步处理成功的消息。

<img src="https://pic4.zhimg.com/v2-fe1741d0a1fe9bfd31cbf683cec119af_r.jpg" alt="img" style="zoom:67%;" />

<img src="https://pic4.zhimg.com/v2-9414030e93355ac8cd45e505f4f26d0b_r.jpg" alt="img" style="zoom:67%;" />

## 3.消息队列比较

| 特性                    | ActiveMq                                                     | RabbitMq                                                     | RocketMQ                                                     | Kafka                                                        |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 成熟度                  | 成熟                                                         | 成熟                                                         | 比较成熟                                                     | 成熟的日志领域                                               |
| 时效性                  |                                                              | 微秒级                                                       | 毫秒级                                                       | 毫秒级                                                       |
| 社区活跃度              | 低                                                           | 高                                                           | 高                                                           | 高                                                           |
| 单机吞吐量              | 万级，吞吐量比RocketMQ和Kafka要低了一个数量级                | 万级，吞吐量比RocketMQ和Kafka要低了一个数量级                | 10万级，RocketMQ也是可以支撑高吞吐的一种MQ                   | 10万级别，这是kafka最大的优点，就是吞吐量高。一般配合大数据类的系统来进行实时数据计算、日志采集等场景 |
| topic数量对吞吐量的影响 |                                                              |                                                              | topic可以达到几百，几千个的级别，吞吐量会有较小幅度的下降这是RocketMQ的一大优势，在同等机器下，可以支撑大量的topic | topic从几十个到几百个的时候，吞吐量会大幅度下降所以在同等机器下，kafka尽量保证topic数量不要过多。如果要支撑大规模topic，需要增加更多的机器资源 |
| 可用性                  | 高，基于主从架构实现高可用性                                 | 高，基于主从架构实现高可用性                                 | 非常高，分布式架构                                           | 非常高，kafka是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用 |
| 消息可靠性              | 有较低的概率丢失数据                                         |                                                              | 经过参数优化配置，可以做到0丢失                              | 经过参数优化配置，消息可以做到0丢失                          |
| 功能支持                | MQ领域的功能极其完备                                         | 基于erlang开发，所以并发能力很强，性能极其好，延时很低       | MQ功能较为完善，还是分布式的，扩展性好                       | 功能较为简单，主要支持简单的MQ功能，在大数据领域的实时计算以及日志采集被大规模使用，是事实上的标准 |
| 优劣势总结              | 非常成熟，功能强大，在业内大量的公司以及项目中都有应用偶尔会有较低概率丢失消息而且现在社区以及国内应用都越来越少，官方社区现维护越来越少，几个月才发布一个版本而且确实主要是基于解耦和异步来用的，较少在大规模吞吐的场景中使用 | rlang语言开发，性能极其好，延时很低；吞吐量到万级，MQ功能比较完备而且开源提供的管理界面非常棒，用起来很好用社区相对比较活跃，几乎每个月都发布几个版本分在国内一些互联网公司近几年用rabbitmq也比较多一些但是问题也是显而易见的，RabbitMQ确实吞吐量会低一些，这是因为他做的实现机制比较重。而且erlang开发，国内有几个公司有实力做erlang源码级别的研究和定制？如果说你没这个实力的话，确实偶尔会有一些问题，你很难去看懂源码，你公司对这个东西的掌控很弱，基本职能依赖于开源社区的快速维护和修复bug。而且rabbitmq集群动态扩展会很麻烦，不过这个我觉得还好。其实主要是erlang语言本身带来的问题。很难读源码，很难定制和掌控。 | 接口简单易用，而且毕竟在阿里大规模应用过，有阿里品牌保障日处理消息上百亿之多，可以做到大规模吞吐，性能也非常好，分布式扩展也很方便，社区维护还可以，可靠性和可用性都是ok的，还可以支撑大规模的topic数量，支持复杂MQ业务场景而且一个很大的优势在于，阿里出品都是java系的，我们可以自己阅读源码，定制自己公司的MQ，可以掌控社区活跃度相对较为一般，不过也还可以，文档相对来说简单一些，然后接口这块不是按照标准JMS规范走的有些系统要迁移需要修改大量代码还有就是阿里出台的技术，你得做好这个技术万一被抛弃，社区黄掉的风险，那如果你们公司有技术实力我觉得用RocketMQ挺好的 | kafka的特点其实很明显，就是仅仅提供较少的核心功能，但是提供超高的吞吐量，ms级的延迟，极高的可用性以及可靠性，而且分布式可以任意扩展同时kafka最好是支撑较少的topic数量即可，保证其超高吞吐量而且kafka唯一的一点劣势是有可能消息重复消费，那么对数据准确性会造成极其轻微的影响，在大数据领域中以及日志采集中，这点轻微影响可以忽略这个特性天然适合大数据实时计算以及日志收集 |

# 二、RabbitMQ核心概念

## **1.四大核心概念** 

- **生产者**

产生数据发送消息的程序是生产者

- **交换机**

交换机是 RabbitMQ 非常重要的一个部件，**一方面它接收来自生产者的消息，另一方面它将消息推送到队列中。**交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推送到多个队列，亦或者是把消息丢弃，这个**得有交换机类型决定**

- **队列**

队列是 RabbitMQ 内部使用的一种数据结构，尽管消息流经 RabbitMQ 和应用程序，但它们只能存储在队列中。**队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。**许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式

- **消费者**

消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者

![image-20221118154243954](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118154243954.png)

- **Broker**：接收和分发消息的应用，RabbitMQ Server 就是 Message Broker

- **Virtual host**：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange／queue 等

- **Connection**：publisher／consumer 和 broker 之间的 TCP 连接

- **Channel**：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。**Channel 作为轻量级的**

- **Connection** **极大减少了操作系统建立** **TCP connection** **的开销** 

- **Exchange**：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到 queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (multicast)

- **Queue**：消息最终被送到这里等待 consumer 取走

- **Binding**：exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据

### RabbitMQ 有哪些重要的组件？它们有什么作用？

- ConnectionFactory（连接管理器）：应用程序与 RabbitMQ 之间建立连接的管理器，程序代码中使用；
- Channel（信道）：消息推送使用的通道；
- Exchange（交换器）：用于接受、分配消息；
- Queue（队列）：用于存储生产者的消息；
- RoutingKey（路由键）：用于把生成者的数据分配到交换器上；
- BindingKey（绑定键）：用于把交换器的消息绑定到队列上。

![动图](https://pic3.zhimg.com/v2-0d58e30272d79d7148f4093ca0dd7cea_b.webp)

## 2.RabbitMQ工作模式

### I.simple模式（最简单的收发模式）

![img](https://img-blog.csdnimg.cn/20210428140230123.png)

- 消息产生消息，将消息放入队列


- 消息的消费者(consumer) 监听 消息队列,如果队列中有消息,就消费掉,消息被拿走后,自动从队列中删除(隐患 消息可能没有被消费者正确处理,已经从队列中消失了,造成消息的丢失，这里可以设置成手动的ack,但如果设置成手动ack，处理完后要及时发送ack消息给队列，否则会造成内存溢出)。

### II.work工作模式(资源的竞争)

![img](https://img-blog.csdnimg.cn/20210428140247268.png)

- 消息生产者将消息放入队列消费者可以有多个,消费者1,消费者2同时监听同一个队列,消息被消费。


- C1 C2共同争抢当前的消息队列内容,谁先拿到谁负责消费消息(隐患：高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronized) 保证一条消息只能被一个消费者使用)。

### III.publish/subscribe发布订阅(共享资源)

![img](https://img-blog.csdnimg.cn/20210428140303224.png)

- 每个消费者监听自己的队列；
- 生产者将消息发给broker，由交换机将消息转发到绑定此交换机的每个队列，每个绑定交换机的队列都将接收到消息

### IV.routing路由模式

![img](https://img-blog.csdnimg.cn/2021042814031782.png)

- 消息生产者将消息发送给交换机按照路由判断,路由是字符串(info) 当前产生的消息携带路由字符(对象的方法),交换机根据路由的key,只能匹配上路由key对应的消息队列,对应的消费者才能消费消息；


- 根据业务功能定义路由字符串**(routingKey)**；


- 从系统的代码逻辑中获取对应的功能字符串,将消息任务扔到对应的队列中。


- 业务场景:error通知;EXCEPTION;错误通知的功能;传统意义的错误通知;客户通知;利用key路由,可以将程序中的错误封装成消息传入到消息队列中,开发者可以自定义消费者,实时接收错误

## **3.**消息应答

### I.概念：

消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉了，会发生什么情况。RabbitMQ 一旦向消费者传递了一条消息，便立即将该消息标记为删除。在这种情况下，突然有个消费者挂掉了，我们将丢失正在处理的消息。以及后续发送给该消费这的消息，因为它无法接收到。为了保证消息在发送过程中不丢失，rabbitmq 引入消息应答机制，消息应答就是:**消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。** 

### **II.自动应答** 

**消息处理快但消息丢失的可能性大**

**消息发送后立即被认为已经传送成功**，这种模式需要在**高吞吐量和数据传输安全性方面做权衡**,因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢失了,当然另一方面这种模式消费者那边可以传递过载的消息，**没有对传递的消息数量进行限制**，当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使得内存耗尽，最终这些消费者线程被操作系统杀死，所以这种模式**仅适用在消费者可以高效并以某种速率能够处理这些消息的情况下使用。**

### **III.手动消息应答&方法**

如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息未发送 ACK 确认，**RabbitMQ 将了解到消息未完全处理，并将对其重新排队。**如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，**即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。**

**A**.Channel.basicAck(用于肯定确认)：RabbitMQ 已知道该消息并且成功的处理消息，可以将其丢弃了

**B**.Channel.basicNack(用于否定确认) 

**C**.Channel.basicReject(用于否定确认) ：与 Channel.basicNack 相比少一个参数，不处理该消息了直接拒绝，可以将其丢弃了

![image-20221118161153581](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118161153581.png)

## 4.RabbitMQ 持久化

### **I.概念** 

**消息的应答机制保证了消费者处理消息时消费者服务崩溃导致的消息丢失问题**，但是**如何保障当 RabbitMQ 服务停掉以后消息生产者发送过来的消息不丢失**。默认情况下 RabbitMQ 退出或由于某种原因崩溃时，它忽视队列和消息，除非告知它不要这样做。确保消息不会丢失需要做两件事：**我们需要将队列和消息都标记为持久化**。

### II.消息实现持久化 

![image-20221118161613473](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118161613473.png)

要想让消息实现持久化需要在消息生产者修改代码，MessageProperties.PERSISTENT_TEXT_PLAIN 添加这个属性。将消息标记为持久化并不能完全保证不会丢失消息。**尽管它告诉 RabbitMQ 将消息保存到磁盘，但是这里依然存在当消息刚准备存储在磁盘的时候 但是还没有存储完，消息还在缓存的一个间隔点。**此时并没有真正写入磁盘。持久性保证并不强，但是对于我们的简单任务队列而言，这已经绰绰有余了。如果需要更强有力的持久化策略

## 5.Exchanges(交换机)

### I.概念：

RabbitMQ 消息传递模型的核心思想是: **生产者生产的消息从不会直接发送到队列**。实际上，通常生产者甚至都不知道这些消息传递传递到了哪些队列中

相反，**生产者只能将消息发送到交换机(exchange)**，交换机工作的内容非常简单，一方面它接收来自生产者的消息，另一方面将它们推入队列。交换机必须确切知道如何处理收到的消息。是应该把这些消息放到特定队列还是说把他们到许多队列中还是说应该丢弃它们。这就的由交换机的类型来决定。

### II.Fanout交换机

Fanout 这种类型非常简单。正如从名称中猜到的那样，它是将接收到的所有消息**广播**到它知道的所有队列中。

![image-20221118162217347](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118162217347.png)

将队列绑定到该交换机上，让两个消费者监听分别监听这两个队列，当消费者发送消息时，这两个队列都会收到，这两个消费者消费的是同一个消息。

### III.Direct exchange

Fanout 这种交换类型并不能给我们带来很大的灵活性-它只能进行无意识的广播，在这里我们将使用 direct 这种类型来进行替换，这种类型的工作方式是，**消息只去到它绑定的routingKey 队列中去**

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118162518215.png" alt="image-20221118162518215" style="zoom:67%;" />

我们可以看到 X 绑定了两个队列，绑定类型是 direct。队列 Q1 绑定键为 orange，队列 Q2 绑定键有两个:一个绑定键为 black，另一个绑定键为 green

在这种绑定情况下，生产者发布消息到 exchange 上，**要指定routingKey**，**指定键为 orange 的消息会被发布到队列Q1**。绑定键为 blackgreen 和的消息会被发布到队列 Q2，其他消息类型的消息将被丢弃。

### IV.Topics交换机

发送到类型是 topic 交换机的消息的 routing_key 不能随意写，必须满足一定的要求，它**必须是一个单词列表，以点号分隔开**。这些单词可以是任意单词，比如说："stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit".这种类型的。当然这个单词列表最多不能超过 255 个字节。在这个规则列表中，其中有两个替换符是大家需要注意的

- **星号（*）可以代替一个单词**
- **井号（#）可以替代零个或多个单词**

![image-20221118163140095](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118163140095.png)

上图是一个队列绑定关系图，我们来看看他们之间数据接收情况是怎么样的：

quick.orange.rabbit 被队列 Q1Q2 接收到

lazy.orange.elephant 被队列 Q1Q2 接收到

当队列绑定关系是下列这种情况时需要引起注意：

**当一个队列绑定键是#,那么这个队列将接收所有数据，就有点像 fanout 了**

**如果队列绑定键当中没有#和*出现，那么该队列绑定类型就是** **direct** **了**

## 6.死信队列

### **I.死信的概念**

死信，顾名思义就是无法被消费的消息，字面意思可以这样理解，一般来说，producer 将消息投递到 broker 或者直接到 queue 里了，consumer 从 queue 取出消息进行消费，但某些时候由于特定的**原因导致** **queue** **中的某些消息无法被消费**，这样的**消息如果没有后续的处理，就变成了死信，有死信自然就有了死信队列。**

**应用场景：**

为了保证订单业务的消息数据不丢失，需要使用到 RabbitMQ 的死信队列机制，当消息消费发生异常时，将消息投入死信队列中用户在商城

用户下单成功并点击去支付后在指定时间未支付时自动失效

### **II.死信的来源**

- 消息 TTL 过期
- 队列达到最大长度(队列满了，无法再添加数据到 mq 中)
- 消息被拒绝(basic.reject 或 basic.nack)并且 requeue=false

### III.代码理解架构

![image-20221118164419698](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118164419698.png)

 

## **7.延迟队列**

### I.概念

延时队列,队列内部是有序的，最重要的特性就体现在它的延时属性上，延时队列中的元素是**希望在指定时间到了以后或之前取出和处理**，简单来说，**延时队列就是用来存放需要在指定时间被处理的元素的队列**

### **II.延迟队列使用场景**

1. 订单在十分钟之内未支付则自动取消
2. 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。
3. 用户注册成功后，如果三天内没有登陆则进行短信提醒。
4. 用户发起退款，如果三天内没有得到处理则通知相关运营人员。
5. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议

这些场景都有一个特点，**需要在某个事件发生之后或者之前的指定时间点完成某一项任务**，如：发生订单生成事件，在十分钟之后检查该订单支付状态，然后将未支付的订单进行关闭；看起来似乎使用定时任务，一直轮询数据，每秒查一次，取出需要被处理的数据，然后处理不就完事了吗？如果数据量比较少，确实可以这样做，比如：对于“如果账单一周内未支付则进行自动结算”这样的需求，如果对于时间不是严格限制，而是宽松意义上的一周，那么每天晚上跑个定时任务检查一下所有未支付的账单，确实也是一个可行的方案。但**对于数据量比较大，并且时效性较强的场景**，如：**“订单十分钟内未支付则关闭“，短期内未支付的订单数据可能会有很多**，活动期间甚至会达到百万甚至千万级别，对这么庞大的数据量仍旧使用轮询的方式显然是不可取的，很可能在一秒内无法完成所有订单的检查，同时会给数据库带来很大压力，无法满足业务要求而且性能低下。

![image-20221118165522325](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118165522325.png)

## 8.TTL 是什么呢？

TTL 是 RabbitMQ 中一个消息或者队列的属性，表明**一条消息或者该队列中的所有消息的最大存活时间**，单位是毫秒。换句话说，如果一条消息设置了 TTL 属性或者进入了设置 TTL 属性的队列，那么这条消息如果在 TTL 设置的时间内没有被消费，则会成为"死信"。如果同时配置了队列的 TTL 和消息的TTL，那么较小的那个值将会被使用，有两种方式设置 TTL。

**二者区别：**

- 队列TTL：如果设置了队列的 TTL 属性，那么一旦消息过期，就会被队列丢弃(如果配置了死信队列被丢到死信队列中)
- 消息TTL：如果消息设置了TTL，消息即使过期，也不一定会被马上丢弃，因为**消息是否过期是在即将投递到消费者之前判定的**，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间；比如先发了一条TTL为10s的消息A，又发了一条TTL为3s的消息B，由于队列FIFO的特性为，**RabbitMQ 只会检查第一个消息是否过期**，之后才会检查靠后的消息，所以消息B提前过期也得不到及时处理。

## 9.基于死信队列的延时队列实现

**根据TTL和死信队列这两个重要元素就可以实现延迟队列。**

![image-20221118171421781](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118171421781.png)

P：消息生产者

QA：设置了TTL=10s的普通队列

QB：设置了TTL=40s的普通队列

QD：死信队列，用于接收过期消息

XA：和QA绑定的routingKey

XB：和QB绑定的routingKey

YD：死信交换机，将过期消息转发给死信队列

C：消息消费者

**核心思想：设置了TTL的队列相当于中转站，只接收消息并等待过期时间，当消息过期时发送到死信队列进行处理。**

## 10.基于延时队列插件的延时队列

上文中提到的问题，确实是一个问题，如果不能实现在消息粒度上的 TTL，并使其在设置的 TTL 时间及时死亡，就无法设计成一个通用的延时队列。那如何解决呢，接下来我们就去解决该问题。

在官网上下载 https://www.rabbitmq.com/community-plugins.html，下载**rabbitmq_delayed_message_exchange** 插件，然后解压放置到 RabbitMQ 的插件目录。进入 RabbitMQ 的安装目录下的 plgins 目录，执行下面命令让该插件生效，然后重启 RabbitMQ/usr/lib/rabbitmq/lib/rabbitmq_server-3.8.8/plugins，rabbitmq-plugins enable rabbitmq_delayed_message_exchange

![image-20221118191520490](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118191520490.png)

![image-20221118191427210](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118191427210.png)

在这里新增了一个队列 delayed.queue,一个自定义交换机 delayed.exchange，绑定关系如下：

![image-20221118191549436](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118191549436.png)



在我们自定义的交换机中，这是一种新的交换类型，该类型消息支持延迟投递机制 消息传递后并不会立即投递到目标队列中，而是存储在 mnesia(一个分布式数据系统)表中，当达到投递时间时，才投递到目标队列中

**配置文件类代码**

```java
@Configuration
public class DelayedQueueConfig {
     public static final String DELAYED_QUEUE_NAME = "delayed.queue";
     public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
     public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";
     @Bean
     public Queue delayedQueue() {
     	return new Queue(DELAYED_QUEUE_NAME);
     }
     //自定义交换机 我们在这里定义的是一个延迟交换机
     @Bean
     public CustomExchange delayedExchange() {
         Map<String, Object> args = new HashMap<>();
         //自定义交换机的类型
         args.put("x-delayed-type", "direct");
         return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false, 
        args);
     }
     @Bean
    public Binding bindingDelayedQueue(@Qualifier("delayedQueue") Queue queue, @Qualifier("delayedExchange") CustomExchange delayedExchange) {
     	return BindingBuilder.bind(queue).to(delayedExchange).with(DELAYED_ROUTING_KEY).noargs();
     }
 }
```

**消息生产者代码** 

```java
public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";

public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";

@GetMapping("sendDelayMsg/{message}/{delayTime}")
public void sendMsg(@PathVariable String message,@PathVariable Integer delayTime) {
	 rabbitTemplate.convertAndSend(DELAYED_EXCHANGE_NAME, DELAYED_ROUTING_KEY, message, correlationData ->{ 
    correlationData.getMessageProperties().setDelay(delayTime);
 return correlationData;
                   });
 log.info(" 当 前 时 间 ： {}, 发送一条延迟 {} 毫秒的信息给队列 delayed.queue:{}", new Date(), delayTime, message);
}
```

**消息消费者代码** 

```java
public static final String DELAYED_QUEUE_NAME = "delayed.queue";
@RabbitListener(queues = DELAYED_QUEUE_NAME)
public void receiveDelayedQueue(Message message){
 String msg = new String(message.getBody());
 log.info("当前时间：{},收到延时队列的消息：{}", new Date().toString(), msg);
}
```

http://localhost:8080/ttl/sendDelayMsg/come on baby1/20000

http://localhost:8080/ttl/sendDelayMsg/come on baby2/2000

![image-20221118192149534](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118192149534.png)

**第二个消息被先消费掉了，符合预期**

## 11.回退消息

**Mandatory 参数** 

**在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息**，**如果发现该消息不可路由，那么消息会被直接丢弃，此时生产者是不知道消息被丢弃这个事件的**。那么如何让无法被路由的消息帮我想办法处理一下？最起码通知我一声，我好自己处理啊。通过设置 **`mandatory`** 参数可以在当消息传递过程中不可达目的地时将消息**返回给生产者**。**用一个回调接口来接收回退消息**

```java

@Configuration//将MyCallbackConfig实例化
@Slf4j
public class MyCallbackConfig implements RabbitTemplate.ConfirmCallback,RabbitTemplate.ReturnsCallback{
    @Autowired//注入RabbitmqTempla
    private RabbitTemplate rabbitTemplate;
    @PostConstruct
    public void init(){
        //将当前配置类注入到ConfirmCallback接口，在调用接口时才能调到该实现类
        rabbitTemplate.setConfirmCallback(this);
        /**
         * true：
         * 交换机无法将消息进行路由时，会将该消息返回给生产者
         * false：
         * 如果发现消息无法进行路由，则直接丢弃
         */
 		rabbitTemplate.setMandatory(true);
        //设置回退消息交给谁处理
        rabbitTemplate.setReturnsCallback(this);
    }
    //交换机确认回调方法
    /**
     * 发消息，交换机接收到后回调
     * CorrelationData：保存回调消息的id和相关信息（在消息发送方设置）
     * ack：交换机是否接受到消息
     * cause：引起失败的原因
     */
    //只能处理交换机与生产者之间的消息问题，若消息是在交换机到队列的过程中丢失的，无法收到异常回调消息
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        //取消息的id值
        String id = correlationData != null ? correlationData.getId() : "";
        if(ack) {
            log.info("交换机已经收到了消息，id为{}",id);
        }else {
            log.info("交换机还未接收到id为{}的消息，原因为{}",id,cause);
        }
    }

    //回退方法，处理消息在交换机和队列间的传输问题
    //只有不可达目的地的时候才会回退
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        log.info("消息:{}被服务器退回，退回原因:{}, 交换机是:{}, 路由 key:{}",new 	String(message.getBody()),replyText, exchange, routingKey);
    }

    @Override
    public void returnedMessage(ReturnedMessage returnedMessage) {

    }
}

```



## 12.备份交换机

有了 mandatory 参数和回退消息，我们获得了对无法投递消息的感知能力，有机会在生产者的消息无法被投递时发现并处理。但有时候，我们并不知道该如何处理这些无法路由的消息，最多打个日志，然后触发报警，再来手动处理。而通过日志来处理这些无法路由的消息是很不优雅的做法，特别是当生产者所在的服务有多台机器的时候，手动复制日志会更加麻烦而且容易出错。而且设置 mandatory 参数会增加生产者的复杂性，需要添加处理这些被退回的消息的逻辑。如果既不想丢失消息，又不想增加生产者的复杂性，该怎么做呢？前面在设置死信队列的文章中，我们提到，可以为队列设置死信交换机来存储那些处理失败的消息，可是这些不可路由消息根本没有机会进入到队列，因此无法使用死信队列来保存消息。在 RabbitMQ 中，有一种备份交换机的机制存在，可以很好的应对这个问题。什么是备份交换机呢？备份交换机可以理解为 RabbitMQ 中交换机的“备胎”，当我们为某一个交换机声明一个对应的备份交换机时，就是为它创建一个备胎，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout ，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进入这个队列了。当然，我们还可以建立一个报警队列，用独立的消费者来进行监测和报警。

![image-20221118195834627](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221118195834627.png)

mandatory 参数与备份交换机可以一起使用的时候，如果两者同时开启，消息究竟何去何从？谁优先级高，答案是**备份交换机优先级高**。

# 三、其它知识点

## 1.幂等性如何保证？

### I.概念：

**用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。**举个最简单的例子，那就是支付，用户购买商品后支付，支付扣款成功，但是返回结果的时候网络异常，此时钱已经扣了，用户再次点击按钮，此时会进行第二次扣款，返回结果成功，用户查询余额发现多扣钱了，流水记录也变成了两条。在以前的单应用系统中，我们只需要把数据操作放入事务中即可，发生错误立即回滚，但是再响应客户端的时候也有可能出现网络中断或者异常等等

### II.消息重复消费：

消费者在消费 MQ 中的消息时，MQ 已把消息发送给消费者，**消费者在给 MQ 返回 ack 时网络中断**，故 MQ 未收到确认信息，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息

**解决思路** 

MQ 消费者的幂等性的解决一般**使用全局 ID 或者写个唯一标识**比如时间戳 或者 UUID 或者订单消费者消费 MQ 中的消息也可利用 MQ 的该 id 来判断，或者可按自己的规则生成一个全局唯一 id，每次消费消息时用该 id 先判断该消息是否已消费过。

### III.消费端的幂等性保障 

在海量订单生成的业务高峰期，生产端有可能就会重复发生了消息，这时候消费端就要实现幂等性，这就意味着我们的消息永远不会被消费多次，即使我们收到了一样的消息。业界主流的幂等性有两种操作:

- 唯一 ID+指纹码机制，利用数据库主键去重, 
- **利用 redis 的原子性去实现**

<img src="https://img-blog.csdnimg.cn/img_convert/ceb02a75ad4e4c4ae828ee3e161bd328.png" alt="img" style="zoom:80%;" />

## 2.如何保证高可用的？RabbitMQ的集群？

RabbitMQ是比较有代表性的，因为是基于主从（非分布式）做高可用性的，我们就以RabbitMQ为例子讲解第一种MQ的高可用性怎么实现。RabbitMQ有三种模式：**单机模式、普通集群模式、镜像集群模式。**

- **单机模式：**

就是Demo级别的，一般就是你本地启动了玩玩，没人生产用单机模式

- **普通集群模式：**

意思就是在多台机器上启动多个RabbitMQ实例，每个机器启动一个。

你创建的queue，只会放在一个RabbitMQ实例上，但是每个实例都同步queue的元数据（元数据可以认为是queue的一些配置信息，通过元数据，可以找到queue所在实例）。

你消费的时候，实际上如果连接到了另外一个实例，那么**那个实例会从queue所在实例上拉取数据过来。**这方案主要是提高吞吐量的，就是说让集群中多个节点来服务某个queue的读写操作。

<img src="https://img-blog.csdnimg.cn/20210529114025908.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MjAyODcz,size_16,color_FFFFFF,t_70" alt="img" style="zoom: 50%;" />

- **镜像集群模式：**

如果 RabbitMQ 集群中只有一个 Broker 节点，那么该节点的失效将导致整体服务的临时性不可用，并且也可能会导致消息的丢失。可以将所有消息都设置为持久化，并且对应队列的durable属性也设置为true，但是这样仍然无法避免由于缓存导致的问题：因为消息在发送之后和被写入磁盘井执行刷盘动作之间存在一个短暂却会产生问题的时间窗。通过 publisherconfirm 机制能够确保客户端知道哪些消息己经存入磁盘，尽管如此，一般不希望遇到因单点故障导致的服务不可用。

这种模式，才是所谓的RabbitMQ的高可用模式。跟普通集群模式不一样的是，在镜像集群模式下，你创建的queue，无论元数据还是 queue 里的消息都会存在于多个实例上，就是说，**每个RabbitMQ节点都有这个queue的一个完整镜像**，包含queue的全部数据的意思。然后每次你写消息到queue的时候，都会自动把消息同步到多个实例的queue上。

RabbitMQ有很好的管理控制台，就是在后台新增一个策略，这个策略是镜像集群模式的策略，指定的时候是可以要求数据同步到所有节点的，也可以要求同步到指定数量的节点，再次创建queue的时候，应用这个策略，就会自动将数据同步到其他的节点上去了。

这样的好处在于，你任何一个机器宕机了，没事儿，其它机器（节点）还包含了这个queue的完整数据，别的consumer都可以到其它节点上去消费数据。坏处在于，第一，这个性能开销也太大了吧，消息需要同步到所有机器上，导致网络带宽压力和消耗很重！RabbitMQ一个queue的数据都是放在一个节点里的，镜像集群下，也是每个节点都放这个queue的完整数据。
<img src="https://img-blog.csdnimg.cn/20210529114125753.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MjAyODcz,size_16,color_FFFFFF,t_70" alt="img" style="zoom:50%;" />

## 3.为什么不应该对所有的message都使用持久化机制？

答：

首先，必然导致性能的下降，因为写磁盘比写RAM慢的多，message的吞吐量可能有10倍的差距。

其次，message的持久化机制用在RabbitMQ的内置cluster方案时会出现“坑爹”问题。矛盾点在于，若message设置了persistent属性，但queue未设置durable属性，那么当该queue的owner node出现异常后，在未重建该queue前，发往该queue 的message将被 blackholed；若 message 设置了 persistent属性，同时queue也设置了durable属性，那么当queue的owner node异常且无法重启的情况下，则该queue无法在其他node上重建，只能等待其owner node重启后，才能恢复该 queue的使用，而在这段时间内发送给该queue的message将被 blackholed 。

所以，是否要对message进行持久化，需要综合考虑性能需要，以及可能遇到的问题。若想达到100,000 条/秒以上的消息吞吐量（单RabbitMQ服务器），则要么使用其他的方式来确保message的可靠delivery ，要么使用非常快速的存储系统以支持全持久化（例如使用**SSD**）。

另外一种处理原则是：仅对关键消息作持久化处理（根据业务重要程度），且应该保证关键消息的量不会导致性能瓶颈。

## 4.如何保证消息不丢失？

![image-20221119145303997](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221119145303997.png)

### I.生产者弄丢了数据

生产者将数据发送到 RabbitMQ 的时候，可能数据就在半路给搞丢了，因为网络问题啥的，都有可能。此时可以选择用 **RabbitMQ 提供的事务功能，就是生产者发送数据之前开启 RabbitMQ 事务channel.txSelect** ，然后发送消息，如果消息没有成功被 RabbitMQ 接收到，那么生产者会收到异常报错，此时就可以回滚事务 channel.txRollback ，然后重试发送消息；如果收到了消息，那么可以提交事务 channel.txCommit 。

```java
// 开启事务 
channel.txSelect try {
    // 这里发送消息 
} catch (Exception e) {
    channel.txRollback
        // 这里再次重发这条消息
}
// 提交事务 
channel.txCommit
```

但是问题是，RabbitMQ 事务机制（同步）一搞，基本上吞吐量会下来，因为太耗性能

所以一般来说，如果你要确保说写 RabbitMQ 的消息别丢，可以开启 confirm 模式，在生产者那里设置开启 confirm 模式之后，你每次写的消息都会分配一个唯一的 id，然后如果写入了 RabbitMQ 中，**RabbitMQ 会给你回传一个 ack 消息**，告诉你说这个消息 ok 了。如果**RabbitMQ 没能处理这个消息，会回调你的一个 nack 接口，告诉你这个消息接收失败，你可以重试。**而且你可以结合这个机制自己在内存里维护每个消息 id 的状态，如果超过一定时间还没接收到这个消息的回调，那么你可以重发。事务机制和 confirm 机制最大的不同在于，事务机制是同步的，你提交一个事务之后会阻 塞在那儿，但是 confirm 机制是异步的，你发送个消息之后就可以发送下一个消息，然后那个消息 RabbitMQ 接收了之后会异步回调你的一个接口通知你这个消息接收到了。**所以一般在生产者这块避免数据丢失，都是用 confirm 机制的。**

### II.RabbitMQ弄丢了数据

就是 RabbitMQ 自己弄丢了数据，这个你必须开启 **RabbitMQ** 的持久化，就是消息写入之后会持久化到磁盘，哪怕是 RabbitMQ 自己挂了，恢复之后会自动读取之前存储的数据，一般数据不会丢。除非极其罕见的是，RabbitMQ 还没持久化，自己就挂了，可能导致少量数据丢失，但是这个概率较小。

- 创建 queue 的时候将其设置为持久化

这样就可以保证 RabbitMQ **持久化 queue 的元数据**，但是它是不会持久化 queue 里的消息的。

- 第二个是发送消息的时候将消息的 deliveryMode 设置为 2(消息持久化)

就是将消息设置为持久化的，此时 RabbitMQ 就会将消息持久化到磁盘上去。必须要同时设置这两个持久化才行，RabbitMQ 哪怕是挂了，再次重启，也会从磁盘上重启恢复queue，恢复这个 queue 里的数据。

注意：哪怕是你给 RabbitMQ 开启了持久化机制，也有一种可能，就是这个消息写到了RabbitMQ 中，但是还没来得及持久化到磁盘上，结果不巧，此时 RabbitMQ 挂了，就会导致内存里的一点点数据丢失。所以，持久化可以跟生产者那边的 confirm 机制配合起来，**只有消息被持久化到磁盘之后，才会通知生产者 ack 了，所以哪怕是在持久化到磁盘之前，RabbitMQ 挂了，数据丢了，生产者收不到 ack ，你也是可以自己重发的**

### III.消费者弄丢了数据

RabbitMQ 如果丢失了数据，主要是因为消费者消费消息的时候，刚得到消息，还没处理，结果消费者进程挂了，比如重启了，那么就尴尬了，RabbitMQ 认为该消息被消费了，这数据就丢了。这个时候得用 RabbitMQ **提供的 ack 机制（手动应答）**，简单来说，就是你**必须关闭 RabbitMQ 的自动ack** ，可以通过一个 api 来调用就行，**然后在消费者代码里判断消息是否真正被消费，然后手动ACK。**这样的话，如果你还没处理完，不就没有 ack 了？那 RabbitMQ 就认为你还没处理完，这个时候 RabbitMQ 会把这个消费分配给别的consumer 去处理，消息是不会丢的。

<img src="C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221119151413053.png" alt="image-20221119151413053" style="zoom:67%;" />

## 5.如何保证消息的顺序性？

对于 RabbitMQ 来说，导致上面顺序错乱的原因通常是消费者是集群部署，不同的消费者消费到了同一订单的不同的消息，如消费者 A 执行了增加，消费者 B 执行了修改，消费者 C 执行了删除，但是消费者 C 执行比消费者 B 快，消费者 B 又比消费者 A 快，就会导致消费 binlog 执行到数据库的时候顺序错乱，本该顺序是增加、修改、删除，变成了删除、修改、增加。

![img](https://static001.geekbang.org/infoq/08/08c3aece9a61a98b2f4b398acaaf7648.png)

顺序错乱要么是**由于多个消费者消费到了同一个订单号的不同消息**，要么是**由于同一个订单号的消息分发到了 MQ 中的不同机器中。**不同的消息队列保证消息顺序性的方案也各不相同。

解决这个问题，我们可以给 RabbitMQ **创建多个 queue**，每个消费者固定消费一个 queue 的消息，生产者发送消息的时候，**同一个订单号的消息发送到同一个 queue 中**，由于同一个 queue 的消息是一定会保证有序的，那么同一个订单号的消息就只会被一个消费者顺序消费，从而保证了消息的顺序性。

如下图是 RabbitMQ 保证消息顺序性的方案：

![img](https://static001.geekbang.org/infoq/48/48cd8e82cc15e0287c952450ad7e41d0.png)

## 6.如果让你写一个消息队列，该如何进行架构设计？说一下你的思路。

- 首先这个 mq 得支持可伸缩性吧，就是需要的时候快速扩容，就可以增加吞吐量和容量
- 其次你得考虑一下这个 mq 的数据要不要落地磁盘
- 其次你考虑一下你的 mq 的可用性，可参考RabbitMQ的镜像集群模式来保障高可用
- 能不能支持数据 0 丢失啊，消息确认应答等。