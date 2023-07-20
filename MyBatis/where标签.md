### < where>标签、< if>标签

`if`：通过判断动态拼接sql语句，一般用于判断查询条件

当查询语句的查询条件由于输入参数的不同而无法确切定义时，可以使用< where>标签对来包裹需要动态指定的SQL查询条件，而在< where>标签对中，可以使用< if test="...">条件来分情况设置SQL查询条件

> 当使用<where>标签对包裹 if 条件语句时，将会忽略查询条件中的第一个and或or

```xml
<!-- 查询用户信息 -->
<select id="queryUserInfo" parameterType="com.mybatis.po.UserParam" resultType="com.mybatis.po.User">
    SELECT * FROM tb_user
    <where>
        <if test="userId > 0">
            and user_id = #{userId}
        </if>
        <if test="userName!= null and userName!=''">
            and user_name like '%${userName}%'
        </if>
        <if test="sex!=null and sex!=''">
            and sex = #{sex}
        </if>
    </where>
</select>
```

```java
@Select({"<script>" +
            " select * from tb_user " +
            "<where>" +
            "<if test = 'userId != null and userId !=\"\" '> " +
            "and user_Id = #{userId} " +
            "</if>" +
            "<if test = 'userPassword != null and userPassword !=\"\" '> " +
            "and user_password like CONCAT('%',#{userPassword},'%')" +
            "</if>" +
            "</where>" +
            "</script>"})
```