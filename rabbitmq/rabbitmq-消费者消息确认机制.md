# rabbitmq-消费者消息确认机制

## 一、前言

### 1、pom.xml

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

### 2、application.yml

```yml
spring:
  # RabbitMQ配置
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    # 填写自己安装rabbitmq时设置的账号密码，默认账号密码为`guest`
    username: admin
    password: admin
    virtual-host: my_vhost # 填写自己的虚拟机名，对应可查看 `127.0.0.1:15672/#/users` 下Admin中的`Can access virtual hosts`信息
    listener:
      simple:
        # 表示消息确认方式，其有三种配置方式，分别是none、manual和auto；默认auto
        acknowledge-mode: manual
        # 最小的消费者数量
        concurrency: 1
        # 最大的消费者数量
        max-concurrency: 1
        # 指定一个请求能处理多少个消息，如果有事务的话，必须大于等于transaction数量.
        prefetch: 1
        retry:
          # 是否开启重试
          enabled: true
          # 最大重试次数
          max-attempts: 5
          # 第一次和第二次尝试发布或传递消息之间的间隔（单位：毫秒）
          initial-interval: 30000
```

### 3、确认模式

org.springframework.amqp.core.AcknowledgeMode（确认模式）: 用于定义如何确认消息的传递

- `NONE`: 自动不确认模式，RabbitMQ 不会等待应用程序显式地返回确认信息。
- `MANUAL`：手动确认模式，手动挡控制 确认消息 / 拒绝消息 / 重发消息(如果程序一直异常，会出现死循环，导致消息积压)
- `AUTO`: 自动确认模式，在消费消息的时候没有抛出异常，服务端MQ就会认为消息正常消费，从而删除队列中的消息；如果有异常，则会进行补偿，自动重试。

## 二、消息确认机制(ack)

> RabbitMQ默认的消息确认机制是：自动确认的
>
> 队列分配消息给监听消费者时，该消息处于未确认状态，不会被删除；当接收到消费者的确认回复才会将消息移除

现在设置为手动模式

![image-20230614111128705](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230614111128705.png)

```java
@RestController
@RequestMapping("/ack")
public class AckSenderController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/send")
    public void send(@RequestParam(value = "message", required = false, defaultValue = "Hello World") String message) {

        IntStream.range(1,11).forEach(i ->{
            String msg = message + " ..." + i;
            System.out.println(" [ 生产者 ] Sent ==> '" + msg + "'");
            rabbitTemplate.convertAndSend("helloWorldExchange","ack", msg);
        });
    }

    private int count1=1;
    private int count2=1;
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "ackQueue",durable = "true"),
            exchange = @Exchange(value = "helloWorldExchange",type = ExchangeTypes.DIRECT,durable = "true"),
            key = "ack"
    ))
    public void receive(Message message) throws InterruptedException {
        Thread.sleep(200);
        System.out.println(" [ 消费者@1号 ] Received ==> '" + new String(message.getBody()) + "'");
        System.out.println(" [ 消费者@1号 ] 处理消息数：" + count1++);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "ackQueue",durable = "true"),
            exchange = @Exchange(value = "helloWorldExchange",type = ExchangeTypes.DIRECT,durable = "true"),
            key = "ack"
    ))
    public void receive2(Message message) throws InterruptedException {
        Thread.sleep(10000);
        System.out.println(" [ 消费者@2号 ] Received ==> '" + new String(message.getBody()) + "'");
        System.out.println(" [ 消费者@2号 ] 处理消息数：" + count2++);
    }
}
```

> 消费者1号、2号分别拿到一条消息进行消费，但没有确认，处于阻塞状态，所以队列不会移除这两条消息，同时设置了prefetch=1，在消费者未确认之前不会重新推送消息给消费者

![image-20230614111341825](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230614111341825.png)

停止程序，发现2条未确认的消息会回到Ready里面等待重新消费

![image-20230614111431147](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230614111431147.png)

再次重启，再次消费2条消息，但仍未确认

![image-20230614111518375](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230614111518375.png)

访问http://localhost:8080/ack/send，再次发布消息，消息堆积

![image-20230614111613152](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230614111613152.png)

## 三、设置手动ack(全局)

### 修改消费者手动确认

```java
package com.yolo.demo.controller;

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.AmqpHeaders;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.handler.annotation.Headers;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;
import java.util.Map;
import java.util.stream.IntStream;

@RestController
@RequestMapping("/ack")
public class AckSenderController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/send")
    public void send(@RequestParam(value = "message", required = false, defaultValue = "Hello World") String message) {

        IntStream.range(1, 11).forEach(i -> {
            String msg = message + " ..." + i;
            System.out.println(" [ 生产者 ] Sent ==> '" + msg + "'");
            rabbitTemplate.convertAndSend("helloWorldExchange", "ack", msg);
        });
    }

    private int count1 = 1;
    private int count2 = 1;
    private int count3 = 1;

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "ackQueue", durable = "true"),
            exchange = @Exchange(value = "helloWorldExchange", type = ExchangeTypes.DIRECT, durable = "true"),
            key = "ack"
    ))
    public void receive(Message message, Channel channel) throws InterruptedException, IOException {
        Thread.sleep(200);
        System.out.println(" [ 消费者@1号 ] Received ==> '" + new String(message.getBody()) + "'");
        System.out.println(" [ 消费者@1号 ] 处理消息数：" + count1++);
        // 确认消息
        // 第一个参数，交付标签，相当于消息ID 64位的长整数(从1开始递增)
        // 第二个参数，false表示仅确认提供的交付标签；true表示批量确认所有消息(消息ID小于自身的ID)，包括提供的交付标签
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "ackQueue", durable = "true"),
            exchange = @Exchange(value = "helloWorldExchange", type = ExchangeTypes.DIRECT, durable = "true"),
            key = "ack"
    ))
    public void receive2(Message message, Channel channel, @Headers Map<String, Object> map) throws InterruptedException, IOException {
        Thread.sleep(600);
        System.out.println(" [ 消费者@2号 ] Received ==> '" + new String(message.getBody()) + "'");
        System.out.println(" [ 消费者@2号 ] 处理消息数：" + count2++);

        // 确认消息
        channel.basicAck((Long) map.get(AmqpHeaders.DELIVERY_TAG), false);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "ackQueue", durable = "true"),
            exchange = @Exchange(value = "helloWorldExchange", type = ExchangeTypes.DIRECT, durable = "true"),
            key = "ack"
    ))
    public void receive3(Message message, Channel channel, @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag) throws InterruptedException, IOException {
        Thread.sleep(1000);
        System.out.println(" [ 消费者@3号 ] Received ==> '" + new String(message.getBody()) + "'");
        System.out.println(" [ 消费者@3号 ] 处理消息数：" + count3++);

        // 确认消息
        channel.basicAck(deliveryTag, false);
    }
}
```

### 手动确认测试结果

> 手动确认通过调用方法实现
> basicAck(long deliveryTag, boolean multiple)
> deliveryTag：交付标签，相当于消息ID 64位的长整数(从1开始递增)
> multiple：false表示仅确认提供的交付标签；true表示批量确认所有消息(消息ID小于自身的ID)，包括提供的交付标签

这里发现程序刚刚启动就全部消费完了

![image-20230614112242888](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230614112242888.png)

继续发布，还是消费完成

![image-20230614112416715](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230614112416715.png)

### 修改消费者手动拒绝

```java
package com.yolo.demo.controller;

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.AmqpHeaders;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.handler.annotation.Headers;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;
import java.util.Map;
import java.util.stream.IntStream;

@RestController
@RequestMapping("/ack")
public class AckSenderController {

    @Autowired
    private RabbitTemplate rabbitTemplate;
  
    private int count1 = 1;

    @GetMapping("/send")
    public void send(@RequestParam(value = "message", required = false, defaultValue = "Hello World") String message) {

        IntStream.range(1, 11).forEach(i -> {
            String msg = message + " ..." + i;
            System.out.println(" [ 生产者 ] Sent ==> '" + msg + "'");
            rabbitTemplate.convertAndSend("helloWorldExchange", "ack", msg);
        });
    }

    

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "ackQueue",durable = "true"
            ),
            exchange = @Exchange(value = "helloWorldExchange",type = ExchangeTypes.DIRECT,durable = "true"),
            key = "ack"
    ))
    public void receive4(Message message, Channel channel) throws IOException, InterruptedException {
        Thread.sleep(200);
        System.out.println(" [ 消费者@4号 ] Received ==> '" + new String(message.getBody()) + "'");
        System.out.println(" [ 消费者@4号 ] 消息被我拒绝了：" + count1++);

        // 拒绝消息方式一
        // 第一个参数，交付标签
        // 第二个参数，false表示仅拒绝提供的交付标签；true表示批量拒绝所有消息，包括提供的交付标签
        // 第三个参数，false表示直接丢弃消息，true表示重新排队
        //channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);

        // 拒绝消息方式二
        // 第一个参数，交付标签
        // 第二个参数，false表示直接丢弃消息，true表示重新排队
        // 跟basicNack的区别就是始终只拒绝提供的交付标签
        channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);
    }

}
```

### 手动拒绝测试结果

> channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);
> 这里是拒绝后，重新进入队列，所以消费的总是第一条消息并且循环不停
> 停止程序后，队列仍然是10条消息

![image-20230614112814169](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230614112814169.png)

> channel.basicReject(message.getMessageProperties().getDeliveryTag(), false);
> 改成false，拒绝后直接丢弃
> 重启后：

![image-20230614113024871](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230614113024871.png)

## 四、单个消息设置手动确定

```java
package com.yolo.demo.controller;

import cn.hutool.core.date.DateTime;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.AcknowledgeMode;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.amqp.rabbit.listener.api.ChannelAwareMessageListener;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@Slf4j
@RestController
@RequestMapping("/mq")
public class AckController {

    @Resource
    private RabbitTemplate rabbitTemplate;
    
    @PostMapping("ack2")
    public String ack2() {
        String msgContent = "Hello World " + DateTime.now();
        log.info("{} [生产者] 发送消息: {}", DateTime.now(), msgContent);
        this.rabbitTemplate.convertAndSend("test_exchange", "test_ack_manual_routing_key", msgContent);
        return "SUCCESS";
    }

    @Bean
    public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.setQueueNames("test_ack_manual_queue");
        // 指定ack模式
        container.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        container.setMessageListener((ChannelAwareMessageListener) (message, channel) -> {
            long deliveryTag = message.getMessageProperties().getDeliveryTag();
            log.info("----------------------------------------------");
            log.info("{} [消费者] deliveryTag:{} 接收消息: {}", DateTime.now(), deliveryTag, new String(message.getBody()));
            if (message.getMessageProperties().getHeaders().get("error") == null) {
                channel.basicAck(deliveryTag, false);
            } else {
                channel.basicReject(deliveryTag, false);
            }
        });
        return container;
    }

}
```

## 五、全局确认消息异常处理

- 自动确认涉及到一个问题就是如果在处理消息的时候抛出异常，消息处理失败，但是因为自动确认而导致 Rabbit 将该消息删除了，造成消息丢失

```java
@Bean
public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory){
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
    container.setConnectionFactory(connectionFactory);
    container.setQueueNames("consumer_queue");                 // 监听的队列
    container.setAcknowledgeMode(AcknowledgeMode.NONE);     // NONE 代表自动确认
    container.setMessageListener((MessageListener) message -> {         //消息监听处理
        System.out.println("====接收到消息=====");
        System.out.println(new String(message.getBody()));
        //相当于自己的一些消费逻辑抛错误
        throw new NullPointerException("consumer fail");
    });
    return container;
}
```

- 手动确认消息

```java
@Bean
public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory){
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
    container.setConnectionFactory(connectionFactory);
    container.setQueueNames("consumer_queue");              // 监听的队列
    container.setAcknowledgeMode(AcknowledgeMode.MANUAL);        // 手动确认
    container.setMessageListener((ChannelAwareMessageListener) (message, channel) -> {      //消息处理
        System.out.println("====接收到消息=====");
        System.out.println(new String(message.getBody()));
        if(message.getMessageProperties().getHeaders().get("error") == null){
        channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
            System.out.println("消息已经确认");
        }else {
            //channel.basicNack(message.getMessageProperties().getDeliveryTag(),false,false);
            channel.basicReject(message.getMessageProperties().getDeliveryTag(),false);
            System.out.println("消息拒绝");
        }
 
    });
    return container;
}
```

- `AcknowledgeMode` 除了 NONE 和 MANUAL 之外还有 AUTO ，它会根据方法的执行情况来决定是否确认还是拒绝（是否重新入queue）
  - 如果消息成功被消费（成功的意思是在消费的过程中没有抛出异常），则自动确认
  - 当抛出 `AmqpRejectAndDontRequeueException` 异常的时候，则消息会被拒绝，且 requeue = false（不重新入队列）
  - 当抛出` ImmediateAcknowledgeAmqpException` 异常，则消费者会被确认
  - 其他的异常，则消息会被拒绝，且 requeue = true（如果此时只有一个消费者监听该队列，则有发生死循环的风险，多消费端也会造成资源的极大浪费，这个在开发过程中一定要避免的）。可以通过 `setDefaultRequeueRejected`（默认是true）去设置
    
    

```java
@Bean
public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory){
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
    container.setConnectionFactory(connectionFactory);
    container.setQueueNames("consumer_queue");              // 监听的队列
    container.setAcknowledgeMode(AcknowledgeMode.AUTO);     // 根据情况确认消息
    container.setMessageListener((MessageListener) (message) -> {
        System.out.println("====接收到消息=====");
        System.out.println(new String(message.getBody()));
        //抛出NullPointerException异常则重新入队列
        //throw new NullPointerException("消息消费失败");
        //当抛出的异常是AmqpRejectAndDontRequeueException异常的时候，则消息会被拒绝，且requeue=false
        //throw new AmqpRejectAndDontRequeueException("消息消费失败");
        //当抛出ImmediateAcknowledgeAmqpException异常，则消费者会被确认
        throw new ImmediateAcknowledgeAmqpException("消息消费失败");
    });
    return container;
}
```

## 六、总结

- 未确认：什么也不用写，消息不会移除，重复消费，积攒越来越多
- 确认：channel.basicAck()；确认后，消息从队列中移除
- 拒绝：channel.basicNack()或channel.basicReject()；拒绝后，消息先从队列中移除，然后可以选择重新排队，或者直接丢弃(丢弃还有一种选择，就是加入到死信队列中，用于追踪问题)

> [Gitee项目地址（demo-rabbitmq-ack）](https://gitee.com/huanglei1111/yolo-springboot-demo/tree/master/demo-rabbitmq/demo-rabbitmq-ack)