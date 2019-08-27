### RabbitMQ 可靠性投递

分析一个场景，如果一个扣款订单，当你扣款完成后发送消息给rabbitmq

```
orderFinsh();//这一步失败了
sendMessage();//但是消息已经发送成功
```

扣款没成功，但是却提示消息了，这就有些不太可靠了。

![1566919159392](C:\Users\99405\AppData\Roaming\Typora\typora-user-images\1566919159392.png)

我们可以对rabbitmq的四种阶段做一个讲解

* 生产者发送消息的时候如何保证消息是否已经被broker接受
  * 提供了事务，原生api有channel.txSelector的事务方式，详情看源码，可提供提交和回滚，但是为了提高效率，提供了确定的模式（confirm）和异步模式
  * 发送消息无法被路由找到，会被broker打回
* exchange如何保证自己一定可以路由到指定的queue
  * 备份交换机，无法被路由的消息会被路由到这里，需要在网页上配置。
  * ![1566919446156](C:\Users\99405\AppData\Roaming\Typora\typora-user-images\1566919446156.png)
  * 当然，你还需要一个队列和他绑定，也需要指定消费者消费，关于后续处理，你可以存到数据库，或者写入日志，又或者通知，关于这些无法被路由的消息如何处理。
* 队列可能存在挂掉的情况
  * 从内存转为持久化，队列的持久化，消息的持久化等，有一个参数可以设置 duration=true
  * 集群
* 消费者如何保证自己可以正确消费到消息
  * 自动的ack basicAck
    * 收到消息的就发送ACK
  * 手动ACK



### 消费方与服务方的应答操作

拿快递追踪作为例子，我发送快递，可以在看到这个快递的物流情况，并且能够发现是否真正被签收了，但是在rabbitmq中，生产者发送消息到交换器，以及消费者去broker消费消息都是独立的操作，并不能相互感应。那么这个情况下如何操作。

* 生产者可以提供一个api，当消费着消费完成的时候，通过这个api修改数据库的标识，来确定是否消费完成。
* 消费者消费完成以后，可不可以返回一个消息给生产者，例如一个ack
  * 如果回执错误，是否考虑超时时间，和重发的方案

### 集群

磁盘节点，内存节点

普通集群

镜像集群

#### 如何动态增加消费者的数量

```java
Bean
    public SimpleMessageListenerContainer messageContainer(ConnectionFactory connectionFactory) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
        //这里
        container.setQueues(getSecondQueue(), getThirdQueue()); //监听的队列
        container.setConcurrentConsumers(1); // 最小消费者数
        container.setMaxConcurrentConsumers(5); //  最大的消费者数量
```

### MQ选型分析

* 使用和管理
* 性能 并发量，吞吐量，消息堆积能力
* 功能
* 可用，持久化，集群