title: RabitMQ基础-学习笔记
categories: JAVA
tags: [JAVA,学习笔记,SpringBoot]
date: 2022-02-25 8:07
---
## 概述

1. 大多应用中，可以通过消息服务中间件来提升系统异步通信、拓展解耦能力。

2. 消息服务中两个重要概念：

   1. 消息代理（message broker）
   2. 目的地（destination）

   当消息发送者发送消息后，将由消息代理接管，消息代理保证消息传递到指定目的地。

3. 消息队列主要有两种形式的目的地

   1. 队列（queue）：点对点消息通信（point-to-point）
   2. 主题（topic）：发布（publish）/订阅（subscribe）消息通信

4. 点对点：

   - 消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移出队列
   - 消息只有唯一的发送者和接收者，但并不是说只能有一个接收者

5. 发布订阅模式

   - 发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息

6. JMS（Java Message Service）Java消息服务：

   - 基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

7. AMQP（Advanced Message Queuing Protocol）

   - 高级消息队列协议，也是一个消息代理的规范，兼容JMS
   - RabbitMQ是AMQP的实现

8. Spring 支持

   - spring-jms 提供了对JMS的支持
   - spring-rabbit 提供了对AMQP的支持
   - 需要ConnectionFactory的实现来连接消息代理
   - 提供Jms Template、Rabbit Template来发送消息
   - @JmsListener（JMS）、@RabbitListener（AMQP）注解在方法上监听消息代理发布的消息
   - @EnableJms、@EnableRabbit开启支持

9. SpringBoot自动配置

   - JmsAutoConfiguration
   - RabbitAutoConfiguratoin

10. 市面的MQ产品

    - ActiveMQ、RabbitMQ、RocketMQ、Kafka

|              | JMS（Java Message Service）                                  | AMQP（Advanced Message Queuing Protocol）                    |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 定义         | Java api                                                     | 网络线级协议                                                 |
| 跨语言       | 否                                                           | 是                                                           |
| 跨平台       | 否                                                           | 是                                                           |
| 支持消息类型 | 1. Peer-2-Peer<br />2. Pub/Sub                               | 1. direct exchange<br />2.fanout exchange<br />3. topic change<br />4.headers exchange<br />5.system exchange<br />后四者本质上与JMS中的Pub/Sub模型没有太大差别，仅是在路由机制上做了更详细的划分。 |
| 支持消息类型 | TextMessage<br />MapMessage<br />BytesMessage<br />StreamMessage<br />ObjectMessage<br />Message（只有消息头和属性） | byte[]<br />（在实际应用中，如果有复杂的消息，可以将消息序列化后发送） |
| 综合评价     | JMS定义了JAVA API层面的标准，在java系统中，多个Client均可以通过JMS进行交互，不需要应用修改代码，但是其对跨平台的支持较差 | AMQP定义了 wire-level层的协议标准，天然具有跨平台、跨语言的特性 |

## RabbitMQ概念

RabbitMQ是一个由erlang开发的AMQP（Advanced Message Queue Protocol）的开源实现。

### 核心概念

#### Message

消息（Message）是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对其它消息的优先权）、delivery-moda（指出该消息可能需要持久性存储）等。

#### Publisher

消息的生产者，也是一个向交换器发布消息的客户端应用程序。

#### Exchange

交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。Exchange有4种类型：direct（默认），fanout，topic，和headers，不同类型的Exchange转发消息的策略有所区别

#### Queue

消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列中。消息一直在队列里面，等待消费者连接到这个队列将其取走。

#### Binding

绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

Exchange和Queue的绑定可以是多对多的关系

#### Connection

网络连接，比如一个TCP连接

#### Channel

信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接。AMQP命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这鞋动作都是通过信道来完成。因为对于操作系统来说建立和销毁TCP都是非常昂贵的开销，所以引入了信道的概念，复用一条TCP连接。

#### Consumer

消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

#### Virtual Host

虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个vhost本质上就是一个mini版的RabbitMQ服务器，拥有自己的队列、交换器、绑定和权限机制。vhost是AMQP概念的基础，必须在连接时指定，RabbitMQ默认的vhost是`/`

#### Broker

表示消息队列服务器实体



![RabbitMQ工作流程](https://qiniu-note-image.ctong.top/note/images/202202212223471.jpg)

## Docker安装RabbitMQ

```
docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management
```

4369,25672（Erlang发现&集群端口）

5672，5671（AMQP端口）

15672（微博管理后台端口，账号密码默认都是：guest）

61613，61614（STOMP协议端口）

1883，8883（MQTT协议端口）

## Exchange 类型

- Exchange分发消息时根据类型的不同分发策略有区别，目前共四种类型 `direct` 、 `fanout` 、 `topic` 、 `headers` 。headers 匹配 AMQP 消息的 header 而不是路由键。header 交换机和 direct 交换机完全一致，但性能差很多

### DireactExchange

如果消息中的路由键（routing-key）于 Binding 中的 binding key 一致，交换器就将消息发送到对应的队列中。路由键需要与队列名（binding key）完全一致。如果一个队列绑定到交换机的binding key 为 “dog”，那么只转发routing key为“dog”的消息。它是完全匹配、单播、点对点模式。

![DireactExchange](https://qiniu-note-image.ctong.top/note/images/202202212223513.png)

### FanoutExchange

每个发到 fanout 类型转换机的消息都会被分发到所有绑定的队列中。fanout 不处理路由键，只是简单的将队列绑定到交换机上。每个发送到交换机的消息都会被转发到与该交换机绑定的所有队列上。fanout 类型转发消息是最快的。

![FanoutExchange](https://qiniu-note-image.ctong.top/note/images/202202212223558.png)

### TopicExchange

topic 交换机通过路由键属性和binding key模式匹配的方式进行分配消息到对应的队列中。将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。他将路由键和绑定键的字符串切分成单词，**这些单词之间需要用 `.` 隔开**，它也是别这两个通配符，分别是 `#` 和 `*` 符号

`#` 表示匹配0个或多个单词

`*` 表示匹配一个单词

![TopicExchange](https://qiniu-note-image.ctong.top/note/images/202202212223704.png)

## 测试Exchange

测试之前先创建四个队列，分别是 `gulimall` 、 `gulimall.emps` 、 `gulimall.news` 、 `gulixueyuan.news`

点击 Queue选项

![1](https://qiniu-note-image.ctong.top/note/images/202202212223728.png)

设置队列信息保存即可

![2](https://qiniu-note-image.ctong.top/note/images/202202212223782.png)

### DireactExchange

创建一个Direct交换机

点击Exchange选项

![1](https://qiniu-note-image.ctong.top/note/images/202202212223830.png)

创建 direct交换机

![2](https://qiniu-note-image.ctong.top/note/images/202202212223877.png)

点击刚刚创建的交换机

![3](https://qiniu-note-image.ctong.top/note/images/202202212223928.png)

新增绑定关系

![4](https://qiniu-note-image.ctong.top/note/images/202202212223980.png)

绑定刚刚创建的所有队列

![截屏2022-02-20 下午10.03.43](https://qiniu-note-image.ctong.top/note/images/202202212223037.png)

#### 测试

选择交换机进行测试

![1](https://qiniu-note-image.ctong.top/note/images/202202212223046.png)

发送测试消息

![2](https://qiniu-note-image.ctong.top/note/images/202202212223110.png)

可以看到我们指定的队列接到两条消息了

![3](https://qiniu-note-image.ctong.top/note/images/202202212223152.png)

单机这个队列，然后获取消息

![4](https://qiniu-note-image.ctong.top/note/images/202202212223189.png)

- Ack Mode 消息处理方式
  - Nack message requeue true 接收消息但不做确认，消息会重新加入队列
  - reject requeue false 拒绝消息，消息会被删除
  - reject requeue false 拒绝消息，消息会重新加入队列
  - Automatic ack 获取消息，应答确认，消息不重新入队，将会从队列中删除

![消息内容](https://qiniu-note-image.ctong.top/note/images/202202212223281.png)

### FanoutExchange

创建一个 fanout 类型的交换机

![ ](https://qiniu-note-image.ctong.top/note/images/202202212223290.png)

进入交换机绑定队列 

![2](https://qiniu-note-image.ctong.top/note/images/202202212223332.png)

绑定刚刚创建的所有队列

![3](https://qiniu-note-image.ctong.top/note/images/202202212223346.png)

#### 测试

当前绑定的队列消息都是空的

![1](https://qiniu-note-image.ctong.top/note/images/202202212223392.png)

测试发送消息

![2](https://qiniu-note-image.ctong.top/note/images/202202212223442.png)

可以发现我们所有的队列都收到了这条消息

![3](https://qiniu-note-image.ctong.top/note/images/202202212223474.png)

先让任意一个队列获取这个消息并回复确认看看

![4](https://qiniu-note-image.ctong.top/note/images/202202212223605.png)

再次查看队列状态，发现只有刚刚处理掉消息的队列删除了这个消息，其它队列还在等待处理。

![5](https://qiniu-note-image.ctong.top/note/images/202202212223607.png)

### TopicExchange

创建Topic交换机

![1](https://qiniu-note-image.ctong.top/note/images/202202212223613.png)

创建成功后就给他绑定队列

使用 `gulimall.#` 表示路由键以 `gulimall` 单词开头的都可以匹配到

![2](https://qiniu-note-image.ctong.top/note/images/202202212223684.png)

根据以下规则绑定队列

![3](https://qiniu-note-image.ctong.top/note/images/202202212223705.png)

#### 测试

发送一个路由键是 `gulimall.news` 的消息

![1](https://qiniu-note-image.ctong.top/note/images/202202212223800.png)

全部队列都收到了，因为`gulimall.#` 表示只要是以`gulimall.`开始的任意字符都能匹配，`*.news` 表示news前面必须要有一个任意单词

![2](https://qiniu-note-image.ctong.top/note/images/202202212223831.png)

再发一个 `hello.news` ，绑定键能匹配上的只有 `gulixueyuan.news` 这个队列

![3](https://qiniu-note-image.ctong.top/note/images/202202211957249.png)

![4](https://qiniu-note-image.ctong.top/note/images/202202211958867.png)

## SpringBoot 整合RabbitMQ

第一步肯定是要引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

引入这个依赖后，在 `RabbitAutoConfiguration` 中，配置了如下几个重要的Bean：

- `RabbitConnectionFactoryBean` 连接工厂
- `RabbitTemplate` 操作组件
- `AmqpAdmin` 高级消息队列管理

引入后需要配置最基础的配置：

```properties
spring.rabbitmq.host=192.168.135.128 # 缺省，默认是localhost

spring.rabbitmq.port=5672 # 缺省，默认是5671、5672

spring.rabbitmq.virtual-host=/ # 缺省，默认/
```

如需进一步配置，可到 `org.springframework.boot.autoconfigure.amqp.RabbitProperties` 类中查看，这是 Rabbit 配置类

### 如何创建 Exchange、Queue、Binding

通过 `AmqpAdmin` 高级消息队列管理组件来进行创建 Exchange、Queue、Binding。

```java
amqpAdmin.declareExchange(Exchange);
```

`declareExchange` 方法需要传一个 exchange

![Exchange集成、实现关系图](https://qiniu-note-image.ctong.top/note/images/202202212134491.png)

根据上图，如果我们需要创建 `TopicExchange` ，我们需要创建一个 `TopicExchange.class` 实例给到 `declareExchange` 方法

实例参数分别是：交换机名称、Durable是否持久化、是否自动删除、交换机参数

```java
TopicExchange topicExchange = new TopicExchange(
  "hello.java.TopicExchange",
  true,
  false,
  new HashMap()
);
amqpAdmin.declareExchange(topicExchange);
```

![代码创建交换机结果图](https://qiniu-note-image.ctong.top/note/images/202202212147783.png)

同样的，可以使用 `declareQueue` 方法创建一个队列

```java
Queue queue = new Queue(
  "hello.java.queue",
  true,
  false,
  false,
  new HashMap<>()
);
amqpAdmin.declareQueue(queue);
```

队列创建参数分别是：队列名称、是否持久化、是否排他（只要有一个连接连上队列，其它连接就不能再连接）、是否是自动删除、队列参数

![代码创建队列结果](https://qiniu-note-image.ctong.top/note/images/202202212215743.png)

交换机和队列创建完毕之后，如需创建绑定关系，可以使用 `declareBinding` 方法创建

```java
Binding binding = new Binding(
    "hello.java.queue",
    Binding.DestinationType.QUEUE,
    "hello.java.TopicExchange",
    "hello.java.queue",
    new HashMap<>()
);
amqpAdmin.declareBinding(binding);
```

`Binding` 类有如下参数：

- `destination` 目的地（交换机/队列名称）
- `destinationType` 绑定类型（和什么东西进行绑定）
  - `EXCHANGE` 指明目的地是一个交换机
  - `QUEUE` 指明目的地是一个队列
- `exchange` 交换机名称
- `routing-key` 路由键
- `argument` 绑定参数 c

代码参数可参考GUI操作内容：

![代码和GUI对应关系](https://qiniu-note-image.ctong.top/note/images/202202212229934.png)

绑定结果

![运行结果](https://qiniu-note-image.ctong.top/note/images/202202212231468.png)

### 发送消息

一切准备就绪后，通过 `RabbitTemplate` 来发送、接收消息

`Message` 需要传递一个字节流来进行初始化实例

`send` 可以将指定`Message` 实例发送到指定交换机

```java
String halo = "hello world";
Message message = new Message(halo.getBytes());
rabbitTemplate.send("hello.java.TopicExchange", "hello.java.queue", message);
```

 ![消息发送结果](https://qiniu-note-image.ctong.top/note/images/202202240851962.png)

![消息内容](https://qiniu-note-image.ctong.top/note/images/202202240852166.png)

同时也可以使用 `convertAndSend` 方法将一个对象序列化为byte发送。

```java
rabbitTemplate.convertAndSend(
    "hello.java.TopicExchange",
    "hello.java.queue",
    new User()
);
```

![convertAndSend 发送消息](https://qiniu-note-image.ctong.top/note/images/202202240938733.png)

可以配置一个 `MessageConverter` 将对象转为JSON格式或其它格式，前提是被序列化对象一定要实现 `Serializeble` 接口。

```java
@Configuration
public class RabbitConfig {

    /**
     * 添加 Rabbit 消息转换器 [Jackson2JsonMessageConverter]
     * @return MessageConverter new Jackson2JsonMessageConverter
     * @author Clover You
     * @date 2022/2/24 9:30 上午
     */
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

![自定义序列化结果](https://qiniu-note-image.ctong.top/note/images/202202240943709.png)

### 监听消息

若想接收 Rabbit 的消息，SpringBoot 封装了一个`@RabbitListener` 注解，可以方便我们接收消息，它可以标注在类和方法上。若需要使用这些注解，必须要开启 `Rabbit` 才能使用

```java
@EnableRabbit
...
public class GulimallOrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(GulimallOrderApplication.class, args);
    }

}
```

使用 `@RabbitListener` 监听指定队列，注意：该注解只在 Spring 容器上生效

```java
@RabbitListener(queues = "hello.java.queue")
public void receiveMessage(Message message, User user) {
  MessageProperties messageProperties = message.getMessageProperties();
  log.info("消息头属性信息：{}", messageProperties);
  log.info("接收到消息：{}", user);// 接收到消息：User(name=Clover You)

}
```

 如果需要获取通道，可以使用 `com.rabbitmq.client.Channel` 参数，例如：

```java
@RabbitListener(queues = "hello.java.queue")
public void receiveMessage(Channel channel);
```

> - Queue 可以有很多人都来监听，只要收到洗消息，队列会将消息删除，而且只能有一个人收到从消息。
> - 只有一个消息处理完成方法结束后才能再接收下一个消息

## RabbitMQ 消息确认机制

- 保证消息不丢失，可靠抵达，可以使用事务消息，性能下降250倍，为此引入确认机制。
- publisher confirmCallback 确认模式
- publisher returnCallback 未投递到 queue 退回模式
- consumer ack 机制

![消息可靠送达制流程图](https://qiniu-note-image.ctong.top/note/images/202202241452531.png)

### 可靠抵达

#### ConfirmCallBack

- 设置 `spring-rabbitmq.publisher-confirm=true`
  - 在创建 `connectionFactory` 的时候设置 PublisherConfirms(true)选项开启confirmCallback。
  - CorrelationData 用来表示当前消息唯一性。
  - 消息只要倍 broker 接收到就会执行 confitmCallback，如果是 cluster(集群) 模式，需要所有 broker 接收到才会调用 confirmCallback
  - 被 broker 接收到只能表示 message 已经到达服务器，并不能保证消息一定会被投递到目标 queue 里。所以需要用到 retrunCallback。

> `spring-rabbitmq.publisher-confirm=true` 配置在新版中被弃用，改用 `spring-rabbitmq.publisher-confirm-type=correlated`

```java
@Slf4j
@Configuration
public class RabbitConfig {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void initRabbitTemplate() {
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            /**
             * Confirmation callback.
             * @param correlationData 当前消息的唯一关联数据
             * @param ack 消息成功还是失败
             * @param cause 失败原因
             */
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                log.info("initRabbitTemplate");
            }
        });
    }
}
```

#### ReturnCallback

- 设置 `spring.rabbitmq.publisher-returns=true` 

- 设置 `spring.rabbitmq.template.mandatory=true` 

  > 如果消息抵达目标队列，那么首先通过异步的方式调用retrunsCallba ck

  - confrim 模式只能保证消息抵达 broker，不能保证消息准确投递到目标 queue 中。在有些业务场景下，我们需要保证消息一定要投递到目标 queue 里，此时就需要用到 return 退回模式。
  - 如果未能投递到目标 queue 里那么将调用 returnCallback，可以记录下详细到投递数据，定期的巡检或者自动纠错都需要这些数据。

```java
@Slf4j
@Configuration
public class RabbitConfig {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void initRabbitTemplate() {
        rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {

            /**
             * 如果消息没有抵达指定的 queue，那么就会触发这个失败回调
             * @param returned 消息和元数据
             */
            @Override
            public void returnedMessage(ReturnedMessage returned) {
                Message message = returned.getMessage();// 投递失败的消息
                String replyText = returned.getReplyText();// RabbitMQ回复信息
                String routingKey = returned.getRoutingKey();// 投递时使用的路由键
                int replyCode = returned.getReplyCode();// RabbitMQ 回复的状态码
                String exchange = returned.getExchange();// 当时消息是发给哪个交换机的

                log.info("message: {}", message);
                log.info("replyText: {}", replyText);
                log.info("routingKey: {}", routingKey);
                log.info("replyCode: {}", replyCode);
                log.info("exchange: {}", exchange);

            }
        });
    }
}
```

```
==================== Returned Callback ====================
message: (Body:'{"name":"Clover You"}' MessageProperties [headers={__TypeId__=top.ctong.gulimall.order.User}, contentType=application/json, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, deliveryTag=0])
replyText: NO_ROUTE
routingKey: hello.java.queue1
replyCode: 312
exchange: hello.java.TopicExchange
```

### 消费端确认

消费端确认机制可以保每个消息都能被正确的处理，此时 broker 才可以删除这个消息。

1. 默认是自动确认，只要消息接收到，客户端会自动回复确认，服务端接收到确认消息后就会自动删除这个消息。
2. 如果需要设置为手动确认，配置需要添加 `spring.rabbitmq.listener.simple.acknowledge-mode=manual`
3. 手动确认模式，只要我们没有明确告诉MQ消息已被处理/没有Ack，消息就一直是 Unacked 状态。即使 Consumer宕机，消息也不会丢失，会重新变为 Ready 状态。

发送4个消息

![检查消息](https://qiniu-note-image.ctong.top/note/images/202202241622172.png)

通过断点阻塞处理方法，所以上图是 Ready=0 ，Unacked=4

![断点](https://qiniu-note-image.ctong.top/note/images/202202241623499.png)

停掉所有断点后检查Queue，状态变为未处理：Ready=4 ，Unacked=0

![取消领取状态](https://qiniu-note-image.ctong.top/note/images/202202241629015.png)

如果需要回复确认，需要使用 Channel 来进行回复

```java
@RabbitListener(queues = "hello.java.queue")
public void receiveMessage(Message message, User user, Channel channel) {
  MessageProperties messageProperties = message.getMessageProperties();
  log.info("消息头属性信息：{}", messageProperties);
  log.info("接收到消息：{}", user);
  try {
    // 第二个参数表示不批量确认
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
  } catch (IOException e) {
    e.printStackTrace();
  }

}
```

同样的，有确认模式也有拒绝模式

```java
@RabbitListener(queues = "hello.java.queue")
public void receiveMessage(Message message, User user, Channel channel) {
  ...
  try {
		// 拒绝消息，第三个参数是表示：true 拒绝后消息重新回到队列，false 拒绝后丢弃消息不重新加入MQ
    channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
  } catch (IOException e) {
    e.printStackTrace();
  }

}
```

#### Ack消息确认机制

- 消费者获取到消息，成功处理，可以回复 Ack 给 Broker
  - basic.ack 用于肯定确认；broker 将移除此消息
  - basic.nack 用于否定确认；可以指定 broker 是否丢弃此消息，可以批量
  - basic.reject 用于是否确认，同上，但不能批量
- 默认：消息被消费者收到，就会从 broker 的 queue 中移除
- 消费者收到消息，默认会自动 ack。但是如果无法确定此消息是否被处理完成，或者成功处理，我们可以开启手动 abc 模式
  - 消息处理成功，`ack()`，接受下一个消息，此消息 broker 就会移除
  - 消息处理失败，`nack()/reject()` ，将重新发送给其他人进行处理，或者容错处理后 ack
  - 消息一直没有调用 `ack/nack` 方法，broker 认为此消息正在被处理，不会投递给其他人。如果此时客户端断开，消息不会被 broker 移除，会投递给别人。

## 如何保证消息可靠性

### 消息丢失

- 消息发送出去，由于网络原因没有抵达服务器
  - 做好容错方法（try-catch），发送消息可能会网络失败，失败后要有重试机制，可记录到数据库，采用定期扫描重发的方式。
  - 做好日志记录，每个消息状态是否都被服务器收到都应该记录
  - 做好定期重发，如果消息没有发送成功，定期去数据库扫描未成功的消息进行重发。
- 消息抵达 Broker，Broker **要将消息写入磁盘（持久化）才算成功**。此时 Broker 尚未持久化完成，宕机。
  - publisher 也必须加入确认回调机制，确认成功的消息，修改数据库消息状态。
- 自动 ACK 状态下，消费者接受到消息，没来的及消费消息如何宕机
  - 一定要开启手动 ACK ，消息成功才能通知 MQ 移除消息，失败或者没来得及处理就 noACK 并重新入队。

### 消息重复

- 消息消费成功，事务已经提交成功了，ack 时，机器宕机导致没有 ack 成功， Broker 的消息重新由 unack 变为 ready，并发送给其他消费者
- 消息消费失败，ack 宕机，消息由 unack 变为 ready，Broker 又重新发送
  - 消费者的业务消费接口应该设计为**具备幂等性**，比如扣库存有工作单的状态标志
  - 使用**防重表** （redis/mysql）， 发送消息每一个都有业务的唯一标识，处理过就不用再处理
  - rabbitMQ 的每一个消息都有 redelivered 字段，可以获取是否是被重新投递过来还是第一次投递过来的。

### 消息积压

- 消费者宕机积压
- 消费者消费能力不足导致消息积压
- 发送者发送流量太大
  - 上线更多的消费者，进行正常消费
  - 上线专门的队列消费服务，将消息先批量取出，记录到数据库，最后离线方式慢慢处理。
