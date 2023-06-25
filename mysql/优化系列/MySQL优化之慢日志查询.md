# MySQL优化之慢日志查询

## 慢日志介绍

MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中

具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。long_query_time的默认值为10，意思是运行10秒以上的语句

由他来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒的sql，结合之前explain进行全面分析

### 慢查询日志相关参数

```mysql
show variables like '%slow_query%';
```

慢查询日志开关默认是关闭的

慢查询日志的路径默认在`/usr/local/mysql/data`下

![image-20230620112847571](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620112847571.png)

慢查询日志记录了包含所有执行时间超过参数long_query_time(单位：秒)所设置值的SQL语句的日志，在MySQL上用命令可以查看，如下：

```mysql
show variables like 'long%';
```

![image-20230620113244339](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620113244339.png)

这个值是可以修改的，如下：

```mysql
set long_query_time = 1;
```

![image-20230620113425457](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620113425457.png)

现在修改成超过1秒的SQL都会被记录在慢查询日志中！可以设置成0.01秒，表示10毫秒

### mysql运行状态查询

```mysql
# 当前时间
show status like 'uptime';
# 共执行多少次select
show stauts like 'com_select';
# 共执行多少次update
show stauts like 'com_update';
# 共执行多少次delete
show stauts like 'com_delete';
# 当前MySQL连接数
show status like 'connections';
# 显示慢查询次数
show status like 'slow_queries';
```

`show [session/global] status like ...` 如果你不写 [session/global] 默认是session会话，指取出当前窗口的执行，如果你想看所有(从mysql启动到现在，则应该global)

## 慢查询日志实践

### 打开慢查询日志开关

```mysql
# 查看慢日志参数
show variables like '%slow_query%';
# 开启慢日志(只对当前数据库生效，如果MySQL重启后则会失效)
set global slow_query_log = ON;
```

![image-20230620114211643](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620114211643.png)

![image-20230620114341220](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620114341220.png)

如果要永久生效，就必须修改配置文件my.cnf(其它系统变量也是如此)

修改my.cnf文件，[mysqld] 下增加或修改参数slow_query_log和slow_query_log_file后，然后重启MySQL服务器。也即将如下两行配置进my.cnf文件

```cnf
slow_query_log =1
slow_query_log_file=/var/lib/mysqatguigu-slow.log
```

关于慢查询的参数slow_query_log_file，它指定慢查询日志文件的存放路径，`系统默认会给一个缺省的文件host_name-slow.log`（如果没有指定参数slow_query_log_file的话）

### 设置合理的慢查询时间上限long_query_time

![image-20230620114800586](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620114800586.png)

查看另一个session，发现还是默认的10s，故long_query_time只影响当前session

![image-20230620114933878](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620114933878.png)

如果想要设置全局的,语法如下：

需要新开一个窗口进行查看

```mysql
set global long_query_time = 1;
```

### 压测执行各种业务

![image-20230620134143693](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620134143693.png)

### 查看慢查询日志

慢查询日志路径：`/usr/local/mysql/data`

![image-20230620134302506](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620134302506.png)

![image-20230620135448492](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620135448492.png)

### 用explain分析这些耗时的sql语句，从而针对性优化

![image-20230620135618995](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620135618995.png)

