### < trim>标签、< set>标签

MyBatis还提供了< trim>标签，我们可以通过自定义< trim>标签来定制< where>标签的功能。比如，和< where>标签等价的自定义 < trim>标

prefixOverrides 属性会忽略通过管道分隔的文本序列（注意此例中的空格也是必要的）。它的作用是移除所有指定在 prefixOverrides 属性中的内容，并且插入 prefix 属性中指定的内容。

使用自定义< trim>标签来定制< where>标签的功能，获取用户信息:

```xml
<!-- 查询用户信息 -->
<select id="queryUserTrim" parameterType="com.mybatis.po.UserParam" resultType="com..mybatis.po.User">
    SELECT * FROM tb_user
    <trim prefix="WHERE" prefixOverrides="AND |OR ">
        <if test="userId>0">
            and user_id = #{userId}
        </if>
        <if test="userName!=null and userName!=''">
            and user_name like '%${userName}%'
        </if>
        <if test="sex!=null and sex!=''">
            and sex = #{sex}
        </if>
    </trim>
</select>
```

在修改用户信息的SQL配置方法中，使用< set>标签过滤多余的逗号：

```xml
<!-- 修改用户信息 -->
<update id="updateUser" parameterType="com.pjb.mybatis.po.UserParam">
    UPDATE tb_user
    <set>
        <if test="userName != null">user_name=#{userName},</if>
        <if test="sex != null">sex=#{sex},</if>
        <if test="age >0 ">age=#{age},</if>
        <if test="blogUrl != null">blog_url=#{blogUrl}</if>
    </set>
    where user_id = #{userId}
</update>
```

这里，< set>标签会动态前置SET关键字，同时也会删掉无关的逗号，因为用了条件语句之后很可能就会在生成的SQL语句的后面留下这些逗号。因为用的是“if”元素，若最后一个“if”没有匹配上而前面的匹配上，SQL 语句的最后就会有一个逗号遗留。