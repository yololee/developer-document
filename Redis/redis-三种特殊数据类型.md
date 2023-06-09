## 三种特殊数据类型

### GEO地理位置

> 简介

Redis 的 GEO 特性在 Redis 3.2 版本中推出， 这个功能可以将用户给定的地理位置信息储存起来， 并对这些信息进行操作。来实现诸如附近位置、摇一摇这类依赖于地理位置信息的功能。geo的数据类型为zset。

GEO 的数据结构总共有六个常用命令：geoadd、geopos、geodist、georadius、
georadiusbymember、gethash  

> 命令介绍

#### geoadd

```shell
# ===================================================
# 语法    
#	geoadd key longitude latitude member ...
#
# 将给定的空间元素(纬度、经度、名字)添加到指定的键里面。
# 这些数据会以有序集he的形式被储存在键里面，从而使得georadius和georadiusbymember这样的命令可以在之后通过位置查询取得这些元素。
# geoadd命令以标准的x,y格式接受参数,所以用户必须先输入经度,然后再输入纬度。
# geoadd能够记录的坐标是有限的:非常接近两极的区域无法被索引。
# 有效的经度介于-180-180度之间，有效的纬度介于-85.05112878 度至 85.05112878 度之间。当用户尝试输入一个超出范围的经度或者纬度时,geoadd命令将返回一个错误
# ===================================================
127.0.0.1:6379> geoadd china:city 116.23 40.22 北京
(integer) 1
127.0.0.1:6379> geoadd china:city 121.48 31.40 上海 113.88 22.55 深圳 120.21
30.20 杭州
(integer) 3
127.0.0.1:6379> geoadd china:city 106.54 29.40 重庆 108.93 34.23 西安 114.02
30.58 武汉
(integer) 3
```

#### geopos

```shell
# ===================================================
# 语法    
#	geopos key member [member...]
#
# 从key里返回所有给定位置元素的位置（经度和纬度）
# ===================================================
127.0.0.1:6379> geopos china:city 北京
1) "116.23000055551528931"
2) "40.2200010338739844"
127.0.0.1:6379> geopos china:city 上海 重庆
1) "121.48000091314315796"
2) "31.40000025319353938"
1) "106.54000014066696167"
2) "29.39999880018641676"
127.0.0.1:6379> geopos china:city 新疆
1) (nil)
```

#### geodis

```shell
# ===================================================
# 语法    
#	geodist key member1 member2 [unit]
#
# 返回两个给定位置之间的距离，如果两个位置之间的其中一个不存在,那么命令返回空值。
# 指定单位的参数unit必须是以下单位的其中一个：
# m表示单位为米
# km表示单位为千米
# mi表示单位为英里
# ft表示单位为英尺
# 如果用户没有显式地指定单位参数,那么geodist默认使用米作为单位。
#geodist命令在计算距离时会假设地球为完美的球形,在极限情况下,这一假设最大会造成0.5%的误差。
# ===================================================
127.0.0.1:6379> geodist china:city 北京 上海
"1088785.4302"
127.0.0.1:6379> geodist china:city 北京 上海 km
"1088.7854"
127.0.0.1:6379> geodist china:city 重庆 北京 km
"1491.6716"
```

#### georadius

```shell
georadius
# ===================================================
# 语法    
#	georadius key longitude latitude radius m|km|ft|mi [withcoord][withdist][withhash][asc|desc][count count]
#
# 以给定的经纬度为中心， 找出某一半径内的元素
# ===================================================
测试：重新连接 redis-cli，增加参数 --raw ，可以强制输出中文，不然会乱码

[root@kuangshen bin]# redis-cli --raw -p 6379
# 在 china:city 中寻找坐标 100 30 半径为 1000km 的城市
127.0.0.1:6379> georadius china:city 100 30 1000 km
重庆
西安
# withdist 返回位置名称和中心距离
127.0.0.1:6379> georadius china:city 100 30 1000 km withdist
重庆
635.2850
西安
963.3171

# withcoord 返回位置名称和经纬度
127.0.0.1:6379> georadius china:city 100 30 1000 km withcoord
重庆
106.54000014066696167
29.39999880018641676
西安
108.92999857664108276
34.23000121926852302

# withdist withcoord 返回位置名称 距离 和经纬度 count 限定寻找个数
127.0.0.1:6379> georadius china:city 100 30 1000 km withcoord withdist count
1 重庆
635.2850
106.54000014066696167
29.39999880018641676
127.0.0.1:6379> georadius china:city 100 30 1000 km withcoord withdist count
2 重庆
635.2850
106.54000014066696167
29.39999880018641676
西安
963.3171
108.92999857664108276
34.23000121926852302
```

#### georadiusbymember

```shell
# ===================================================
# 语法    
#	georadiusbymember key member radius m|km|ft|mi [withcoord][withdist][withhash][asc|desc][count count]

# 找出位于指定范围内的元素，中心点是由给定的位置元素决定
# ===================================================
127.0.0.1:6379> GEORADIUSBYMEMBER china:city 北京 1000 km
北京
西安
127.0.0.1:6379> GEORADIUSBYMEMBER china:city 上海 400 km
杭州
上海
```

#### geohash

```shell
# ===================================================
# 语法    
#	geohash key member [member...]

# Redis使用geohash将二维经纬度转换为一维字符串，字符串越长表示位置更精确,两个字符串越相似表示距离越近
# ===================================================
127.0.0.1:6379> geohash china:city 北京 重庆
wx4sucu47r0
wm5z22h53v0
127.0.0.1:6379> geohash china:city 北京 上海
wx4sucu47r0
wtw6sk5n300
```

#### zrem  

GEO没有提供删除成员的命令，但是因为GEO的底层实现是zset，所以可以借用zrem命令实现对地理位置信息的删除  

```shell
127.0.0.1:6379> geoadd china:city 116.23 40.22 beijin
1
127.0.0.1:6379> zrange china:city 0 -1 # 查看全部的元素
重庆
西安
深圳
武汉
杭州
上海
beijin
北京
127.0.0.1:6379> zrem china:city beijin # 移除元素
1
127.0.0.1:6379> zrem china:city 北京 # 移除元素
1 1
27.0.0.1:6379> zrange china:city 0 -1
重庆
西安
深圳
武汉
杭州
上海
```

### HyperLogLog  

> 简介

Redis 在 2.8.9 版本添加了 HyperLogLog 结构  

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比  

HyperLogLog则是一种算法，它提供了不精确的去重计数方案  

> 什么是基数？  

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。基数估计就是在误差可接受的范围内，快速计算基数  

> 基本命令

| 命令                                       | 描述                                                 |
| ------------------------------------------ | ---------------------------------------------------- |
| [PFADD key element [element ...]           | 添加指定元素到 HyperLogLog 中                        |
| [PFCOUNT key [key ...]                     | 返回给定 HyperLogLog 的基数估算值                    |
| [PFMERGE destkey sourcekey [sourcekey ...] | 将多个 HyperLogLog 合并为一个 HyperLogLog，并 集计算 |

```shell
127.0.0.1:6379> PFADD mykey a b c d e f g h i j
1 1
27.0.0.1:6379> PFCOUNT mykey
10
127.0.0.1:6379> PFADD mykey2 i j z x c v b n m
1 1
27.0.0.1:6379> PFMERGE mykey3 mykey mykey2
OK
127.0.0.1:6379> PFCOUNT mykey3
15
```

### BitMap  

> 简介

在开发中，可能会遇到这种情况：需要统计用户的某些信息，如活跃或不活跃，登录或者不登录；又如需要记录用户一年的打卡情况，打卡了是1， 没有打卡是0，如果使用普通的 key/value存储，则要记录365条记录，如果用户量很大，需要的空间也会很大，所以 Redis 提供了 Bitmap 位图这中数据结构，Bitmap 就是通过操作二进制位来进行记录，即为 0 和 1；如果要记录 365 天的打卡情况，使用 Bitmap表示的形式大概如下：0101000111000111...........................，这样有什么好处呢？当然就是节约内存了，365 天相当于 365 bit，又 1 字节 = 8 bit , 所以相当于使用 46 个字节即可。

BitMap 就是通过一个 bit 位来表示某个元素对应的值或者状态, 其中的 key 就是对应元素本身，实际上底层也是通过对字符串的操作来实现。Redis 从 2.2 版本之后新增了setbit, getbit, bitcount 等几个bitmap 相关命令  

#### setbit  

`SETBIT key offset value` : 设置 key 的第 offset 位为value (1或0)  

```shell
# 使用 bitmap 来记录上述事例中一周的打卡记录如下所示：
# 周一：1，周二：0，周三：0，周四：1，周五：1，周六：0，周天：0 （1 为打卡，0 为不打卡）
127.0.0.1:6379> setbit sign 0 1
0
127.0.0.1:6379> setbit sign 1 0
0
127.0.0.1:6379> setbit sign 2 0
0
127.0.0.1:6379> setbit sign 3 1
0
127.0.0.1:6379> setbit sign 4 1
0
127.0.0.1:6379> setbit sign 5 0
0
127.0.0.1:6379> setbit sign 6 0
0
```

#### getbit  

`GETBIT key offset `获取offset设置的值，未设置过默认返回0  

```shell
127.0.0.1:6379> getbit sign 3 # 查看周四是否打卡
1
127.0.0.1:6379> getbit sign 6 # 查看周七是否打卡
0
```

#### bitcount  

`bitcount key [start, end] `统计 key 上位为1的个数  

```shell
# 统计这周打卡的记录，可以看到只有3天是打卡的状态：
127.0.0.1:6379> bitcount sign
3
```