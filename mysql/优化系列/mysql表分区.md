# mysql表分区

一般情况下我们创建的表对应一组存储文件，使用`MyISAM`存储引擎时是一个`.MYI`和`.MYD`文件，使用`Innodb`存储引擎时是一个`.ibd`和`.frm`（表结构）文件。

<font color ='red'>当数据量较大时（一般千万条记录级别以上），MySQL的性能就会开始下降，这时我们就需要将数据分散到多组存储文件，保证其单个文件的执行效率。</font>

最常见的分区方案是按`id`分区，如下将`id`的哈希值对10取模将数据均匀分散到10个`.ibd`存储文件中：

```mysql
create table article(
	id int auto_increment PRIMARY KEY,
	title varchar(64),
	content text
)PARTITION by HASH(id) PARTITIONS 10
```

![image-20230625135031512](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230625135031512.png)

> <font color = 'blue'>服务端的表分区对于客户端是透明的，客户端还是照常插入数据，但服务端会按照分区算法分散存储数据</font>

## MySQL提供的分区算法

分区依据的字段必须是主键的一部分，分区是为了快速定位数据，因此该字段的搜索频次较高应作为强检索字段，否则依照该字段分区毫无意义

### hash(field)

相同的输入得到相同的输出。输出的结果跟输入是否具有规律无关。<font color='red'>仅适用于整型字段</font>

```mysql
create table article(
	id int auto_increment PRIMARY KEY,
	title varchar(64),
	content text
)PARTITION by HASH(id) PARTITIONS 10
```

### key(field)

和`hash(field)`的性质一样，只不过`key`是<font color ='red'>处理字符串的</font>，比`hash()`多了一步从字符串中计算出一个整型在做取模操作

```mysql
create table article_key(
	id int auto_increment,
	title varchar(64),
	content text,
	PRIMARY KEY (id,title)	-- 要求分区依据字段必须是主键的一部分
)PARTITION by KEY(title) PARTITIONS 10
```

### range算法

是一种<font color ='red'>条件分区</font>算法，按照数据大小范围分区（将数据使用某种条件，分散到不同的分区中）

如下，按文章的发布时间将数据按照2018年8月、9月、10月分区存放：

```mysql
create table article_range(
	id int auto_increment,
	title varchar(64),
	content text,
	created_time int,	-- 发布时间到1970-1-1的毫秒数
	PRIMARY KEY (id,created_time)	-- 要求分区依据字段必须是主键的一部分
)charset=utf8
PARTITION BY RANGE(created_time)(
	PARTITION p201808 VALUES less than (1535731199),	-- select UNIX_TIMESTAMP('2018-8-31 23:59:59')
	PARTITION p201809 VALUES less than (1538323199),	-- 2018-9-30 23:59:59
	PARTITION p201810 VALUES less than (1541001599)	-- 2018-10-31 23:59:59
)
```

![image-20230625140647004](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230625140647004.png)

注意：条件运算符只能使用<font color = 'red'>less than</font>，这以为着较小的范围要放在前面，比如上述`p201808,p201819,p201810`分区的定义顺序依照`created_time`数值范围从小到大，不能颠倒。

```mysql
insert into article_range values(null,'MySQL优化','内容示例',1535731180);
flush tables;	-- 使操作立即刷新到磁盘文件
```

由于插入的文章的发布时间`1535731180`小于`1535731199`（`2018-8-31 23:59:59`），因此被存储到`p201808`分区中，这种算法的存储到哪个分区取决于数据状况。

### list算法

也是一种条件分区，按照列表值分区（`in (值列表)`）

```mysql
create table article_list(
	id int auto_increment,
	title varchar(64),
	content text,
	status TINYINT(1),	-- 文章状态：0-草稿，1-完成但未发布，2-已发布
	PRIMARY KEY (id,status)	-- 要求分区依据字段必须是主键的一部分
)charset=utf8
PARTITION BY list(status)(
	PARTITION writing values in(0,1),	-- 未发布的放在一个分区	
	PARTITION published values in (2)	-- 已发布的放在一个分区
)
```

## 分区管理语法

### range/list

#### 增加分区

文中我们尝试使用`range`对文章按照月份归档，随着时间的增加，我们需要增加一个月份：

```mysql
alter table article_range add partition(
	partition p201811 values less than (1543593599)	-- select UNIX_TIMESTAMP('2018-11-30 23:59:59')
	-- more
);
```

![image-20230625142126103](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230625142126103.png)

#### 删除分区

```mysql
alter table article_range drop PARTITION p201808
```

注意：删除分区后，分区中原有的数据也会随之删除！

### key/hash

#### 新增分区

```mysql
alter table article_key add partition partitions 4
```

#### 销毁分区

```mysql
alter table article_key coalesce partition 6
```

`key/hash`分区的管理不会删除数据，但是每一次调整（新增或销毁分区）都会将所有的数据重写分配到新的分区上。效率极低，最好在设计阶段就考虑好分区策略

## 分区的使用

当数据表中的数据量很大时，分区带来的效率提升才会显现出来。

只有检索字段为分区字段时，分区带来的效率提升才会比较明显。因此，分区字段的选择很重要，并且业务逻辑要尽可能地根据分区字段做相应调整（尽量使用分区字段作为查询条件）。