# springcloud：Hystrix组件实现服务熔断

## 一、Hystrix介绍

> 注意：Hystrix 已经停止开发新版本了，仅在进行部分维护工作，最后一版为2018年11月发布

### Hystrix 定义

Hystrix： 是由 Netflix 开源的一个服务隔离组件。在分布式环境中，服务与服务之间的依赖错综复杂，一种不可避免的情况就是总会有某些服务会出现故障，导致依赖于它们的其它服务出现远程调度的线程阻塞。Hystrix 提供了 熔断器 功能，能够阻止分布式系统中出现联动故障。Hystrix 是通过隔离服务的访问点阻止联动故障的，并提供了故障的解决方案，从而提高了整个分布式系统的弹性

**官方文档：** https://github.com/Netflix/Hystrix/wiki/

**GitHub：** https://github.com/Netflix/Hystrix

### Hystrix 的三种状态

1、熔断关闭状态（Closed）

服务没有故障时，熔断器所处的状态，对调用方的调用不做任何限制。

2、熔断开启状态（Open）

在固定时间窗口内（Hystrix 默认是 10 秒），接口调用出错比率达到一个阈值（Hystrix 默认为 50%），会进入熔断开启状态。进入熔断状态后，后续对该服务接口的调用不再经过网络，直接执行本地的 fallback() 方法。

3、半熔断状态（Half-Open）

在进入熔断开启状态一段时间后（Hystrix 默认是 5 秒），熔断器会进入半熔断状态。所谓半熔断就是尝试恢复服务调用，允许有限的流量调用该服务，并监控调用成功率。如果成功率达到预期，则说明服务已恢复，进入熔断关闭状态；如果成功率仍旧很低，则重新进入熔断关闭状态。

![](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230803092104970.png)

### Hystrix 的设计原理

- 防止单个依赖项耗尽所有容器（例如 Tomcat）用户线程。
- 快速卸载负载，让请求失败而不是排队。
- 在可行的情况下提供回退以保护用户后续请求正常。
- 使用隔离技术（例如 bulkhead（隔板），swimlane（泳道），circuit breaker（断路器）模式）来限制任何一个依赖服务的影响。
- 通过近乎实时的指标、监控和告警优化故障发现时间。
- 通过配置更改的低延迟传播和对 Hystrix 大部分方面的动态属性更改的支持来优化恢复时间，让我们可以通过低延迟反馈循环进行实时操作修改。
- 防止整个依赖客户端执行中的故障，而不仅仅是网络流量中的故障

### Hystrix 的实现原理

- 将所有对外部系统（或“依赖项”）的调用包装在一个 HystrixCommand 或 HystrixObservableCommand 对象中，该对象通常在单独的线程中执行（这是命令模式的一个示例
- 超过您定义的阈值的超时调用。有一个默认值，但对于大多数依赖项，我们可以通过“properties”自定义设置这些超时，以便它们略高于每个依赖项测量的第99.5个百分位性能。
- 为每个依赖项维护一个小线程池（或信号量）；如果它已满，发往该依赖项的请求将立即被拒绝，而不是排队。
- 测量成功、失败（客户端抛出的异常）、超时和线程拒绝。
- 如果服务的错误百分比超过阈值，则手动或自动触发断路器以停止对特定服务的所有请求一段时间。
  当请求失败、被拒绝、超时或短路时执行回退逻辑。
- 近乎实时地监控指标和配置变化

## 二、Hystrix-降级实现

![image-20230803095024466](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230803095024466.png)

### 服务端降级

**在服务提供方，引入 hystrix 依赖**

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.2.RELEASE</version>
        </dependency>
```

**定义降级方法**

```java
    /**
     * 定义降级方法：
     *  1. 方法的返回值需要和原方法一样
     *  2. 方法的参数需要和原方法一样
     */
    public Goods findOne_fallback(int id){
        Goods goods = new Goods();
        goods.setTitle("降级了~~~");

        return goods;
    }
```

**使用 @HystrixCommand 注解配置降级方法**

```java
 /**
     * 降级：
     *  1. 出现异常
     *  2. 服务调用超时
     *      * 默认1s超时
     *
     *  @HystrixCommand(fallbackMethod = "findOne_fallback")
     *      fallbackMethod：指定降级后调用的方法名称
     */
    @HystrixCommand(fallbackMethod = "findOne_fallback",commandProperties = {
            //设置Hystrix的超时时间，默认1s
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })
    @GetMapping("/findOne/{id}")
    public Goods findOne(@PathVariable("id") int id){
        //1.造个异常
        int i = 3/0;
        try {
            //2. 休眠2秒
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Goods goods = new Goods(id, "苹果手机", 3999, 10000);
        goods.setTitle(goods.getTitle() + ":" + "8899");//
        return goods;
    }
```

**在启动类上开启Hystrix功能**

```java
@SpringBootApplication
@EnableEurekaClient
@EnableHystrix
public class HystrixProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixProviderApplication.class, args);
    }

}
```

**测试**

```java
get请求：127.0.0.1:8002/order/goods/1
响应结果：
{
	"id": 0,
	"title": "降级了~~~provider02",
	"price": 0,
	"count": 0
}
```

### 消费端降级

消费方一般使用feign调用服务, feign 组件中已经集成了 hystrix 组件。我们不需要再引入依赖

**定义feign 调用接口实现类，复写方法，即 降级方法**

```java
/**
 * Feign 客户端的降级处理类
 * 1. 定义类 实现 Feign 客户端接口
 * 2. 使用@Component注解将该类的Bean加入SpringIOC容器
 */
@Component
public class GoodsFeignClientFallback implements GoodsFeignClient {

    @Override
    public Goods findOne(int id) {
        Goods goods = new Goods();
        goods.setTitle("又被降级了~~~");
        return goods;
    }
}
```

**在 @FeignClient 注解中使用 fallback 属性设置降级处理类**

```java
@FeignClient(value = "hystrix-provider",configuration = FeignConfig.class,fallback = GoodsFeignClientFallback.class)
public interface GoodsFeignClient {
    @GetMapping("/goods/findOne/{id}")
    Goods findOne(@PathVariable("id") int id);

}
```

**配置开启 `feign.hystrix.enabled = true`**

```yml
# 开启feign对hystrix的支持
feign:
  hystrix:
    enabled: true
```

**测试**

测试消费端降级的时候给服务端降级操作关闭

```java
get请求：127.0.0.1:8002/order/goods/1
响应结果：
{
	"id": 0,
	"title": "又被降级了~~~",
	"price": 0,
	"count": 0
}
```

## 三、Hystrix-熔断

**熔断器介绍**

Hystrix在运行过程中会向每个commandKey对应的熔断器报告成功、失败、超时和拒绝的状态，熔断器维护并统计这些数据，并根据这些统计信息来决策熔断开关是否打开。如果打开，熔断后续请求，快速返回。隔一段时间（默认是5s）之后熔断器尝试半开，放入一部分流量请求进来，相当于对依赖服务进行一次健康检查，如果请求成功，熔断器关闭

**修改服务提供者中的controller**

```java
    @GetMapping("/{id}")
    @HystrixCommand(fallbackMethod = "circuitBreakerFallback",commandProperties = {
            //开启熔断
            @HystrixProperty(name="circuitBreaker.enabled",value = "true"),
            //设置Hystrix的超时时间，默认1s
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "3000"),
            //监控时间 默认5000 毫秒
            @HystrixProperty(name="circuitBreaker.sleepWindowInMilliseconds",value = "5000"),
            //失败次数。默认20次
            @HystrixProperty(name="circuitBreaker.requestVolumeThreshold",value = "10"),
            //失败率 默认50%
            @HystrixProperty(name="circuitBreaker.errorThresholdPercentage",value = "50") })
    public String circuitBreaker(@PathVariable("id") Integer id) {
        if (id<0){
            throw  new RuntimeException("id ="+id+"，不能为负数");
        }
        return "调用成功 id=" + id;
    }
    public String circuitBreakerFallback(@PathVariable("id") Integer id) {
        return "id =" + id + ", 不能为负数";
    }
```

**修改服务消费者的feign接口**

```java
@FeignClient(value = "hystrix-provider",configuration = FeignConfig.class,fallback = GoodsFeignClientFallback.class)
public interface GoodsFeignClient {
    @GetMapping("/goods/findOne/{id}")
    Goods findOne(@PathVariable("id") int id);

    @GetMapping("/goods/{id}")
    String circuitBreaker(@PathVariable("id") Integer id);

}
```

**修改服务消费者的controller**

```java
    @GetMapping("/{id}")
    public String circuitBreaker(@PathVariable("id") int id){
        return goodsFeignClient.circuitBreaker(id);
    }
```

**测试**

正常访问

![image-20230803101328996](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230803101328996.png)

错误访问

![image-20230803101342143](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230803101342143.png)

多次错误访问之后

![image-20230803101356094](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230803101356094.png)

## 四、完整版配置

```yml
spring:
  main:
    allow-bean-definition-overriding: true

#default可替换
hystrix:
  command:
    default:
      execution:
        isolation:
          #线程池隔离还是信号量隔离 默认是THREAD 信号量是SEMAPHORE
          strategy: THREAD
          semaphore:
            #使用信号量隔离时，支持的最大并发数 默认10
            maxConcurrentRequests: 10
          thread:
            #command的执行的超时时间 默认是1000
            timeoutInMilliseconds: 2000
            #HystrixCommand.run()执行超时时是否被打断 默认true
            interruptOnTimeout: true
            #HystrixCommand.run()被取消时是否被打断 默认false
            interruptOnCancel: false
        timeout:
          #command执行时间超时是否抛异常 默认是true
          enabled: true
        fallback:
          #当执行失败或者请求被拒绝，是否会尝试调用hystrixCommand.getFallback()
          enabled: true
          isolation:
            semaphore:
              #如果并发数达到该设置值，请求会被拒绝和抛出异常并且fallback不会被调用 默认10
              maxConcurrentRequests: 10
      circuitBreaker:
        #用来跟踪熔断器的健康性，如果未达标则让request短路 默认true
        enabled: true
        #一个rolling window内最小的请求数。如果设为20，那么当一个rolling window的时间内
        #（比如说1个rolling window是10秒）收到19个请求，即使19个请求都失败，也不会触发circuit break。默认20
        requestVolumeThreshold: 5
        # 触发短路的时间值，当该值设为5000时，则当触发circuit break后的5000毫秒内
        #都会拒绝request，也就是5000毫秒后才会关闭circuit，放部分请求过去。默认5000
        sleepWindowInMilliseconds: 5000
        #错误比率阀值，如果错误率>=该值，circuit会被打开，并短路所有请求触发fallback。默认50
        errorThresholdPercentage: 50
        #强制打开熔断器，如果打开这个开关，那么拒绝所有request，默认false
        forceOpen: false
        #强制关闭熔断器 如果这个开关打开，circuit将一直关闭且忽略
        forceClosed: false
      metrics:
        rollingStats:
          #设置统计的时间窗口值的，毫秒值，circuit break 的打开会根据1个rolling window的统计来计算。若rolling window被设为10000毫秒，
          #则rolling window会被分成n个buckets，每个bucket包含success，failure，timeout，rejection的次数的统计信息。默认10000
          timeInMilliseconds: 10000
          #设置一个rolling window被划分的数量，若numBuckets＝10，rolling window＝10000，
          #那么一个bucket的时间即1秒。必须符合rolling window % numberBuckets == 0。默认10
          numBuckets: 10
        rollingPercentile:
          #执行时是否enable指标的计算和跟踪，默认true
          enabled: true
          #设置rolling percentile window的时间，默认60000
          timeInMilliseconds: 60000
          #设置rolling percentile window的numberBuckets。逻辑同上。默认6
          numBuckets: 6
          #如果bucket size＝100，window＝10s，若这10s里有500次执行，
          #只有最后100次执行会被统计到bucket里去。增加该值会增加内存开销以及排序的开销。默认100
          bucketSize: 100
        healthSnapshot:
          #记录health 快照（用来统计成功和错误绿）的间隔，默认500ms
          intervalInMilliseconds: 500
      requestCache:
        #默认true，需要重载getCacheKey()，返回null时不缓存
        enabled: true
      requestLog:
        #记录日志到HystrixRequestLog，默认true
        enabled: true
  collapser:
    default:
      #单次批处理的最大请求数，达到该数量触发批处理，默认Integer.MAX_VALUE
      maxRequestsInBatch: 2147483647
      #触发批处理的延迟，也可以为创建批处理的时间＋该值，默认10
      timerDelayInMilliseconds: 10
      requestCache:
        #是否对HystrixCollapser.execute() and HystrixCollapser.queue()的cache，默认true
        enabled: true
  threadpool:
    default:
      #并发执行的最大线程数，默认10
      coreSize: 10
      #Since 1.5.9 能正常运行command的最大支付并发数
      maximumSize: 10
      #BlockingQueue的最大队列数，当设为－1，会使用SynchronousQueue，值为正时使用LinkedBlcokingQueue。
      #该设置只会在初始化时有效，之后不能修改threadpool的queue size，除非reinitialising thread executor。
      #默认－1。
      maxQueueSize: -1
      #即使maxQueueSize没有达到，达到queueSizeRejectionThreshold该值后，请求也会被拒绝。
      #因为maxQueueSize不能被动态修改，这个参数将允许我们动态设置该值。if maxQueueSize == -1，该字段将不起作用
      queueSizeRejectionThreshold: 5
      #Since 1.5.9 该属性使maximumSize生效，值须大于等于coreSize，当设置coreSize小于maximumSize
      allowMaximumSizeToDivergeFromCoreSize: false
      #如果corePoolSize和maxPoolSize设成一样（默认实现）该设置无效。
      #如果通过plugin（https://github.com/Netflix/Hystrix/wiki/Plugins）使用自定义实现，该设置才有用，默认1.
      keepAliveTimeMinutes: 1
      metrics:
        rollingStats:
          #线程池统计指标的时间，默认10000
          timeInMilliseconds: 10000
          #将rolling window划分为n个buckets，默认10
          numBuckets: 10
```

> [Gitee项目地址（demo-hystrix）](https://gitee.com/huanglei1111/yolo-springcloud-demo/tree/master/demo-hystrix)

