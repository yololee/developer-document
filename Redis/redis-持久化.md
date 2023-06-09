# Redis的持久化

Redis 是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。所以 Redis 提供了持久化功能  

## RDB（Redis DataBase）  

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里  

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失  

### 执行时机

RDB持久化在四种情况下会执行：

1. 执行save命令
2. 执行bgsave命令
3. Redis停机时
4. 触发RDB条件时

 启用快照 (rdb触发机制)

> RDB默认开启，可不用改

redis.conf

```shell
# 触发时间策略  
# 格式：save <seconds> <changes>
# 可设置多个
# 900秒内，如果超过1个key被修改，则发起快照保存  执行的是`bgsave`命令
save 900 1
# 300秒内，如果超过10个key被修改，则发起快照保存
save 300 10
# 60秒内，如果1万个key被修改，则发起快照保存
save 60 10000


# 文件名称
dbfilename dump.rdb

# 文件保存路径，AOF文件同样存放在此目录下。默认为当前工作目录。
dir ./

# 如果持久化出错，主进程是否停止写入操作，yes->停止 => 为了保护持久化的数据一致性问题
stop-writes-on-bgsave-error yes

# 是否压缩
rdbcompression yes

# 导入时是否检查
rdbchecksum yes
```

会产生一个`dump.rdb`文件

### 禁用快照

redis.conf

```shell
save ""
```

### 手动生成快照

#### SAVE

使用同步的方式生成RDB快照文件，在这个过程中会阻塞所有其他客户端的请求，直到RDB文件被创建完毕。 -- 线上应该禁止使用

```shell
save
```

#### BGSAVE

```shell
bgsave
# 查看操作是否成功
lastsave
```

- fork主进程得到一个子进程，共享内存空间
- 子进程读取内存数据并写入新的RDB文件
- 用新RDB文件替换旧的RDB文件

fork采用的是copy-on-write技术：

- 当主进程执行读操作时，访问共享内存；
- 当主进程执行写操作时，则会拷贝一份数据，执行写操作。

![image-20230609141009846](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230609141009846.png)

#### 如何恢复  

1、将备份文件（dump.rdb）移动到redis安装目录并启动服务即可
2、CONFIG GET dir 获取目录  

```shell
127.0.0.1:6379> config get dir
dir
/usr/local/bin
```

#### 优点和缺点  

优点：  

1、适合大规模的数据恢复
2、对数据完整性和一致性要求不高  

缺点：  

1、在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改
2、Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑  

## AOF（Append Only File） 

以日志的形式来记录每个写操作，将Redis执行过的所有指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作 

aof保存的是 appendonly.aof 文件   

### 启用AOF

> AOF默认关闭

redis.conf

```shell
# 是否开启aof
appendonly yes

# 文件存放目录，与RDB共用。默认为当前工作目录。
dir ./

# 7.0新版本新增的存放文件夹
appenddirname "appendonlydir"

# 文件名称
appendfilename "appendonly.aof"


# 同步方式
# always：   表示每执行一次写命令，立即记录到AOF文件
# everysec： 写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件 （默认方案）
# no：       写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘。一般而言为了提高效率，操作系统会等待缓存区被填满，才会开始同步数据到磁盘
appendfsync everysec
```

### AOF重写

> 将原来多条命令压缩成一条命令，减少aof文件大的问题
> `set name hello`&`set name zhengqingya`&`set age 18` => `mset set name zhengqingya age 18`

#### 自动重写触发配置

redis.conf

```shell
# Redis会记住自从上一次重写后AOF文件的大小（如果自Redis启动后还没重写过，则记住启动时使用的AOF文件的大小）。
# AOF文件对比上次文件 增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# 禁用自动的日志重写功能 -- 百分比设置为0
# auto-aof-rewrite-percentage 0
# AOF文件体积最小多大以上才触发重写 
auto-aof-rewrite-min-size 64mb

# aof重写期间是否同步
no-appendfsync-on-rewrite no
```

#### AOF 启动/修复/恢复  

正常恢复：  

- 启动：设置Yes，修改默认的appendonly no，改为yes
- 将有数据的aof文件复制一份保存到对应目录（config get dir）
- 恢复：重启redis然后重新加载  

异常恢复：  

- 启动：设置Yes
- 故意破坏 appendonly.aof 文件！
- 修复： redis-check-aof --fix appendonly.aof 进行修复
- 恢复：重启 redis 然后重新加载  

#### Rewrite  

AOF 采用文件追加方式，文件会越来越大，为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis 就会启动AOF 文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用命令 bgrewriteaof ！  

**重写原理：**

AOF 文件持续增长而过大时，会fork出一条新进程来将文件重写（也是先写临时文件最后再
rename），遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，这点和快照有点类似！  

**触发机制：  **

Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的已被且文件大于64M的触发。  

### 优点和缺点  

**优点：**  

1、每修改同步：appendfsync always 同步持久化，每次发生数据变更会被立即记录到磁盘，性能较差但数据完整性比较好
2、每秒同步： appendfsync everysec 异步操作，每秒记录 ，如果一秒内宕机，有数据丢失
3、不同步： appendfsync no 从不同步  

**缺点：**

1、相同数据集的数据而言，aof 文件要远大于 rdb文件，恢复速度慢于 rdb。
2、Aof 运行效率要慢于 rdb，每秒同步策略效率较好，不同步效率和rdb相同    

## 总结

1、RDB 持久化方式能够在指定的时间间隔内对你的数据进行快照存储

2、AOF 持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis 协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。

3、只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化  

4、同时开启两种持久化方式  

- 在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
- RDB 的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢？作者建议不要，因为RDB更适合用于备份数据库（AOF在不断变化不好备份），快速重启，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。

5、性能建议  

- 因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留 save 900 1 这条规则。
- 如果Enable AOF ，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了，代价一是带来了持续的IO，二是AOF rewrite 的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。
- 如果不Enable AOF ，仅靠 Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔IO，也减少了rewrite时带来的系统波动。代价是如果Master/Slave 同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB文件，载入较新的那个，微博就是这种架构  