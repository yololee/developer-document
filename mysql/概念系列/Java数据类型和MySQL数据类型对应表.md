# Java数据类型和MySql数据类型对应表

| 类型名称                   | 数据库类型         | JAVA类型                               |
| -------------------------- | ------------------ | -------------------------------------- |
| Int,Integer [(M)]          | Integer            | java.lang.Integer                      |
| Int,Integer [(M)] unsigned | Integer unsigned   | java.lang.Long                         |
| bigint [(M)]               | bigint             | java.lang.Long                         |
| bigint [(M)] unsigned      | bigint unsigned    | java.math.BigInteger                   |
| float [(M,D)]              | float              | java.lang.Float                        |
| double [(M,B)] [unsigned]  | double             | java.lang.Double（不管是否带符号）     |
| decimal [(M,B)] [unsigned] | decimal            | java.math.BigDecimal（不管是否带符号） |
| char (M)                   | char               | java.lang.String                       |
| varchar (M)                | varchar            | java.lang.String                       |
| tinyint (1)                | tinyint            | java.lang.Boolean                      |
| tinyint (any) [unsigned]   | tinyint [unsigned] | java.lang.Integer                      |
| date                       | date               | java.sql.Date                          |
| datetime                   | datetime           | java.time.LocalDateTime                |
| timestamp [(m)]            | timestamp          | java.sql.Timestamp                     |
| time                       | time               | java.sql.Time                          |
| text                       | text               | java.lang.String                       |
| blob                       | blob               | byte[]                                 |

[官方mysql数据类型对应java数据类型关系表](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-type-conversions.html)

