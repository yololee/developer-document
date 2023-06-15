# Rabbitmq注解以及消息序列化

## MessageConvert

- 涉及网络传输的应用序列化不可避免，发送端以某种规则将消息转成 byte 数组进行发送，接收端则以约定的规则进行 byte[] 数组的解析
- RabbitMQ 的序列化是指 Message 的 body 属性，即我们真正需要传输的内容，**RabbitMQ 抽象出一个 MessageConvert 接口处理消息的序列化**，其实现有 `SimpleMessageConverter`（默认）、`Jackson2JsonMessageConverter `等
- 当调用了 convertAndSend 方法时会使用 MessageConvert 进行消息的序列化
- `SimpleMessageConverter` 对于要发送的消息体 body 为 byte[] 时不进行处理，如果是 String 则转成字节数组,如果是 Java 对象，则使用 jdk 序列化将消息转成字节数组，转出来的结果较大，含class类名，类相应方法等信息。因此性能较差
- 当使用 RabbitMQ 作为中间件时，数据量比较大，此时就要考虑使用类似 `Jackson2JsonMessageConverter` 等序列化形式以此提高性能

## @RabbitListener 用法

> **使用注解`@RabbitListener`标注方法 **

 `@RabbitListener` 的 bindings 属性声明 Binding（若 RabbitMQ 中不存在该绑定所需要的 `Queue`、`Exchange`、`RouteKey` 则自动创建，若存在则抛出异常）

```java
@RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "queue_email", durable = "true"),
            exchange = @Exchange(
                    value = "topic.exchange",
                    ignoreDeclarationExceptions = "true",
                    type = ExchangeTypes.TOPIC
            ),
            key = {"topic.#.email.#","email.*"}))
    public void receiveEmail(String msg){
        System.out.println(" [邮件服务] received : " + msg + "!");
    }
```

> **@RabbitListener 可以标注在类上面，需配合 @RabbitHandler 注解一起使用**
>
> **@RabbitListener 标注在类上面表示当有收到消息的时候，就交给 @RabbitHandler 的方法处理，具体使用哪个方法处理，根据 MessageConverter 转换后的参数类型**

```java
@Component
@RabbitListener(queues = "consumer_queue")
public class Receiver {

    @RabbitHandler
    public void processMessage1(String message) {
        System.out.println(message);
    }

    @RabbitHandler
    public void processMessage2(byte[] message) {
        System.out.println(new String(message));
    }
}
```

#### ==注意：==

- 消息处理方法参数是由`MessageConverter`转化，若使用自定义`MessageConverter`则需要在`RabbitListenerContainerFactory`实例中取设置（spring默认实现的是`SimpleRabbitListenerContainerFactory`）

- 消息的`content_type `属性表示消息body数据以什么数据格式存储，接收消息除了使用`Message`对象接收消息(包含消息属性等信息)之外，还可以直接使用对应类型接收消息body内容但若方法参数类型不正确会抛出异常
  - **application/octet-stream**：二进制字节数组存储，使用 byte[]
  - **application/x-java-serialized-object**：java 对象序列化格式存储，使用 Object、相应类型（反序列化时类型应该同包同名，否者会抛出找不到类异常）
  - **text/plain**：文本数据类型存储，使用 String
  - **application/json**：JSON 格式，使用 Object、相应类型

## @Headers和@Payload

- @Header 注入消息头的单个属性
- @Payload 注入消息体到一个JavaBean中
- @Headers 注入所有消息头到一个Map中

```java
    /**
     * 可以直接通过注解声明交换器、绑定、队列。但是如果声明的和rabbitMq中已经存在的不一致的话
     * 会报错便于测试，我这里都是不使用持久化，没有消费者之后自动删除
     * {@link RabbitListener}是可以重复的。并且声明队列绑定的key也可以有多个.
     *
     * @param headers
     * @param msg
     */
@RabbitListener(
        bindings = @QueueBinding(
            exchange = @Exchange(value = RabbitMQConstant.DEFAULT_EXCHANGE, type = ExchangeTypes.TOPIC,
                durable = RabbitMQConstant.FALSE_CONSTANT, autoDelete = RabbitMQConstant.true_CONSTANT),
            value = @Queue(value = RabbitMQConstant.DEFAULT_QUEUE, durable = RabbitMQConstant.FALSE_CONSTANT,
                autoDelete = RabbitMQConstant.true_CONSTANT),
            key = DKEY
        ),
        //手动指明消费者的监听容器，默认Spring为自动生成一个SimpleMessageListenerContainer
        containerFactory = "container",
        //指定消费者的线程数量,一个线程会打开一个Channel，一个队列上的消息只会被消费一次（不考虑消息重新入队列的情况）,下面的表示至少开启5个线程，最多10个。线程的数目需要根据你的任务来决定，如果是计算密集型，线程的数目就应该少一些
        concurrency = "5-10"
    )
    public void process(@Headers Map<String, Object> headers, @Payload ExampleEvent msg) {
        log.info("basic consumer receive message:{headers = [" + headers + "], msg = [" + msg + "]}");
    }
 
 
/**
     * {@link Queue#ignoreDeclarationExceptions}声明队列会忽略错误不声明队列，这个消费者仍然是可用的
     *
     * @param headers
     * @param msg
     */
    @RabbitListener(queuesToDeclare = @Queue(value = RabbitMQConstant.DEFAULT_QUEUE, ignoreDeclarationExceptions = RabbitMQConstant.true_CONSTANT))
    public void process2(@Headers Map<String, Object> headers, @Payload ExampleEvent msg) {
        log.info("basic2 consumer receive message:{headers = [" + headers + "], msg = [" + msg + "]}");
    }
```

#### 注意

如果是`com.rabbitmq.client.Channel`,`org.springframework.amqp.core.Message`和`org.springframework.messaging.Message`这些类型，可以不加注解，直接可以注入。如果不是这些类型，那么不加注解的参数将会被当做消息体。不能多于一个消息体。如下方法ExampleEvent就是默认的消息体

```java
public void process2(@Headers Map<String, Object> headers,ExampleEvent msg);
```