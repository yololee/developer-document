# springboot：整合p6spy实现自定义日志打印

### 介绍

P6Spy 是针对数据库访问操作的动态监测框架（为开源项目，项目首页：www.p6spy.com）它使得数据库数据可无缝截取和操纵，而不必对现有应用程序的代码作任何修改。P6Spy 分发包包括P6Log，它是一个可记录任何 Java 应用程序的所有JDBC事务的应用程序。其配置完成使用时，可以进行数据访问性能的监测。

我们最需要的功能，查看sql语句，不是预编译的带问号的哦，而是真正的数据库执行的sql，更直观，更简单

### pom文件

```xml
        <!-- sql性能分析插件 -->
        <dependency>
            <groupId>p6spy</groupId>
            <artifactId>p6spy</artifactId>
            <version>3.9.1</version>
        </dependency>
```

### 数据库连接配置

修改连接驱动名称和url地址为p6spy格式

```yml
spring:
  datasource:
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver
    url: jdbc:p6spy:mysql://localhost:3306/demo?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8&autoReconnect=true&rewriteBatchedStatements=true
    password: root
    username: 123456
```

### 自定义日志打印文件

```java
import com.p6spy.engine.spy.appender.MessageFormattingStrategy;

import java.text.SimpleDateFormat;
import java.util.Date;

public class P6SpyLogger implements MessageFormattingStrategy {

    private SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS");

    /**
     * 自定义sql日志打印
     *
     * @param connectionId 连接标识
     * @param now          执行时间
     * @param elapsed      执行秒数ms
     * @param category     statement
     * @param prepared     准备sql语句
     * @param sql          sql语句
     * @param s4           数据库url连接
     * @return {@link String}
     */
    @Override
    public String formatMessage(int connectionId, String now, long elapsed, String category, String prepared, String sql, String s4) {
        System.out.println();
        if (!"".equals(sql.trim())) {
            String sqlBegin = "============== SQL LOGGER BEGIN ==============";

            String sqlExecuteTime = "SQL 执行时间       :" + this.format.format(new Date()) + "\n";
            String elapsedStr = "SQL 执行毫秒       :" + elapsed + "ms" + "\n";
            String sqlPrint = "SQL 执行语句       :" + sql;

            //String sqlPrint = !"".equals(sql.trim()) ? this.format.format(new Date()) + " | took " + elapsed + "ms | " + category + " | connection " + connectionId + "\n " + sql + ";" : "";
            String sqlEnd = "==============  SQL LOGGER END  ==============";

            return sqlBegin + "\r\n" + sqlExecuteTime + elapsedStr + sqlPrint + "\r\n" + sqlEnd;
        }
        return "";
    }
}
```

### 配置文件

> 指定自定义日志打印的位置

```properties
# p6spy 性能分析插件配置文件
modulelist=com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory,com.p6spy.engine.outage.P6OutageFactory
# 自定义日志打印
#logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger
logMessageFormat=com.yolo.framework.extend.p6spy.P6SpyLogger
#日志输出到控制台
appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger
# 使用日志系统记录 sql
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
# 设置 p6spy driver 代理
#deregisterdrivers=true
# 取消JDBC URL前缀
useprefix=true
# 配置记录 Log 例外,可去掉的结果集有error,info,batch,debug,statement,commit,rollback,result,resultset.
excludecategories=info,debug,result,commit,resultset
# 日期格式
dateformat=yyyy-MM-dd HH:mm:ss
# SQL语句打印时间格式
databaseDialectTimestampFormat=yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
#driverlist=org.h2.Driver
# 是否开启慢SQL记录
outagedetection=true
# 慢SQL记录标准 2 秒
outagedetectioninterval=2
# 是否过滤 Log
filter=true
# 过滤 Log 时所排除的 sql 关键字，以逗号分隔
exclude=SELECT 1
```

![image-20230810101548063](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230810101548063.png)