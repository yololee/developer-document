# springboot-项目启动后自动执行其他业务功能

## 一、业务场景

1. 需要在容器启动的时候执行一些内容（读取配置文件，数据库连接之类）
2. 应用服务启动时，加载一些数据和执行一些应用的初始化动作（删除临时文件，清除缓存，读取配置文件信息等）

## 二、解决方案

### 方案一：实现ApplicationRunner接口

```java
package com.hl.springbootrunner.service;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
 
@Component  //被 spring 容器管理
@Order(1)   //如果多个自定义的 ApplicationRunner  ，用来标明执行的顺序
public class ApplicationRunnerStartService implements ApplicationRunner {
	
	private static final Logger logger =  LoggerFactory.getLogger(ApplicationRunnerStartService.class);
	
	@Override
	public void run(ApplicationArguments args){
		logger.info("===SpringBoot项目启动后，执行ApplicationRunnerStartService方法，进行初始化操作 ============= 1");
	}
 
}
```

```java
package com.hl.springbootrunner.service;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Order(2)
public class ApplicationRunnerStartService2 implements ApplicationRunner {

    private static final Logger logger =  LoggerFactory.getLogger(ApplicationRunnerStartService2.class);

    @Override
    public void run(ApplicationArguments args) {
        logger.info("===SpringBoot项目启动后，执行ApplicationRunnerStartService2方法，进行初始化操作 ============= 2");
    }
}

```

![image-20230518154200512](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230518154200512.png)


查看执行结果可以知道，这俩个方法在容器启动的时候也自动执行了，这里需要主要的是，如果有多个类实现了`ApplicationRunner`接口，可以用`@Order`来指定执行顺序，数值越小，优先级越高

### 方案2：实现CommandLineRunnerStartService接口

```java
package com.hl.springbootrunner.service;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
 
@Component  //被 spring 容器管理
@Order(10)  //如果多个自定义的 CommandLineRunner，用来标明执行的顺序，数字越小，顺序越靠前
public class CommandLineRunnerStartService  implements CommandLineRunner{
 
	private static final Logger logger =  LoggerFactory.getLogger(CommandLineRunnerStartService.class);

	
	@Override
	public void run(String... args) throws Exception {
		logger.info("===SpringBoot项目启动后，执行CommandLineRunnerStartService方法，进行初始化操作 =============");
		//执行自己的业务逻辑
	}
 
}
```

![image-20230518154139464](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230518154139464.png)


### 方案三：@Scheduled注解

```java
package com.hl.springbootrunner.service;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class OACheckScheduleTask {
 
    private static final Logger LOG = LoggerFactory.getLogger(OACheckScheduleTask.class);

    //@Scheduled(cron = "0 0 2 1/1 * ? ")//每天02:00执行
    @Scheduled(fixedDelay = 1 * 60 * 1000)//每1分钟一次
    public void configureTasks() {
 
        LOG.info("*********");
        LOG.info("*********检查人员数据异常信息开始************");
        LOG.info("************************************");
 
        /**
         * 获取角色配置离职人员配置信息
         */

        LOG.info("*************************************");
        LOG.info("*************检查人员数据异常信息结束******************");
        LOG.info("***********************************");
 
    }
}
```

> 注意：
>
> 在启动类上加上`@EnableScheduling`注解
>
> 这一种方法和上面的方法不一样，这是一个定时任务