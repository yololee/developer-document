# springcloud-Ribbon组件实现负载均衡

## 一、Ribbon介绍

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。

简单来说Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法和服务调用。Ribbon客户端组件提供一系列完善的配置项，如连接超时，重试等。简单来说，就是在配置文件中列出Load Balancer(简称LB)后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义负载均衡算法

## 二、案例

![image-20230801110942021](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801110942021.png)

- ribbon-server作为注册中心
- ribbon-provider、ribbon-provider02作为服务提供者
- ribbon-consumer作为服务消费者

### 导入依赖

因为Eureka中已经集成了Ribbon，所以我们无需引入新的依赖

![image-20230801111217971](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801111217971.png)

### 使用Ribbon

在消费者使用RestTemplate进行调用的时候，添加注解`@LoadBalanced`开启负载均衡,默认是轮询模式

```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    @LoadBalanced//开启负载均衡
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

}
```

修改消费者的调用方法

```java
    @GetMapping("/test/ribbon/{id}")
    public Goods testRibbon(@PathVariable("id") int id) {

        //restTemplate 请求访问 order订单服务的 order URL ,
        // 参数URL   http://order/order   第一个order 就是服务名字 就是注册在Eureka上的 服务名字 第二个order就是 controller 入口
        // 第二个参数String.class 就是 访问返回值
        String url = "http://RIBBON-PROVIDER/goods/findOne/"+id;
        Goods result = restTemplate.getForObject(url, Goods.class);
        log.info("user 发起UserController 测试ribbon 调用 :{} ", result);
        return result;
    }
```

这里的`RIBBON-PROVIDER`是控制台中容器的名称

![image-20230801112452702](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801112452702.png)

而且这里需要注意的是一个消费者从俩个提供者中采用Ribbon来进行拿数据，这俩个ribbon的容器名字应该一样

**其中{Spring.application.name}都是一样的，不可以变**

![image-20230801113103486](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801113103486.png)

### 测试

访问接口轮询出现结果

![image-20230801113314327](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801113314327.png)

## 三、Ribbon组件IRule

默认的是RoundBobinRule(轮询)

| 命名     | **内置负载均衡规则类**    | **规则描述**                                                 |
| -------- | ------------------------- | ------------------------------------------------------------ |
| 轮询     | RoundRobinRule            | 简单轮询服务列表来选择服务器。它是Ribbon默认的负载均衡规则。 |
| 可用过滤 | AvailabilityFilteringRule | 对以下两种服务器进行忽略：<br>（1）在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加。<br>（2）并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule规则的客户端也会将其忽略。 |
| 权重     | WeightedResponseTimeRule  | 为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。 |
| 区域权衡 | ZoneAvoidanceRule         | 以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。 |
| 最低并发 | BestAvailableRule         | 忽略哪些短路的服务器，并选择并发数较低的服务器。             |
| 随机     | RandomRule                | 随机选择一个可用的服务器。                                   |
| 重试     | RetryRule                 | 在一个配置时间段内，当选择server不成功，则一直尝试选择一个可用的server |

使用随机轮询

```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    @LoadBalanced//开启负载均衡
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

    @Bean
    public IRule myRule() {
//        return new RoundRobinRule();
        return new RandomRule();
//        return new RetryRule();
    }

}
```

![image-20230801113853626](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801113853626.png)

## 四、自定义负载均衡算法

所谓的自定义Ribbon Client的主要作用就是使用自定义配置替代Ribbon默认的负载均衡策略，注意：自定义的Ribbon Client是有针对性的，一般一个自定义的Ribbon Client是对一个服务提供者(包括服务名相同的一系列副本)而言的。自定义了一个Ribbon Client 它所设定的负载均衡策略只对某一特定服务名的服务提供者有效，但不能影响服务消费者与别的服务提供者通信所使用的策略。

**自定义算法配置**

```java
@Configuration
/*
    配置Ribbon的负载均衡策略
    name：设置服务提供方的应用名称
    configuration:设置负载均衡的Bean
 */
@RibbonClient(name="riboon-provider",configuration= MySelfRule.class)
public class MySelfRule{
	@Bean
	public IRule myRule(){
		return new MyRandomRule();  // 我自定义为每台机器5次，5次之后在轮询到下一个
	}
}
```

**具体算法**

```java
package com.yolo.demo.config;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.Server;

import java.util.List;

public class MyRandomRule extends AbstractLoadBalancerRule {
 
	// total = 0 // 当total==5以后，我们指针才能往下走，
	// index = 0 // 当前对外提供服务的服务器地址，
	// total需要重新置为零，但是已经达到过一个5次，我们的index = 1
	// 分析：我们5次，但是微服务只有8001 8002 8003 三台，OK？
	
	private int total = 0; 	    // 总共被调用的次数，目前要求每台被调用5次
	private int currentIndex = 0;	    // 当前提供服务的机器号
 
	public Server choose(ILoadBalancer lb, Object key){
		if (lb == null) {
			return null;
		}
		Server server = null;
 
		while (server == null) {
			if (Thread.interrupted()) {
				return null;
			}
			List<Server> upList = lb.getReachableServers();  //激活可用的服务
			List<Server> allList = lb.getAllServers();  //所有的服务
 
			int serverCount = allList.size();
			if (serverCount == 0) {
				return null;
			}
		
                        if(total < 5){
	                    server = upList.get(currentIndex);
	                    total++;
                        }else {
	                    total = 0;
	                    currentIndex++;
	                    if(currentIndex >= upList.size()){
	                      currentIndex = 0;
	                    }
                        }							
			if (server == null) {
				Thread.yield();
				continue;
			}
 
			if (server.isAlive()) {
				return (server);
			}
 
			// Shouldn't actually happen.. but must be transient or a bug.
			server = null;
			Thread.yield();
		}
		return server;
	}
	@Override
	public Server choose(Object key){
		return choose(getLoadBalancer(), key);
	}
 
	@Override
	public void initWithNiwsConfig(IClientConfig clientConfig){
	}
}
```

**测试结果**

![image-20230801114356651](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230801114356651.png)

> [Gitee项目地址（demo-riboon）](https://gitee.com/huanglei1111/yolo-springcloud-demo/tree/master/demo-riboon)

