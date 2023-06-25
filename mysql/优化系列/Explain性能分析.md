# Explain性能分析

## 一、简介

### 1.1 概念

查看执行计划：使用 EXPLAIN 关键字可以模拟优化器执行 SQL 查询语句，从而知道 MySQL 是如何处理 SQL 语句的。分析查询语句或是表结构的性能瓶颈。

### 1.2 作用

- 表的读取顺序 
- 数据读取操作的操作类型 
- 哪些索引可以使用 
- 哪些索引被实际使用 
- 表之间的引用 
- 每张表有多少行被优化器查询

### 1.3 语法

> Explain + SQL 语句

<font color ='red'>执行计划包含的信息（重点） ：| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |</font>


面试重点：<font color ='red'>id、type、key、rows、Extra</font>

例子：

```sql
Explain select a.*, ad.* from test_a as a left join test_a_description as ad on a.id=ad.parent_id;
```

![image-20230620140500920](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620140500920.png)


## 二、名词解释

### id（表的读取顺序）

select 查询的序列号，包含一组数字，表示查询中执行 select 子句或操作表的顺序

- id 相同，执行顺序由上至下

![image-20230620143705298](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620143705298.png)

- id 不同，如果是子查询，id 的序号会递增，id 值越大优先级越高，越先被执行

![image-20230620143728109](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620143728109.png)

- id 有相同也有不同：id 如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id 值 越大，优先级越高，越先执行

![image-20230620143902942](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620143902942.png)

> id 号每个号码，表示一趟独立的查询。一个 sql 的查询趟数越少越好

### select_type （数据读取操作的操作类型）

代表查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询，取值

- simple：简单的 select 查询，查询中不包含子查询或者 UNION
- primary：查询中若包含任何复杂的子部分，最外层查询则被标记为 primary
- derived：在 FROM 列表中包含的子查询被标记为 DERIVED (衍生)，MySQL 会递归执行这些 子查询, 把结果放在临时表里
- subquery：在 SELECT 或 WHERE 列表中包含了子查询
- depedent subquery：在 SELECT 或 WHERE 列表中包含了子查询，子查询基于外层
- uncacheable subquery：无法使用缓存的子查询
- union：若第二个 SELECT 出现在 UNION 之后，则被标记为 UNION；若 UNION 包含在 FROM 子句的子查询中，外层 SELECT 将被标记为：DERIVED
- union result：从 UNION 表获取结果的 SELECT

### table（显示执行的表名）

这一列表示 explain 的一行正在访问哪个表。

当 from 子句中有子查询时，table列是 格式，表示当前查询依赖 id=N 的查询，于是先执行 id=N 的查询。 当有 union 时，UNION RESULT 的 table 列的值为<union1,2>，1和2表示参与 union 的 select 行id

### type（访问类型排列）

查询的访问类型。是较为重要的一个指标，结果值从最好到最坏依次是：

> system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

![image-20230620144129292](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620144129292.png)

> 只需要记住：system > const > eq_ref > ref > range > index > ALL 就行了，其他的不常见。

- system：表只有一行记录（等于系统表），这是 const 类型的特列，平时不会出现，这个也 可以忽略不计
- const：表示通过索引一次就找到了，const 用于比较 primary key 或者 unique 索引。因为 只匹配一行数据，所以很快。如将主键置于 where 列表中，MySQL 就能将该查询转换为一个 常量。

![image-20230620144447301](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620144447301.png)

- eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一 索引扫描

![image-20230620144640149](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620144640149.png)

- ref：非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，它返回 所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫 描的混合体

![image-20230620144706380](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620144706380.png)

- range：只检索给定范围的行，使用一个索引来选择行。key 列显示使用了哪个索引一般就是 在 where 语句中出现了 between、<、>、in 等的查询这种范围扫描索引扫描比全表扫描要 好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引

![image-20230620144821258](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620144821258.png)

- index：出现 index 是 sql 使用了索引但是没用索引进行过滤，一般是使用了覆盖索引或者是 利用索引进行了排序分组

![image-20230620144836470](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620144836470.png)

- all：将遍历全表以找到匹配的行

![image-20230620144859790](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620144859790.png)

- index_merge：在查询过程中需要多个索引组合使用，通常出现在有 or 关键字的 sql 中
- ref_or_null：对于某个字段既需要过滤条件，也需要 null 值的情况下。查询优化器会选择用 ref_or_null 连接查询
- index_subquery：利用索引来关联子查询，不再全表扫描
- unique_subquery：该联接类型类似于 index_subquery。子查询中的唯一索引

### possible_keys（哪些索引可以使用）

显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索 引，则该索引将被列出，但不一定被查询实际使用

### key（哪些索引被实际使用）

实际使用的索引。如果为 NULL，则没有使用索引

### key_len（消耗的字节数）

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好

key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。key_len 显示的值 为索引字段的最大可能长度，并非实际使用长度。如何计算 key_len？

- 先看索引上字段的类型 + 长度，比如：int=4; varchar(20)=20; char(20)=20
- 如果是 varchar 或者 char 这种字符串字段，视字符集要乘不同的值，比如 utf-8 要乘 3， GBK 要乘 2 
- varchar 这种动态字符串要加 2 个字节 
- 允许为空的字段要加 1 个字节

![image-20230620145233565](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620145233565.png)

### ref（表之间的引用）

显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上 的值。

![image-20230620145315989](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620145315989.png)

### rows（每张表有多少行被优化器查询）

显示 MySQL 认为它执行查询时必须检查的行数。越少越好！

![image-20230620145419852](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620145419852.png)

### 2.10 Extra

其他的额外重要的信息

- Using filesort：说明 mysql 会对数据使用一个外部的索引排序，而不是按照表内的索引顺序 进行读取。MySQL 中无法利用索引完成的排序操作称为“文件排序”。排序字段若通过索引去 访问将大大提高排序速度

![image-20230625103022775](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230625103022775.png)

- Using temporary：使用临时表保存中间结果，MySQL 在对查询结果排序时使用临时表。常 见于排序 order by 和分组查询 group by

![image-20230625103536207](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230625103536207.png)

- Using index：

  表示相应的select操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错！

情况一：

如果同时出现 using where，表明索引被用来执行索引键值的查找

![image-20230625104121602](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230625104121602.png)

情况二：

如果 没有同时出现 using where，表明索引只是用来读取数据而非利用索引执行查找

![image-20230625104239295](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230625104239295.png)

- Using where：表明使用了 where 过滤。
- Using join buffer：使用了连接缓存。

![image-20230625105000981](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230625105000981.png)

- impossible where：where 子句的值总是 false，不能用来获取任何数据

![image-20230625104658776](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230625104658776.png)

- select tables optimized away：在没有 group by 子句的情况下，基于索引优化 MIN/MAX 操 作或者对于 MyISAM 存储引擎优化 COUNT(*) 操作，不必等到执行阶段再进行计算，查询执 行计划生成的阶段即完成优化
- distinct：优化 distinct 操作，在找到第一匹配的元祖后即停止找同样值的动作