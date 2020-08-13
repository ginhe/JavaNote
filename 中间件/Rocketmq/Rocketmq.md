# 什么是消息队列

它是一个存放消息的容器，当我们需要使用消息的时候可以取出消息供自己使用。

**为什么要用消息队列**

(1) 通过异步处理提高系统性能

如下图所示。在不使用消息队列服务器的时候，用户的请求数据直接写入数据库，在高并发的情况下数据库压力剧增，使得响应速度变；但是在使用消息队列之后，用户的请求数据发送给消息队列之后立即 返回，再由消息队列的消费者进程从消息队列中获取数据，异步写入数据库。由于消息队列服务器处理速度快于数据库（消息队列也比数据库有更好的伸缩性），因此响应速度得到大幅改善。



![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36ecf5ca7c234d78bb9af615004e0295~tplv-k3u1fbpfcp-zoom-1.image)

通过异步处理，将短时间高并发产生的事务消息存储在消息队列中，从而削平高峰期的并发事务。 举例：在电子商务一些秒杀、促销活动中，合理使用消息队列可以有效抵御促销活动刚开始大量订单涌入对系统的冲击。如下图所示：

<img src="https:////upload-images.jianshu.io/upload_images/2509688-f9b9af2c6620e724?imageMogr2/auto-orient/strip|imageView2/2/w/780/format/webp" alt="img" style="zoom:67%;" />

用户请求数据写入消息队列之后就立即返回给用户了，但是请求数据在后续的业务校验、写数据库等操作中可能失败。因此使用消息队列进行异步处理之后，需要适当修改业务流程进行配合，比如用户在提交订单之后，订单数据写入消息队列，不能立即返回用户订单提交成功，需要在消息队列的订单消费者进程真正处理完该订单之后，甚至出库后，再通过电子邮件或短信通知用户订单成功，以免交易纠纷。

(2) 降低系统耦合性

如果模块之间不存在直接调用，那么新增模块或者修改模块就对其他模块影响较小，这样系统的可扩展性无疑更好一些。

![img](https://upload-images.jianshu.io/upload_images/2509688-f3bddbdea97bb30c?imageMogr2/auto-orient/strip|imageView2/2/w/790/format/webp)

由上图可看出消息发送者（生产者）和消息接受者（消费者）之间没有直接耦合，消息发送者将消息发送至分布式消息队列即结束对消息的处理，消息接受者从分布式消息队列获取该消息后进行后续处理，并不需要知道该消息从何而来。对新增业务，只要对该类消息感兴趣，即可订阅该消息，对原有系统和业务没有任何影响，从而实现网站业务的可扩展性设计。

**使用消息队列带来的一些问题**

（1）系统可用性降低：在加入MQ之前，你不用考虑消息丢失或者说MQ挂掉等等的情况，但是，引入MQ之后你就需要去考虑了

（2）系统复杂性提高： 加入MQ之后，你需要保证消息没有被重复消费、处理消息丢失的情况、保证消息传递的顺序性等问题

（3）消息队列带来的异步确实可以提高系统响应速度。但是万一消息的真正消费者并没有正确消费消息怎么办？这样就会导致数据不一致的情况了!



# 模型

所有的broker，producer和consumer都需要到nameserver注册：

（1）每个broker会告诉nameserver它的ip，queue和queue负责的topic。每一个broker至少有一个消息队列queue。

（2）producer注册在nameserver后，如果它向投递以某一topic为主题的消息时，它只需要往对应的broker投递即可。它可以采用负载轮询的方式投递：比如第一次请求投在queue1生成一个message1，第二次是在queue2生成message2。

（3）当consumer启动后，会到nameserver以topic为单位抓取信息。它会向topicA的两个队列建立长连接：

- 如果有消息则直接拉取并执行，执行完后会告知消息已被消费掉，然后消息就会被删掉；
- 如果没有消息则长轮询等待，即保持连接不返回。当有producer向队列投递消息时，consumer就会被唤醒，然后拉取消息。

如果有第二个关注topicA的consumer启动，则它会与queue2建立连接。如果还有第三个关注topicA的consumer，则它是无法与那两个queue建立连接的。因此如果conmser_group的consumer数量大于负责topicA的队列数量，则有的consumer不能消费消息；如果consumer小于队列数量，则有的consumer会消费两个queue的消息。

**如何划分consumer_group**

比如说订单系统属于一个consumer_group，它里面的所有订单模型组成一个consumer_group；另外一个consumer_group属于商品系统。Producer并不知道消息会被哪个consumer_group消费。



<img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL1NraXJvbllvbmcvc2tpcm9uSW1ncy9pbWcvMjAyMDA3MDExMDI4NTQucG5n?x-oss-process=image/format,png" alt="img" style="zoom:50%;" />



**broker主从复制机制**

在机制下会有一个broker2会作为broker1的slave，并且它用于同样的队列，但不会提供服务，仅仅同步消息。一般来说，我们将主broker是否成功生产，是否成功消费作为评定标准。从broker采取异步复制的方式，只要网络延迟和CPU处理足够快，丢消息的概率较小。

当主broker1出现异常，nameserver将broker2竞选为主，并告知对应的producer和consumer之后要从broker做生产消费。



<img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL1NraXJvbllvbmcvc2tpcm9uSW1ncy9pbWcvMjAyMDA3MDExMTE4MjYucG5n?x-oss-process=image/format,png" alt="img" style="zoom:50%;" />





# 参考资料

[消息队列简介](https://www.jianshu.com/p/36a7775b04ec)