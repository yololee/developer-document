## 导出

- 导出建库建表的SQL

```mysql
//导出所有库
mysqldump -u 用户名 -p --all-databases > ~/xxx.sql
//一次可以导出多个库
mysqldump -u 用户名 -p --databases db1[db2] > ~/xxx.sql
//导出库或者库里面的某张表
mysqldump -u 用户名 -p dbname [tablename]> ~/xxx.sql

```

- 导出纯数据

```mysql
mysql -u 用户名 -p -D school -e 'select * from user where age>10' > ~/user.txt
```

## 导入

登录mysql，在mysql的shell上执行下面语句

```mysql
# 后面是sql的路径
source ~/school.sql
```

