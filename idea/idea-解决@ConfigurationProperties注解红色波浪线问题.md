# idea-解决@ConfigurationProperties注解红色波浪线问题

### 问题

在springboot项目中读取application.properties文件的时候使用@ConfigurationProperties注解，注解下面会出现红色波浪线的

![image-20230511140945840](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511140945840.png)


### 解决方法

>  第一步添加依赖

```xml
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
  </dependency>
```

> 第二步：在注解的上方增加一个注解

@EnableConfigurationProperties({当前类.class})

![image-20230511141007894](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511141007894.png)