# springboot-数据库（URL）参数介绍

## 一、字符集

```java
useUnicode=true&characterEncoding=UTF-8    
```

useUnicode 为是否使用Unicode字符集，如果参数characterEncoding设置为UTF-8或gbk，本参数值必须设置为true

这里设置characterEncoding=UTF-8是因为安装mysql是一般选用的是utf-8,也有可能问GBK编码，为防止错误选择characterEncoding=UTF-8

## 二、时区

```java
serverTimezone=UTC
```

为什么要设置serverTimezone=UTC，是因为JDBC连接Mysql6 以上`com.mysql.cj.jdbc.Driver`， 需要指定时区serverTimezone:

UTC为世界标准时间，比北京时间早8个小时

也可以设置`serverTimezone=CST`或者设置为`serverTimezone=Asia/Shanghai`

`CST`表示上海

## 三、允许批量操作

```java
allowMultiQueries=true
```

mybatis是默认不允许批量操作的，这里在URL后面拼接之后，就允许批量操作了

## 四、是否重新连接

```java
autoReconnect=true&failOverReadOnly=false
```

当数据库连接异常中断时，是否自动重新连接，设置为true表示会重新连接

如果连接闲置8小时 (8小时内没有进行数据库操作), mysql就会自动断开连接

`failOverReadOnly` 自动重连成功后，连接是否设置为只读？

## 五、兼容更高版本的数据库

```java
useSSL=false
```

SSL(Secure Sockets Layer 安全套接字协议)，在mysql进行连接的时候,如果mysql的版本是5.7之后的版本必须要加上useSSL=false,mysql5.7以及之前的版本则不用进行添加useSSL=false，会默认为false，一般情况下都是使用useSSL=false，尤其是在将项目部署到linux上时，一定要使用useSSL=false！useSSL=true是进行安全验证，一般通过证书或者令牌什么的，useSSL=false就是通过账号密码进行连接，通常使用useSSL=false！

## 六、其他参数

| 参数名称                      | 参数说明                                                     | 缺省  | 最低版本要求 |
| ----------------------------- | ------------------------------------------------------------ | ----- | ------------ |
| user                          | 数据库用户名，用于连接数据库                                 |       | 所有版本     |
| password                      | 用户密码（用于连接数据库）                                   |       | 所有版本     |
| useUnicode                    | 是否使用Unicode字符集，如果参数characterEncoding设置为gb2312或gbk，本参数值必须设置为true | false | 1.1g         |
| characterEncoding             | 当useUnicode设置为true时，指定字符编码。比如可设置为gb2312或gbk，utf8 | false | 1.1g         |
| autoReconnect                 | 当数据库连接异常中断时，是否自动重新连接？                   | false | 1.1          |
| autoReconnectForPools         | 是否使用针对数据库连接池的重连策略                           | false | 3.1.3        |
| failOverReadOnly              | 自动重连成功后，连接是否设置为只读？                         | true  | 3.0.12       |
| maxReconnectsautoReconnect    | 设置为true时，重试连接的次数                                 | 3     | 1.1          |
| initialTimeoutautoReconnect设 | 置为true时，两次重连之间的时间间隔，单位：秒                 | 2     | 1.1          |
| connectTimeout                | 和数据库服务器建立socket连接时的超时，单位：毫秒。 0表示永不超时，适用于JDK 1.4及更高版本 | 0     | 3.0.1        |
| socketTimeoutsocket           | 操作（读写）超时，单位：毫秒。 0表示永不超时                 | 0     | 3.0.1        |

对应中文环境，通常mysq[l](http://www.2cto.com/database/mysql/)连接URL可以设置为：

```java
jdbc:mysql://localhost:3306/test?user=root&password=&useUnicode=true&characterEncoding=gbk&autoReconnect=true&failOverReadOnly=false
```

在使用数据库连接池的情况下，最好设置如下两个参数：

```java
autoReconnect=true&failOverReadOnly=false
```

需要注意的是，在xml配置文件中，url中的&符号需要转义成&。比如在tomcat的server.xml中配置数据库连接池时，mysql jdbc url样例如下：

```xml
jdbc:mysql://localhost:3306/test?user=root&amp;password=&amp;useUnicode=true&amp;characterEncoding=gbk
&amp;autoReconnect=true&amp;failOverReadOnly=false
```

`amp`; 是html中的或者url地址栏中的转义字符，就是代表&的意思