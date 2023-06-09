## 五大基本数据类型

### redis键(key)

```shell
# keys * 查看所有的key
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set name qinjiang
OK
127.0.0.1:6379> keys *
1) "name"

# exists key 的名字，判断某个key是否存在
127.0.0.1:6379> EXISTS name
(integer) 1
127.0.0.1:6379> EXISTS name1
(integer) 0

# move key db ---> 当前库就没有了，被移除了
127.0.0.1:6379> move name 1
(integer) 1
127.0.0.1:6379> keys *
(empty list or set)

# expire key 秒钟：为给定 key 设置生存时间，当 key 过期时(生存时间为 0 )，它会被自动删除。
# expireat key 时间戳
# pexpore key 毫秒值
# pexpireat key 时间戳以毫秒计算
# ttl key 查看还有多少秒过期，-1 表示永不过期，-2 表示已过期
127.0.0.1:6379> set name qinjiang
OK
127.0.0.1:6379> EXPIRE name 10
(integer) 1
127.0.0.1:6379> ttl name
(integer) 4
127.0.0.1:6379> ttl name
(integer) 3
127.0.0.1:6379> ttl name
(integer) 2
127.0.0.1:6379> ttl name
(integer) 1
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> keys *
(empty list or set)

# type key 查看你的key是什么类型
127.0.0.1:6379> set name qinjiang
OK
127.0.0.1:6379> get name
"qinjiang"
127.0.0.1:6379> type name
string
```

### 字符串类型(String)

- String是redis最基本的类型，你可以理解成Memcached一模一样的类型，一个key对应一个value。
- String类型是二进制安全的，意思是redis的string可以包含任何数据，比如jpg图片或者序列化的对象。
- String类型是redis最基本的数据类型，一个redis中字符串value最多可以是512M  

> 简单命令说明

<font color='red'><B>单值单Value </B></font>

```shell
# ===================================================
# set、get、del、append、strlen
# ===================================================
127.0.0.1:6379> set key1 value1 # 设置值
OK
127.0.0.1:6379> get key1 # 获得key
"value1"
127.0.0.1:6379> del key1 # 删除key
(integer) 1
127.0.0.1:6379> keys * # 查看全部的key
(empty list or set)
127.0.0.1:6379> exists key1 # 确保 key1 不存在
(integer) 0
127.0.0.1:6379> append key1 "hello" # 对不存在的 key 进行 APPEND ，等同于 SET
key1 "hello"
(integer) 5 # 字符长度
127.0.0.1:6379> APPEND key1 "-2333" # 对已存在的字符串进行 APPEND
(integer) 10 # 长度从 5 个字符增加到 10 个字符
127.0.0.1:6379> get key1
"hello-2333"
127.0.0.1:6379> STRLEN key1 # # 获取字符串的长度
(integer) 10

# ===================================================
# incr、decr 一定要是数字才能进行加减，+1 和 -1。
# incrby、decrby 命令将 key 中储存的数字加上指定的增量值。
# ===================================================
127.0.0.1:6379> set views 0 # 设置浏览量为0
OK
127.0.0.1:6379> incr views # 浏览 + 1
(integer) 1
127.0.0.1:6379> incr views # 浏览 + 1
(integer) 2
127.0.0.1:6379> decr views # 浏览 - 1
(integer) 1
127.0.0.1:6379> incrby views 10 # +10
(integer) 11
127.0.0.1:6379> decrby views 10 # -10
(integer) 1

# ===================================================
# range [范围]
# getrange 获取指定区间范围内的值，类似between...and的关系，从零到负一表示全部
# ===================================================
127.0.0.1:6379> set key2 abcd123456 # 设置key2的值
OK
127.0.0.1:6379> getrange key2 0 -1 # 获得全部的值
"abcd123456"
127.0.0.1:6379> getrange key2 0 2 # 截取部分字符串
"abc"

# ===================================================
# setrange 设置指定区间范围内的值，格式是setrange key值 具体值
# ===================================================
127.0.0.1:6379> get key2
"abcd123456"
127.0.0.1:6379> SETRANGE key2 1 xx # 替换值
(integer) 10
127.0.0.1:6379> get key2
"axxd123456"

# ===================================================
# setex（set with expire）键秒值
# setnx（set if not exist）
# ===================================================
127.0.0.1:6379> setex key3 60 expire # 设置过期时间
OK
127.0.0.1:6379> ttl key3 # 查看剩余的时间
(integer) 55
127.0.0.1:6379> setnx mykey "redis" # 如果不存在就设置，成功返回1
(integer) 1
127.0.0.1:6379> setnx mykey "mongodb" # 如果存在就设置，失败返回0
(integer) 0
127.0.0.1:6379> get mykey
"redis"

# ===================================================
# mset Mset 命令用于同时设置一个或多个 key-value 对。
# mget Mget 命令返回所有(一个或多个)给定 key 的值。
# 如果给定的 key 里面，有某个 key 不存在，那么这个 key 返回特殊值 nil 。
# msetnx 当所有 key 都成功设置，返回 1 。
# 如果所有给定 key 都设置失败(至少有一个 key 已经存在)，那么返回 0 。原子操作 
#===================================================
127.0.0.1:6379> mset k10 v10 k11 v11 k12 v12
OK
127.0.0.1:6379> keys *
1) "k12"
2) "k11"
3) "k10"
127.0.0.1:6379> mget k10 k11 k12 k13
1) "v10"
2) "v11"
3) "v12"
4) (nil)
127.0.0.1:6379> msetnx k10 v10 k15 v15 # 原子性操作！
(integer) 0
127.0.0.1:6379> get key15
(nil)

# 传统对象缓存
set user:1 value(json数据)
# 可以用来缓存对象
mset user:1:name zhangsan user:1:age 2
mget user:1:name user:1:age

# ===================================================
# getset（先get再set）
# ===================================================
127.0.0.1:6379> getset db mongodb # 没有旧值，返回 nil
(nil)
127.0.0.1:6379> get db
"mongodb"
127.0.0.1:6379> getset db redis # 返回旧值 mongodb
"mongodb"
127.0.0.1:6379> get db
"redis"
```

### Hash（哈希，类似 Java里的Map)

- Redis hash 是一个键值对集合。
- Redis hash 是一个String类型的field和value的映射表，hash特别适合用于存储对象。
- 类似Java里面的Map<String,Object>  

> 常用命令

<font color='red'><B>kv模式不变，但V是一个键值对     </B></font>

```shell
# ===================================================
# hset、hget 命令用于为哈希表中的字段赋值 。
# hmset、hmget 同时将多个field-value对设置到哈希表中。会覆盖哈希表中已存在的字段。
# hgetall 用于返回哈希表中，所有的字段和值。
# hdel 用于删除哈希表 key 中的一个或多个指定字段
# ===================================================
127.0.0.1:6379> hset myhash field1 "kuangshen"
(integer) 1
127.0.0.1:6379> hget myhash field1
"kuangshen"
127.0.0.1:6379> HMSET myhash field1 "Hello" field2 "World"
OK
127.0.0.1:6379> HGET myhash field1
"Hello"
127.0.0.1:6379> HGET myhash field2
"World"
127.0.0.1:6379> hgetall myhash
1) "field1"
2) "Hello"
3) "field2"
4) "World"
127.0.0.1:6379> HDEL myhash field1
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "field2"
2) "World"

# ===================================================
# hlen 获取哈希表中字段的数量。
# ===================================================
127.0.0.1:6379> hlen myhash
(integer) 1
127.0.0.1:6379> HMSET myhash field1 "Hello" field2 "World"
OK
127.0.0.1:6379> hlen myhash
(integer) 2

# ===================================================
# hexists 查看哈希表的指定字段是否存在。
# ===================================================
127.0.0.1:6379> hexists myhash field1
(integer) 1
127.0.0.1:6379> hexists myhash field3
(integer) 0

# ===================================================
# hkeys 获取哈希表中的所有域（field）。
# hvals 返回哈希表所有域(field)的值。
# ===================================================
127.0.0.1:6379> HKEYS myhash
1) "field2"
2) "field1"
127.0.0.1:6379> HVALS myhash
1) "World"
2) "Hello"

# ===================================================
# hincrby 为哈希表中的字段值加上指定增量值。
# ===================================================
127.0.0.1:6379> hset myhash field 5
(integer) 1
127.0.0.1:6379> HINCRBY myhash field 1
(integer) 6
127.0.0.1:6379> HINCRBY myhash field -1
(integer) 5
127.0.0.1:6379> HINCRBY myhash field -10
(integer) -5

# ===================================================
# hsetnx 为哈希表中不存在的的字段赋值 。
# ===================================================
127.0.0.1:6379> HSETNX myhash field1 "hello"
(integer) 1 # 设置成功，返回 1 。
127.0.0.1:6379> HSETNX myhash field1 "world"
(integer) 0 # 如果给定字段已经存在，返回 0 。
127.0.0.1:6379> HGET myhash field1
"hello"
```

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。
存储部分变更的数据，如用户信息等  

### List（列表）  

Redis列表是简单的字符串列表，按照插入顺序排序，你可以添加一个元素到列表的头部（左边）或者尾部（右边）。**它的底层实际是个链表 !  **

> 常见命令

<font color='red'><B>单值多Value   </B></font>

```shell
# ===================================================
# Lpush：将一个或多个值插入到列表头部。（左）
# rpush：将一个或多个值插入到列表尾部。（右）
# lrange：返回列表中指定区间内的元素，区间以偏移量 START 和 END 指定。
# 其中 0 表示列表的第一个元素， 1 表示列表的第二个元素，以此类推。
# 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。
# ===================================================
127.0.0.1:6379> LPUSH list "one"
(integer) 1
127.0.0.1:6379> LPUSH list "two"
(integer) 2
127.0.0.1:6379> RPUSH list "right"
(integer) 3
127.0.0.1:6379> Lrange list 0 -1
1) "two"
2) "one"
3) "right"
127.0.0.1:6379> Lrange list 0 1
1) "two"
2) "one"

# ===================================================
# lpop 命令用于移除并返回列表的第一个元素。当列表 key 不存在时，返回 nil 。
# rpop 移除列表的最后一个元素，返回值为移除的元素。
# ===================================================
127.0.0.1:6379> Lpop list
"two"
127.0.0.1:6379> Rpop list
"right"
127.0.0.1:6379> Lrange list 0 -1
1) "one"

# ===================================================
# Lindex，按照索引下标获得元素（-1代表最后一个，0代表是第一个）
# ===================================================
127.0.0.1:6379> Lindex list 1
(nil)
127.0.0.1:6379> Lindex list 0
"one"
127.0.0.1:6379> Lindex list -1
"one"

# ===================================================
# llen 用于返回列表的长度。
# ===================================================
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> Lpush list "one"
(integer) 1
127.0.0.1:6379> Lpush list "two"
(integer) 2
127.0.0.1:6379> Lpush list "three"
(integer) 3
127.0.0.1:6379> Llen list # 返回列表的长度
(integer) 3

# ===================================================
# lrem key 根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。
# ===================================================
127.0.0.1:6379> lrem list 1 "two"
(integer) 1
127.0.0.1:6379> Lrange list 0 -1
1) "three"
2) "one"

# ===================================================
# Ltrim key 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
# ===================================================
127.0.0.1:6379> RPUSH mylist "hello"
(integer) 1
127.0.0.1:6379> RPUSH mylist "hello"
(integer) 2
127.0.0.1:6379> RPUSH mylist "hello2"
(integer) 3
127.0.0.1:6379> RPUSH mylist "hello3"
(integer) 4
127.0.0.1:6379> ltrim mylist 1 2
OK
127.0.0.1:6379> lrange mylist 0 -1
1) "hello"
2) "hello2"

# ===================================================
# rpoplpush 移除列表的最后一个元素，并将该元素添加到另一个列表并返回。
# ===================================================
127.0.0.1:6379> rpush mylist "hello"
(integer) 1
127.0.0.1:6379> rpush mylist "foo"
(integer) 2
127.0.0.1:6379> rpush mylist "bar"
(integer) 3
127.0.0.1:6379> rpoplpush mylist myotherlist
"bar"
127.0.0.1:6379> lrange mylist 0 -1
1) "hello"
2) "foo"
127.0.0.1:6379> lrange myotherlist 0 -1
1) "bar"

# ===================================================
# lset key index value 将列表 key 下标为 index 的元素的值设置为 value 。
# ===================================================
127.0.0.1:6379> exists list # 对空列表(key 不存在)进行 LSET
(integer) 0
127.0.0.1:6379> lset list 0 item # 报错
(error) ERR no such key
127.0.0.1:6379> lpush list "value1" # 对非空列表进行 LSET
(integer) 1
127.0.0.1:6379> lrange list 0 0
1) "value1"
127.0.0.1:6379> lset list 0 "new" # 更新值
OK
127.0.0.1:6379> lrange list 0 0
1) "new"
127.0.0.1:6379> lset list 1 "new" # index 超出范围报错
(error) ERR index out of range

# ===================================================
# linsert key before/after pivot value 用于在列表的元素前或者后插入元素。
# 将值 value 插入到列表 key 当中，位于值 pivot 之前或之后。
# ===================================================
redis> RPUSH mylist "Hello"
(integer) 1
redis> RPUSH mylist "World"
(integer) 2
redis> LINSERT mylist BEFORE "World" "There"
(integer) 3
redis> LRANGE mylist 0 -1
1) "Hello"
2) "There"
3) "World"
```

> 总结

- 它是一个字符串链表，left，right 都可以插入添加
- 如果键不存在，创建新的链表
- 如果键已存在，新增内容
- 如果值全移除，对应的键也就消失了
- 链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了。  

list就是链表，略有数据结构知识的人都应该能理解其结构。使用Lists结构，我们可以轻松地实现最新消息排行等功能。List的另一个应用就是消息队列，可以利用List的PUSH操作，将任务存在List中，然后工作线程再用POP操作将任务取出进行执行。Redis还提供了操作List中某一段的api，你可以直接查询，删除List中某一段的元素。
Redis的list是每个子元素都是String类型的双向链表，可以通过push和pop操作从列表的头部或者尾部添加或者删除元素，这样List即可以作为栈，也可以作为队列。  

### Set（集合） 

Redis的Set是String类型的无序集合，它是通过HashTable实现的 !  

> 常见命令

<font color='red'><B>单值多Value   </B></font>

```shell
# ===================================================
# sadd 将一个或多个成员元素加入到集合中，不能重复
# smembers 返回集合中的所有的成员。
# sismember 命令判断成员元素是否是集合的成员。
# ===================================================
127.0.0.1:6379> sadd myset "hello"
(integer) 1
127.0.0.1:6379> sadd myset "kuangshen"
(integer) 1
127.0.0.1:6379> sadd myset "kuangshen"
(integer) 0
127.0.0.1:6379> SMEMBERS myset
1) "kuangshen"
2) "hello"
127.0.0.1:6379> SISMEMBER myset "hello"
(integer) 1
127.0.0.1:6379> SISMEMBER myset "world"
(integer) 0

# ===================================================
# scard，获取集合里面的元素个数
# ===================================================
127.0.0.1:6379> scard myset
(integer) 2

# ===================================================
# srem key value 用于移除集合中的一个或多个成员元素
# ===================================================
127.0.0.1:6379> srem myset "kuangshen"
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "hello"

# ===================================================
# srandmember key 命令用于返回集合中的一个随机元素。
# ===================================================
127.0.0.1:6379> SMEMBERS myset
1) "kuangshen"
2) "world"
3) "hello"
127.0.0.1:6379> SRANDMEMBER myset
"hello"
127.0.0.1:6379> SRANDMEMBER myset 2
1) "world"
2) "kuangshen"
127.0.0.1:6379> SRANDMEMBER myset 2
1) "kuangshen"
2) "hello"

# ===================================================
# spop key 用于移除集合中的指定 key 的一个或多个随机元素
# ===================================================
127.0.0.1:6379> SMEMBERS myset
1) "kuangshen"
2) "world"
3) "hello"
127.0.0.1:6379> spop myset
"world"
127.0.0.1:6379> spop myset
"kuangshen"
127.0.0.1:6379> spop myset
"hello"

# ===================================================
# smove SOURCE DESTINATION MEMBER
# 将指定成员 member 元素从 source 集合移动到 destination 集合。
# ===================================================
127.0.0.1:6379> sadd myset "hello"
(integer) 1
127.0.0.1:6379> sadd myset "world"
(integer) 1
127.0.0.1:6379> sadd myset "kuangshen"
(integer) 1
127.0.0.1:6379> sadd myset2 "set2"
(integer) 1
127.0.0.1:6379> smove myset myset2 "kuangshen"
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "world"
2) "hello"
127.0.0.1:6379> SMEMBERS myset2
1) "kuangshen"
2) "set2"

# ===================================================
#- 数字集合类
#- 差集： sdiff
#- 交集： sinter
#- 并集： sunion
# ===================================================
127.0.0.1:6379> sadd key1 "a"
(integer) 1
127.0.0.1:6379> sadd key1 "b"
(integer) 1
127.0.0.1:6379> sadd key1 "c"
(integer) 1
127.0.0.1:6379> sadd key2 "c"
(integer) 1
127.0.0.1:6379> sadd key2 "d"
(integer) 1
127.0.0.1:6379> sadd key2 "e"
(integer) 1
127.0.0.1:6379> SDIFF key1 key2 # 差集
1) "a"
2) "b"
127.0.0.1:6379> SINTER key1 key2 # 交集
1) "c"
127.0.0.1:6379> SUNION key1 key2 # 并集
1) "a"
2) "b"
3) "c"
4) "e"
5) "d"
```

在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis还为集合提供了求交集、并集、差集等操作，可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中  

### Zset（sorted set：有序集合）  

- Redis zset 和 set 一样，也是String类型元素的集合，且不允许重复的成员。
- 不同的是每个元素都会关联一个double类型的分数。Redis正是通过分数来为集合中的成员进行从小到大的排序，zset的成员是唯一的，但是分数（Score）却可以重复。

> 常见命令

在set基础上，加一个score值。之前set是k1 v1 v2 v3，现在zset是 k1 score1 v1 score2 v2  

```shell
# ===================================================
# zadd 将一个或多个成员元素及其分数值加入到有序集当中。
# zrange 返回有序集中，指定区间内的成员
# ===================================================
127.0.0.1:6379> zadd myset 1 "one"
(integer) 1
127.0.0.1:6379> zadd myset 2 "two" 3 "three"
(integer) 2
127.0.0.1:6379> ZRANGE myset 0 -1
1) "one"
2) "two"
3) "three"

# ===================================================
# zrangebyscore 返回有序集合中指定分数区间的成员列表。有序集成员按分数值递增(从小到大)
次序排列。
# ===================================================
127.0.0.1:6379> zadd salary 2500 xiaoming
(integer) 1
127.0.0.1:6379> zadd salary 5000 xiaohong
(integer) 1
127.0.0.1:6379> zadd salary 500 kuangshen
(integer) 1

# Inf无穷大量+∞,同样地,-∞可以表示为-Inf。
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf # 显示整个有序集
1) "kuangshen"
2) "xiaoming"
3) "xiaohong"
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf withscores # 递增排列
1) "kuangshen"
2) "500"
3) "xiaoming"
4) "2500"
5) "xiaohong"
6) "5000"
127.0.0.1:6379> ZREVRANGE salary 0 -1 WITHSCORES # 递减排列
1) "xiaohong"
2) "5000"
3) "xiaoming"
4) "2500"
5) "kuangshen"
6) "500"
127.0.0.1:6379> ZRANGEBYSCORE salary -inf 2500 WITHSCORES # 显示工资 <=2500
的所有成员
1) "kuangshen"
2) "500"
3) "xiaoming"
4) "2500"

# ===================================================
# zrem 移除有序集中的一个或多个成员
# ===================================================
127.0.0.1:6379> ZRANGE salary 0 -1
1) "kuangshen"
2) "xiaoming"
3) "xiaohong"
127.0.0.1:6379> zrem salary kuangshen
(integer) 1
127.0.0.1:6379> ZRANGE salary 0 -1
1) "xiaoming"
2) "xiaohong"

# ===================================================
# zcard 命令用于计算集合中元素的数量。
# ===================================================
127.0.0.1:6379> zcard salary
(integer) 2
OK

# ===================================================
# zcount 计算有序集合中指定分数区间的成员数量。
# ===================================================
127.0.0.1:6379> zadd myset 1 "hello"
(integer) 1
127.0.0.1:6379> zadd myset 2 "world" 3 "kuangshen"
(integer) 2
127.0.0.1:6379> ZCOUNT myset 1 3
(integer) 3
127.0.0.1:6379> ZCOUNT myset 1 2
(integer) 2

# ===================================================
# zrank 返回有序集中指定成员的排名。其中有序集成员按分数值递增(从小到大)顺序排列。
# ===================================================
127.0.0.1:6379> zadd salary 2500 xiaoming
(integer) 1
127.0.0.1:6379> zadd salary 5000 xiaohong
(integer) 1
127.0.0.1:6379> zadd salary 500 kuangshen
(integer) 1
127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES # 显示所有成员及其 score 值
1) "kuangshen"
2) "500"
3) "xiaoming"
4) "2500"
5) "xiaohong"
6) "5000"
127.0.0.1:6379> zrank salary kuangshen # 显示 kuangshen 的薪水排名，最少
(integer) 0
127.0.0.1:6379> zrank salary xiaohong # 显示 xiaohong 的薪水排名，第三
(integer) 2

# ===================================================
# zrevrank 返回有序集中成员的排名。其中有序集成员按分数值递减(从大到小)排序。
# ===================================================
127.0.0.1:6379> ZREVRANK salary kuangshen
(integer) 2
127.0.0.1:6379> ZREVRANK salary xiaohong # 小红第一
(integer) 0
```

和set相比，sorted set增加了一个权重参数score，使得集合中的元素能够按score进行有序排列，比如一个存储全班同学成绩的sorted set，其集合value可以是同学的学号，而score就可以是其考试得分，这样在数据插入集合的时候，就已经进行了天然的排序。可以用sorted set来做带权重的队列，比如普通消息的score为1，重要消息的score为2，然后工作线程可以选择按score的倒序来获取工作任务。让重要的任务优先执行。  

排行榜应用，取TOP N操作 ！  