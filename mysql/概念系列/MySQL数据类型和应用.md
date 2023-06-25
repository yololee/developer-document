# MySQL数据类型和应用

## 一、数据类型

### 1.整型

| 类型                | 所占字节 | 范围(有符号)       | 范围(无符号) | 用途       |
| ------------------- | -------- | ------------------ | ------------ | ---------- |
| tinyint[unsigned]   | 1 Bytes  | (-128，127)        | (0，255)     | 小整数值   |
| smallint[unsigned]  | 2 Bytes  | (-2^15^，2^15^ -1) | (0，2^16^-1) | 大整数值   |
| mediumint[unsigned] | 3 Bytes  | (-2^23^，2^23^-1)  | (0，2^24^-1) | 大整数值   |
| int[unsigned]       | 4 Bytes  | (-2^31^，2^31^-1)  | (0，2^32^-1) | 大整数值   |
| bigint[unsigned]    | 8 Bytes  | (-2^63^，2^63^-1)  | (0，2^64^-1) | 极大整数值 |

> 注意：

- 默认整数类型是带符号的，即可以有正负值

```sql
# 带符号
create  table  zhengxing1(num1  int, num2  tinyint);

# 不带符号的设置如下
create  table  zhengxing2(num1  int  unsigned, num2  tinyint  unsigned);
```

- int(11)这里的11不表示限制 int 的长度为 11 位，而是`字符的显示宽度`无论你显示宽度设置为多少，int 类型能存储的最大值和最小值永远都是固定的

### 2.小数型

| 类型                      | 含义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| float[(m,d)] [unsigned]   | 单精度浮点型 8位精度(4字节) ，约7位有效数字，m总个数，d小数位 |
| double[(m,d)] [unsigned]  | 双精度浮点型 16位精度(8字节) ，约17位有效数字，m总个数，d小数位 |
| decimal[(m,d)] [unsigned] | 定点数m表示该小数的总的有效位数（最大65），d表示小数点的位数 |

### 3.日期类型

| 类型      | 所占字节 | 范围                                    | 格式                | 用途                     |
| --------- | -------- | --------------------------------------- | ------------------- | ------------------------ |
| date      | 3 Bytes  | 1000-01-01/9999-12-31                   | YYYY-MM-DD          | 日期值                   |
| time      | 3 Bytes  | '-838:59:59'/'838:59:59'                | HH:MM:SS            | 时间值或持续时间         |
| year      | 1 Bytes  | 1901/2155                               | YYYY                | 年份值                   |
| datetime  | 8 Bytes  | 1000-01-01 00:00:00/9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |
| timestamp | 4 Bytes  | 1970-01-01 00:00:00/2038                | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |

### 4.字符串类型

| 类型       | 所占字节        | 用途                            |
| ---------- | --------------- | ------------------------------- |
| char       | 0-255           | 定长字符串                      |
| varchar    | 0-65535         | 变长字符串                      |
| tinyblob   | 0-255           | 不超过 255 个字符的二进制字符串 |
| tinytext   | 0-255           | 短文本字符串                    |
| blob       | 0-65 535        | 二进制形式的长文本数据          |
| text       | 0-65 535        | 长文本数据                      |
| mediumblob | 0-16 777 215    | 二进制形式的中等长度文本数据    |
| mediumtext | 0-16 777 215    | 中等长度文本数据                |
| longblob   | 0-4 294 967 295 | 二进制形式的极大文本数据        |
| longtext   | 0-4 294 967 295 | 极大文本数据                    |

> 定长字符类型char：

适用于存储的字符长度为固定长度的字符，比如中国邮政编码，中国身份证号码，手机号码等。

设定形式：字段名称 char(字符个数)

特点是：

1，存储的字符长度固定，最长可设定为255个字符。

2，如果实际写入的字符不足设定长度，内部会自动用空格填充到设定的长度。

3，相对varchar类型，其存取速度更快。

> 变长字符类型varchar

适用于存储字符长度经常不确定的字符，比如姓名，用户名，标题，内容，等大多数场合的字符。

设定形式：字段名称 varchar(字符个数)

特点是：

1，存储的字符长度是写入的实际长度，但不超过设定的长度。最长可设定为65532（字节）

2，如果实际写入的字符不足设定的长度，就按实际的长度存储。

3，相对于char字符串，其存取速度相对更慢。

注意：

由于其最长的限制是字节数，因此存储中文和英文的实际字符个数是不同的；

- 英文：一个字符占一个字节；
- 中文（gbk编码）：一个字符占2个字节；
- 中文（utf8编码）：一个字符占3个字节；

> text长文本类型

适用于存储“较长的文本内容”，比如文章内容。最长可存储65535个字符。

设定形式：字段名称  text

注意：

- text类型的字段不能设置默认值。
- text类型虽然是字符类型，但不能设置长度
- text类型的数据不存在行中

> 注意：
>
> char(n) 和 varchar(n) 中括号中 n 代表字符的个数，并不代表字节个数，比如 CHAR(30) 就可以存储 30 个字符。
>
> 一个表中的行也有一个“最大字节长度的限制”，一行最多存储65532字节。

## 二、应用

### 1.tinyint类型(-128~127)

```mysql
create table user (num TINYINT);

insert INTO USER VALUES(66);

insert INTO USER VALUES(128);
```

![image-20230620103938833](../../../../../Library/Application Support/typora-user-images/image-20230620103938833.png)


发现只有一条数据，这是因为128越界了，不能插入

```mysql
# 创建一个无符号的tinyint(0~255)

create table user2 (num TINYINT UNSIGNED);

insert INTO USER2 VALUES(-5);
```

![image-20230620104006728](../../../../../Library/Application Support/typora-user-images/image-20230620104006728.png)


无符号的tinyint的范围是（0~255），所以-5是不可以插入user2的

### 2.float类型

语法：float[(m, d)] [unsigned] : m总长度，d指定小数位数

例子：`float(4,2),他的范围是-99.99~99.99`

![image-20230620104313710](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620104313710.png)


float也存在有符号和无符号。创建一个无符号的

![image-20230620104510005](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620104510005.png)


这里添加一个负数进去，会添加失败

### 3.decimal类型

语法：decimal(m, d) [unsigned]：定点数m指定长度，d表示小数点的位数

![image-20230620104524971](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620104524971.png)


### 4.时间日期类型

1. date:日期 `yyyy-mm-dd`，占用三字节
2. timestamp：时间戳，从1970年开始的` yyyy-MM-dd HH:mm:ss`格式和datetime完全一致，占用四字节
3. datetime 时间日期格式` yyyy-MM-dd HH:mm:ss`表示范围从1000到9999，占用八字节

![image-20230620104537084](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620104537084.png)


### 5.char类型

语法：char(L): 固定长度字符串，L是可以存储的长度，单位为字符（字母/汉字），最大长度值可以为255

![image-20230620104557297](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620104557297.png)


### 6.varchar类型

语法：varchar(L): 可变长度字符串，L表示字符长度，最大长度65535个字节

![image-20230620104610067](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620104610067.png)


虽说varchar的最大长度是65535，但是有1-2个字节是用来记录数据大小的，所以有效的字节是65533。

编码不同，varchar(L)里的最大值L是不同的。
utf8中，一个字符占3个字节，所以L = 65533/3 = 21844
gbk编码中一个字符占2个字节，所以L = 65533 / 2 = 32766

![image-20230620104623639](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230620104623639.png)

> 上面测试的是mysql5.5环境