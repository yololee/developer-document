# MySQL-数据不存在插入，存在则更新

建表语句

```mysql
CREATE TABLE `image` (
  `id` bigint(20) NOT NULL COMMENT '主键',
  `cloud_resource_pool_id` bigint(20) NOT NULL COMMENT '云资源池id',
  `os_type` varchar(20) DEFAULT NULL COMMENT '系统镜像类型',
  `image_name` varchar(200) DEFAULT NULL COMMENT '镜像名称',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `removed` tinyint(2) DEFAULT NULL COMMENT '删除标记',
  `delete_time` datetime DEFAULT NULL COMMENT ' 删除时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `poolId_type_name` (`cloud_resource_pool_id`,`os_type`,`image_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**DUPLICATE**

```mysql
INSERT INTO 表名(唯一索引列, 列2, 列3) VALUE(值1, 值2, 值3) ON DUPLICATE KEY UPDATE 列=值, 列=值
```

**REPLACE INTO**

这个是替换，删除原有的，然后在插入新数据

```mysql
REPLACE INTO 表名称(列1, 列2, 列3) VALUES(值1, 值2, 值3)
```

**mp中sql**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="tm.multicloud.mapper.ImageMapper">

    <insert id="insertOrUpdateBatch">
        INSERT ignore INTO image (id,cloud_resource_pool_id,os_type,image_name,create_time,update_time,removed)
        VALUES
        <foreach collection="list" separator="," item="item">
            (#{item.id},#{item.cloudResourcePoolId},#{item.osType},#{item.imageName},
            #{item.createTime},#{item.updateTime},#{item.removed}
            )
        </foreach>
        ON DUPLICATE KEY
        UPDATE cloud_resource_pool_id = VALUES(cloud_resource_pool_id),
        os_type = VALUES(os_type),
        image_name = VALUES(image_name),
        update_time=NOW()
    </insert>

    <insert id="insertOrUpdate">
        insert
        into image (id, cloud_resource_pool_id, os_type, image_name, create_time, update_time, removed)
        values (#{id}, #{cloudResourcePoolId}, #{osType}, #{imageName},#{createTime}, #{updateTime}, #{removed})
        on duplicate key
        update
            cloud_resource_pool_id = #{cloudResourcePoolId},
            os_type  = #{osType},
            image_name = #{imageName}
    </insert>

    <insert id="insertOrUpdateBatch2">
        replace into image (id,cloud_resource_pool_id,os_type,image_name,create_time,update_time,removed) VALUES
        <foreach collection="list" separator="," item="item">
            (#{item.id},#{item.cloudResourcePoolId},#{item.osType},#{item.imageName},
            #{item.createTime},#{item.updateTime},#{item.removed}
            )
        </foreach>
    </insert>


</mapper>
```

