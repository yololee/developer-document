## Redis事务

### redis事务的概念

redis事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化，在事务执行过程，会按照顺序序列化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中

总结来说，redis事务介绍一次性、顺序性，排他性的执行一个队列中的一系列命令

### redis事务没有隔离级别的概念

批量操作在发送`exec`命令前被放入队列缓存，并不会被实际执行

### redis不保证原子性

redis中，单条命令是原子性执行的，但事务不保证原子性，且没有回滚，事务中任意命令执行失败，其余的命令仍会被执行

### redis事务的三个阶段

- 开始事务
- 命令入列
- 执行事务

### redis事务相关命令

```shell
watch key1 key2 ... #监视一或多个key,如果在事务执行之前，被监视的key被其他命令改动，则
事务被打断 （ 类似乐观锁 ）
multi # 标记一个事务块的开始（ queued ）
exec # 执行所有事务块的命令 （ 一旦执行exec后，之前加的监控锁都会被取消掉 ）
discard # 取消事务，放弃事务块中的所有命令
unwatch # 取消watch对所有key的监控
```

### 命令实践

> 正常执行

```shell
127.0.0.1:6379> multi			# 开启事务
OK
127.0.0.1:6379> set k1 v1       # 命令入队
QUEUED
127.0.0.1:6379> set k2 v2       # 命令入队
QUEUED
127.0.0.1:6379> get k2			# 命令入队
QUEUED
127.0.0.1:6379> set k3 v3		# 命令入队
QUEUED
127.0.0.1:6379> exec			# 执行事务
1) OK
2) OK
3) "v2"				# 输出结果
4) OK
```

> 放弃事务  

```shell
127.0.0.1:6379> multi			# 开启事务
OK
127.0.0.1:6379> set k1 v1		# 命令入队
QUEUED
127.0.0.1:6379> set k2 v22		# 命令入队
QUEUED
127.0.0.1:6379> set k3 v33		# 命令入队
QUEUED
127.0.0.1:6379> discard			#取消事务
OK
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379> get k3
```

> 若在事务队列中存在命令性错误（类似于java编译性错误），则执行EXEC命令时，所有命令都不会执行  

```shell
127.0.0.1:6379> multi			#开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> getssfa k3		#错误命令
(error) ERR unknown command 'getssfa'
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> set k5 v5
QUEUED
127.0.0.1:6379> exec			#执行事务出错
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get v4          #命令未执行
(nil)
```

> 若在事务队列中存在语法性错误（类似于java的1/0的运行时异常），则执行EXEC命令时，其他正确命令会被执行，错误命令抛出异常。  

```shell
127.0.0.1:6379> multi     		#开启事务
OK
127.0.0.1:6379> incr k1			#命令入队
QUEUED
127.0.0.1:6379> set k2 v22		#命令入队
QUEUED	
127.0.0.1:6379> set k3 v33		#命令入队
QUEUED
127.0.0.1:6379> set k4 v4		#命令入队
QUEUED
127.0.0.1:6379> get k4			#命令入队
QUEUED
127.0.0.1:6379> exec			#执行事务
1) (error) ERR value is not an integer or out of range	#第一条命令报错其他正常执行
2) OK
3) OK
4) OK
5) "v4"
```

> Watch 监控  

<b>悲观锁：  </b>

悲观锁(Pessimistic Lock),顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿到这个数据就会block直到它拿到锁。传统的关系型数据库里面就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在操作之前先上锁  

<b>乐观锁：</b>

乐观锁(Optimistic Lock),顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁。但是在更新的时候会判断一下再此期间别人有没有去更新这个数据，可以使用版本号等机制，乐观锁适用于多读的应用类型，这样可以提高吞吐量，乐观锁策略：提交版本必须大于记录当前版本才能执行更新  

<b>测试 </b>

1、初始化信用卡可用余额和欠额  

```shell
127.0.0.1:6379> set balance 100
OK
127.0.0.1:6379> set debt 0
OK
```

2、使用watch检测balance，事务期间balance数据未变动，事务执行成功  

```shell
127.0.0.1:6379> watch balance
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> decrby balance 20
QUEUED
127.0.0.1:6379> incrby debt 20
QUEUED
127.0.0.1:6379> exec
1) (integer) 80
2) (integer) 20
```

3、使用watch检测balance，事务期间balance数据变动，事务执行失败！  

```shell
# 窗口一
127.0.0.1:6379> watch balance
OK
127.0.0.1:6379> MULTI # 执行完毕后，执行窗口二代码测试
OK
127.0.0.1:6379> decrby balance 20
QUEUED
127.0.0.1:6379> incrby debt 20
QUEUED
127.0.0.1:6379> exec # 修改失败！
(nil)
# 窗口二
127.0.0.1:6379> get balance
"80"
127.0.0.1:6379> set balance 200
OK
# 窗口一：出现问题后放弃监视，然后重来！
127.0.0.1:6379> UNWATCH # 放弃监视
OK
127.0.0.1:6379> watch balance
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> decrby balance 20
QUEUED
127.0.0.1:6379> incrby debt 20
QUEUED
127.0.0.1:6379> exec # 成功！
1) (integer) 180
2) (integer) 40
```

**说明：**
一但执行 EXEC 开启事务的执行后，无论事务使用执行成功， WARCH 对变量的监控都将被取消。
故当事务执行失败后，需重新执行WATCH命令对变量进行监控，并开启新的事务进行操作  

<b>小结 :</b> 

watch指令类似于乐观锁，在事务提交时，如果watch监控的多个KEY中任何KEY的值已经被其他客户端更改，则使用EXEC执行事务时，事务队列将不会被执行，同时返回Nullmulti-bulk应答以通知调用者事务执行失败  