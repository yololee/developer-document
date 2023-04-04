# 返回List集合

原本响应结构

```json
[
  {
    "name": "test",
    "code": "persion"
  },
  {
    "name": "test",
    "code": "admin"
  }
]
```

期待返回结构

```json
[
  {
    "name": "test",
    "codeList": [
      "persion",
      "admin"
    ]
  }
]
```

使用collection嵌套list对象的时候，其它字段(ex:name)都要做映射才有值！

```java
public class SysRoleRePermListVO {
    @ApiModelProperty(value = "名称")
    private String name;

    @ApiModelProperty(value = "list")
    private List<String> codeList;
}
```

```
<resultMap id="listRoleRePerm" type="com.zhengqing.system.model.vo.SysRoleRePermListVO">
    <result property="name" column="name"/>
    <collection property="codeList" ofType="String" javaType="list">
        <result column="code"/>
    </collection>
</resultMap>

<select id="listRoleRePerm" resultMap="listRoleRePerm">
    SELECT sp.id,
           sp.name,
           r.code
    FROM t_sys_permission sp
    LEFT JOIN t_sys_role_permission srp ON sp.id = srp.permission_id
    LEFT JOIN t_sys_role r ON srp.role_id = r.role_id
</select>
```
