# springcloud：Feign组件之服务调用

## 一、Feign概述


### Feign是什么

通过RestTemplate调用其它服务的API时，所需要的参数须在请求的URL中进行拼接，如果参数少的话或许我们还可以忍受，一旦有多个参数的话，这时拼接请求字符串就会效率低下

Feign 是一个声明式的 REST 客户端，它用了基于接口的注解方式，很方便实现客户端配置

而Feign则会完全代理HTTP请求，我们只需要像调用方法一样调用它就可以完成服务请求及相关处理。Feign整合了Ribbon和Hystrix，可以让我们不再需要显式地使用这两个组件

### Feign集成了 Ribbon

利用 Ribbon维护了 Paymente的服务列表信息,并且通过轮询实现了客户端的负载均衡。而与 Ribbon不同的是,通过fign只需要定义服务绑定接口且以声明式的方法,优雅而简单的实现了服务调用

### Fegin和OpenFegion区别

| Feign                                                        | OpenFeign                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Feign是SpringCloud组件的一个清量级restful的http服务客户端Feign内置了Ribbon，用来做客户端均衡，去调用服务注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务 | OpenFeign是SpringCloud在Feign的基础上支持了SpringMVC的注解，如@RequestMapper等，OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapper注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用到其他服务 |
| `<dependency><groupId>org.springframework.cloud</groupId>    <artifactId>spring-cloud-starter-feign</artifactId>    <version>1.4.7.RELEASE</version></dependency>` | `<dependency>     <groupId>org.springframework.cloud</groupId>     <artifactId>spring-cloud-starter-openfeign</artifactId>     <version>2.2.6.RELEASE</version> </dependency> ` |

## 二、案例

项目搭建基于Eureka搭建

![image-20230801141657127](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801141657127.png)

- feign-server：注册中心
- feign-comsumer：服务消费端
- feign-provider：服务提供端

### 简单实用

**消费者端引入依赖**

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
```

**定义feign接口**

这里feign接口里面的方法要跟服务提供者的接口保持一致

```java
/**
 *
 * feign声明式接口。发起远程调用的。
 * String url = "http://FEIGN-PROVIDER/goods/findOne/"+id;
 * Goods goods = restTemplate.getForObject(url, Goods.class);
 * 1. 定义接口
 * 2. 接口上添加注解 @FeignClient,设置value属性为 服务提供者的 应用名称
 * 3. 编写调用接口，接口的声明规则 和 提供方接口保持一致。
 * 4. 注入该接口对象，调用接口方法完成远程调用
 */
@FeignClient(value = "feign-provider")
public interface GoodsFeignClient {
    @GetMapping("/goods/findOne/{id}")
    Goods findOne(@PathVariable("id") int id);
}
```

**添加注解开启feign功能**

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class FeignConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignConsumerApplication.class, args);
    }

}
```

**测试**

```java
@RestController
@RequestMapping("/order")
@Slf4j
public class OrderController {
    @Autowired
    private GoodsFeignClient goodsFeignClient;

    @GetMapping("/goods/{id}")
    public Goods findOne(@PathVariable("id") int id){
        Goods goods = goodsFeignClient.findOne(id);
        log.info("feign接口调用结果{}",goods);
        return goods;
    }
}
```

![image-20230801142900289](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801142900289.png)

### Feign超时配置

Feign 底层依赖于 Ribbon 实现负载均衡和远程调用

**修改服务提供者的controller**

```java
    @GetMapping("/threeSeconds")
    public String threeSeconds(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "hello word";
    }
```

**修改消费者的feign接口**

```java
@FeignClient(value = "feign-provider")
public interface GoodsFeignClient {
    @GetMapping("/goods/findOne/{id}")
    Goods findOne(@PathVariable("id") int id);

    @GetMapping("/goods/threeSeconds")
    String threeSeconds();

}
```

```java
@RestController
@RequestMapping("/order")
@Slf4j
public class OrderController {
    @Autowired
    private GoodsFeignClient goodsFeignClient;

    @GetMapping("/goods/{id}")
    public Goods findOne(@PathVariable("id") int id){
        Goods goods = goodsFeignClient.findOne(id);
        log.info("feign接口调用结果{}",goods);
        return goods;
    }

    @GetMapping("/test")
    public String threeSeconds(){
        String seconds = goodsFeignClient.threeSeconds();
        log.info("feign接口调用结果{}",seconds);
        return seconds;
    }
}
```

现在启动服务访问，会出现连接超时的错误

![image-20230801143549297](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801143549297.png)

**解决方法：修改feign默认超时时间**

```yml
feign:
  client:
    config:
      default:   # 这里填具体的服务名称(也可以填default，表示对所有服务生效)
        # connect-timeout 和 read-timeout 这俩个一起配置才生效
        connect-timeout: 4000  #建立连接所用的时间，适用于网络状况正常的情况下，两端连接所需要的时间
        read-timeout: 10000    #指建立连接后从服务端读取到可用资源所用的时间
```

### Feign日志记录

Feign 只能记录 debug 级别的日志信息

```yml
# 设置当前的日志级别 debug，feign只支持记录debug级别的日志
logging:
  level:
    com.yolo.demo: debug
```

**定义Feign日志级别Bean**

```java
@Configuration
public class FeignConfig {
    /*
        NONE,不记录
        BASIC,记录基本的请求行，响应状态码数据
        HEADERS,记录基本的请求行，响应状态码数据，记录响应头信息
        FULL;记录完成的请求 响应数据
     */
    @Bean
    public feign.Logger.Level level(){
        return Logger.Level.FULL;
    }
}
```

**启动bean**

```java
@FeignClient(value = "feign-provider",configuration = FeignConfig.class)
public interface GoodsFeignClient {
    @GetMapping("/goods/findOne/{id}")
    Goods findOne(@PathVariable("id") int id);

    @GetMapping("/goods/threeSeconds")
    String threeSeconds();

}
```

![image-20230801144820086](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801144820086.png)

### Feign上传文件

需求：消费者上传文件到提供者，然后打印文件名称

**提供者添加接口**

```java
@RestController
@RequestMapping("/file")
@Slf4j
public class FileController {

    @PostMapping(value = "/uploadFile", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public String handleFileUpload(@RequestPart(value = "file") MultipartFile file) {
        String originalFilename = file.getOriginalFilename();
        log.info("文件名称为{}",originalFilename);
        return originalFilename;
    }
}
```

**消费者修改feign接口和配置**

```java
@FeignClient(value = "feign-provider",configuration = FeignConfig.class)
public interface GoodsFeignClient {
    @GetMapping("/goods/findOne/{id}")
    Goods findOne(@PathVariable("id") int id);

    @GetMapping("/goods/threeSeconds")
    String threeSeconds();

    @PostMapping(value = "/file/uploadFile", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    String handleFileUpload(@RequestPart(value = "file") MultipartFile file);

}
```

```java
package com.yolo.demo.config;
import feign.Logger;
import feign.codec.Encoder;
import feign.form.spring.SpringFormEncoder;
import org.springframework.boot.autoconfigure.http.HttpMessageConverters;
import org.springframework.cloud.openfeign.support.SpringEncoder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.Scope;
import org.springframework.web.client.RestTemplate;

@Configuration
public class FeignConfig {
    /*
        NONE,不记录
        BASIC,记录基本的请求行，响应状态码数据
        HEADERS,记录基本的请求行，响应状态码数据，记录响应头信息
        FULL;记录完成的请求 响应数据
     */
    @Bean
    public feign.Logger.Level level(){
        return Logger.Level.FULL;
    }

    @Bean
    @Primary
    @Scope("prototype")
    public Encoder multipartFormEncoder() {
        return new SpringFormEncoder(new SpringEncoder(() -> new HttpMessageConverters(new RestTemplate().getMessageConverters())));
    }
}
```

**测试**

```java
@RestController
@RequestMapping("/file")
@Slf4j
public class FileController {

    @Autowired
    private GoodsFeignClient goodsFeignClient;


    @PostMapping("/test")
    public String threeSeconds(@RequestPart(value = "file", required = false) MultipartFile file){
        String result = goodsFeignClient.handleFileUpload(file);
        log.info("feign接口调用结果{}",result);
        return result;
    }
}
```

![](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801151204705.png)

> [Gitee项目地址（demo-feign）](https://gitee.com/huanglei1111/yolo-springcloud-demo/tree/master/demo-feign)

