# 一、RabbitMq基础入门

### 1、架构图

![image-20200922164920785](images/image-20200922164920785.png)

- routingkey：交换机用来绑定对应的队列
- Exchange：生产者发送消息并且携带routingkey到交换机，交换机通过routingkey来进行绑定路由到对应的queue中
- coustomer：消费者监听对应的queue

### 2、交换机的种类

- 直接交换机(Direct Exchange)：发送消息时带入的key必须要完全和交换机绑定的key必须要完全匹配
- 主题交换机(Topic Exchange)：routingkey可以使用 (* | # )来进行模糊匹配。*匹配一个单词，#匹配多个单词
- 扇形交换机(Fanout Exchange)：没有routingkey，只要队列订阅了交换机就可以获取到消息（性能更好，不需要routingkey的匹配）

### 3、什么是消息的可靠性投递

> 生产者发送消息给broker之后需要返回一个ack，而broker发送消息给消费者，消费者收到了消息之后要返回一个ack代表其收到
>
> 生产者 ========> broker =========> 消费者

当前架构有性能问题

<img src="../../imgs/image-20200924163425969.png" alt="image-20200924163425969" style="zoom:150%;" />

### 4、ReturnListener消息处理机制

```java
情况一：消息发送出去routingkey写错了没有exchange来接手，消息生产端mandatory设置为true，那么就会调用Return Listener来处理。如果mandatory设置为false（默认），那么mq就会自动删除消息
情况二：消息能够投递到broker的交换机上，但是routingkey路由不到某一个队列上。

```

### 5、Confirm Listener消息确认机制

> 是指生产者将消息投递到了broker上，如果broker接受到了就会给生产者一个应答

