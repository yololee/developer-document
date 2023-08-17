# springboot：整合shedlock实现分布式定时任务redis版

## 一、简介

ShedLock是一个在分布式环境中使用的定时任务框架，用于解决在分布式环境中的多个实例的相同定时任务在同一时间点重复执行的问题。ShedLock确保计划的任务最多同时执行一次。如果一个任务正在一个节点上执行，它会获得一个锁，该锁将阻止从另一个节点（或线程）执行同一任务。请注意，如果一个任务已经在一个节点上执行，则在其他节点上的执行不会等待，只是将其跳过。。简单来说，ShedLock本身只做一件事情：保证一个任务最多同时执行一次。所以如官网所说的，ShedLock不是一个分布式调度器，只是一个锁!

> 注意：ShedLock支持Mongo，Redis，Hazelcast，ZooKeeper以及任何带有JDBC驱动程序的东西。本例子使用的是基于Redis的方式。之所以不适用基于jdbc存储的主要原因是考虑到大量数据下的数据库压力的原因，若本身基于jdbc等，可直接参考官网给出的提示：https://github.com/lukas-krecan/ShedLock#jdbctemplate. 创建对应的表结构
> 

## 二、整合

### pom文件

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- 对象池，使用redis时必须引入 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
        <!-- shedlock: 分布式定时任务锁 -->
        <!-- https://mvnrepository.com/artifact/net.javacrumbs.shedlock/shedlock-spring -->
        <dependency>
            <groupId>net.javacrumbs.shedlock</groupId>
            <artifactId>shedlock-spring</artifactId>
            <version>4.29.0</version>
        </dependency>
        <!-- 使用redis做分布式任务 -->
        <!-- https://mvnrepository.com/artifact/net.javacrumbs.shedlock/shedlock-provider-redis-spring -->
        <dependency>
            <groupId>net.javacrumbs.shedlock</groupId>
            <artifactId>shedlock-provider-redis-spring</artifactId>
            <version>4.29.0</version>
        </dependency>

        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.8.18</version>
        </dependency>
```

### 配置

```java
package com.example.demo.config;

import net.javacrumbs.shedlock.core.LockProvider;
import net.javacrumbs.shedlock.provider.redis.spring.RedisLockProvider;
import net.javacrumbs.shedlock.spring.annotation.EnableSchedulerLock;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.scheduling.annotation.EnableScheduling;

import java.util.Objects;


/**
 * Shedlock配置类
 */
@Configuration
// 开启定时器
@EnableScheduling
// 开启定时任务锁，并设置默认锁最大时间为30分钟(PT为固定格式，M为时间单位-分钟)
@EnableSchedulerLock(defaultLockAtMostFor = "PT30M")
public class ShedlockConfig {

    @Value("${spring.profiles.active}")
    private String env;

    /**
     * 使用redis存储
     */
    @Bean
    public LockProvider lockProvider(RedisTemplate<String,String> redisTemplate) {
        // keyPrefix: redis key的前缀
        // env和keyPrefix 主要用于区分数据来源，保证最终redis-key在使用时不串用即可  ex=> keyPrefix:dev:scheduledTaskName
        return new RedisLockProvider(Objects.requireNonNull(redisTemplate.getConnectionFactory()), env, "keyPrefix");
    }

}

```

```yml
server:
  port: 80

spring:
  profiles:
    active: dev
  redis:
    host: 127.0.0.1
    # 连接超时时间（记得添加单位，Duration）
    timeout: 10000ms
    # Redis默认情况下有16个分片，这里配置具体使用的分片
    database: 0
    lettuce:
      pool:
        # 连接池最大连接数（使用负值表示没有限制） 默认 8
        max-active: 8
        # 连接池最大阻塞等待时间（使用负值表示没有限制） 默认 -1
        max-wait: -1ms
        # 连接池中的最大空闲连接 默认 8
        max-idle: 8
        # 连接池中的最小空闲连接 默认 0
        min-idle: 0

```

### 使用

```java
package com.example.demo.task;

import cn.hutool.core.date.DateTime;
import lombok.extern.slf4j.Slf4j;
import net.javacrumbs.shedlock.spring.annotation.SchedulerLock;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

/**
 * 数据定时任务
 */
@Slf4j
@Component
public class Timer {

    /**
     * 每5秒执行一次
     *
     * @SchedulerLock 注解参数
     * name：锁的名称，同一时间只能执行一个同名的任务
     * lockAtMostFor：该属性指定在执行节点死亡的情况下应保持锁定多长时间。这只是一个回退，在正常情况下，一旦任务完成就会释放锁。 您必须设置lockAtMostFor一个比正常执行时间长得多的值。如果任务花费的时间超过 lockAtMostFor所产生的行为可能是不可预测的（多个进程将有效地持有锁）
     * lockAtLeastFor：该属性指定应保留锁的最短时间。它的主要目的是在节点之间的任务和时钟差异非常短的情况下防止从多个节点执行。
     * <p>
     * 通过设置lockAtMostFor我们确保即使节点死亡也会释放锁，通过设置lockAtLeastFor 我们确保它在5s内不会执行超过一次。请注意，这lockAtMostFor只是一个安全网，以防执行任务的节点死亡，因此将其设置为明显大于最大估计执行时间的时间。 如果任务花费的时间超过lockAtMostFor，它可能会再次执行并且结果将是不可预测的（更多的进程将持有锁）。
     */
    @Scheduled(cron = "*/5 * * * * ?")
    @SchedulerLock(name = "scheduledTaskName", lockAtMostFor = "10s", lockAtLeastFor = "4s")
    public void printCurrentTime() {
        log.info("现在时间：【{}】", DateTime.now());
    }

}
```

