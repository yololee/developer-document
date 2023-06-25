## show profile 分析

```MySQL
# 查看
show variables like 'profiling';
# 开启
set profiling = on ;
```

`show [session/global] status like ...` 如果你不写 [session/global] 默认是session会话，指取出当前窗口的执行，如果你想看所有(从mysql启动到现在，则应该global)

![image-20230620135919216](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620135919216.png)

没有报错，说明profiling变量只影响当前session

> 查看结果

```mysql
# 查看结果
show profiles;
```

![image-20230620140034949](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620140034949.png)

> 诊断SQL，`show profile cpu,block io for query ID号;（ID号为第4步Query_ID列中数字）`

```mysql
show profile cpu,block io for query 5;
```

![image-20230620151220088](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620151220088.png)

参数备注（写在代码中）：show profile cpu,block io for query 3;（如此代码中的cpu,block）

- ALL：显示所有的开销信息。
- BLOCK IO：显示块lO相关开销。
- CONTEXT SWITCHES ：上下文切换相关开销。
- CPU：显示CPU相关开销信息。
- IPC：显示发送和接收相关开销信息。
- MEMORY：显示内存相关开销信息。
- PAGE FAULTS：显示页面错误相关开销信息。
- SOURCE：显示和Source_function，Source_file，Source_line相关的开销信息。
- SWAPS：显示交换次数相关开销的信息。

日常开发需要注意的结论（Status列中的出现此四个问题严重）

- converting HEAP to MyISAM：查询结果太大，内存都不够用了往磁盘上搬了。
- Creating tmp table：创建临时表，拷贝数据到临时表，用完再删除
- Copying to tmp table on disk：把内存中临时表复制到磁盘，危险!
- locked：锁了
  

