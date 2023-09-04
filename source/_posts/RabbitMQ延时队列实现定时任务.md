---
title: RabbitMQ延时队列实现定时任务
date: 2022-03-06 15:51:23
tags: [Java,随笔,微服务,SpringCloud]
categories: 随笔
---

# RabbitMQ延时队列（实现定时任务）

比如有一个未付款订单，超过一定时间后，系统自动取消订单并释放占有的商品。

可以使用spring 的 schedule 定时任务轮询数据库，但是使用这种方式会极其消耗系统内存、增加数据库压力并且存在较大的时间误差

![时间误差](https://qiniu-note-image.ctong.top/note/images/202203030930376.png)

以上问题可以使用 RabbitMQ 的消息TTL和死信 Exchange 结合，下单后，如果30分钟未支付就会关闭订单和解锁库存，不需要全表扫描

![延时队列](https://qiniu-note-image.ctong.top/note/images/202203030931785.png)

## 消息的TTL（Time To Live）

- 消息的TTL就是消息的存活时间
- RabbitMQ 可以对队列和消息分别设置TTL
  - 对队列设置就是队列没有消费者连接着的保留时间，也可以对每一个单独的消息做单独的设置。超过了这个时间，我们认为这个消息就死了，称之为死信。
  - 如果队列设置了，消息也设置了，那么会取最小的。所以一个消息如果被路由到不同的队列中，这个消息死亡的时间有可能不一样（不同队列设置）。可以通过设置消息的 expiration 字段或者 x-message-ttl 属性来设置时间，两者都是一样的效果。

## Dead Letter Exchanges（DLX）

- 一个消息在满足如下条件下，会进入**死信路由**（路由不是队列），一个路由可以对应多个队列。
  - 一个消息被 Cnsumer 拒收了，并且 reject 方法的参数里 requeue 是false。也就是说不会被再次放在队列里，被其他消费者使用。（basic.reject/basic.nack）requeue=false
  - 上面的消息的 TTL 到了，消息过期了
  - 队列的长长度限制满了。排在前面的消息被丢弃或者仍在死信路由上
- Dead Letter Exchange 其实就是一种普通的 Exchange，和创建其它 exchange 没有两样。只是在某一个设置 Dead Letter Exchange 中有消息过期了，会自动触发消息的转发，发送到 Dead Letter Exchange 中去。
- 我们既可以控制消息在一段时间后变成死信，又可以控制变成死信的消息被路由到某一个指定的交换机，结合二者，就可以实现一个延时队列。

可以给队列设置过期时间：

![延时队列实现1](https://qiniu-note-image.ctong.top/note/images/202203031507600.png)

给每个消息设置过期时间

![延时队列实现2](https://qiniu-note-image.ctong.top/note/images/202203031508551.png)

> 死信路由不能有消费者

## 实现

先创建好Queue、Exchange、Binding

使用SpringBoot容器进行创建

```java
@Slf4j
@Configuration
public class AmqpMqConfig {

    /**
     * 延时队列
     * @return Queue
     * @author Clover You
     * @date 2022/3/3 3:20 PM
     */
    @Bean
    public Queue orderDelayQueue() {
        Map<String, Object> args = new HashMap<>(3);
      	// 过期后发到哪个交换机/死信路由
        args.put("x-dead-letter-exchange", "order-event-exchange");
       // 过期后转发到指定死信路由时使用的路由键
        args.put("x-dead-letter-routing-key", "order.release.order");
       // 每个消息的过期时间毫秒为单位
        args.put("x-message-ttl", 60000);
        return new Queue(
            "order.delay.queue",
            true,
            false,
            false,
            args
        );
    }

    /**
     * 死信队列
     * @return Queue
     * @author Clover You
     * @date 2022/3/3 3:26 PM
     */
    @Bean
    public Queue orderReleaseOrderQueue() {
        return new Queue(
            "order.release.order.queue",
            true,
            false,
            false
        );
    }

    /**
     * 死信路由
     * @return Exchange
     * @author Clover You
     * @date 2022/3/3 3:25 PM
     */
    @Bean
    public Exchange orderEventExchange() {
        return new TopicExchange("order-event-exchange", true, false);
    }

    /**
     * 延时队列绑定
     * @return Binding
     * @author Clover You
     * @date 2022/3/3 3:25 PM
     */
    @Bean
    public Binding orderCreateOrderBinding() {
        return new Binding(
            "order.delay.queue",
            Binding.DestinationType.QUEUE,
            "order-event-exchange",
            "order.create.order",
            null
        );
    }

    /**
     * 死信队列绑定
     * @return Binding
     * @author Clover You
     * @date 2022/3/3 3:25 PM
     */
    @Bean
    public Binding orderReleaseOrderBinding() {
        return new Binding(
            "order.release.order.queue",
            Binding.DestinationType.QUEUE,
            "order-event-exchange",
            "order.release.order",
            null
        );
    }
}
```

创建好后监听我们的死信队列，不能监听延时队列，因为延时队列中的消息到了过期时间没人处理后就会被当作死信，按照规则发到死信路由中，再由死信路由以指定路由键发到指定队列，然后队列将消息分发给消费者。

```java
@Component
public class Listener {
  @RabbitListener(queues = "order.release.order.queue")
  public void test(Message message,String str, Channel channel) throws IOException {
    log.info("有订单过期啦： ====>>> {}",str);
    // 如果没开启手动ack则不用写这行
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
  }
}
```

然后再创建一个控制器准备发送消息

```java
@RestController
@RequestMapping("order/order")
public class OrderController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/amqp")
    public String testAmqp() {
        rabbitTemplate.convertAndSend("order-event-exchange","order.create.order", "Hello World");
        return "successful";
    }
}
```

调用 `OrderController.testAmqp()` 就可以看到结果了
