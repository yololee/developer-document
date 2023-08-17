# springboot-整合knife4j

Knife4j是一款基于Swagger 2的在线API文档框架

```
在Spring Boot中，使用此框架时，需要：

- 添加依赖
- 在配置文件（`application.properties`）中开启增强模式
- 编写配置类（代码相对固定，建议CV）
```

## Spring Boot版本兼容性

![image-20230817141205524](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230817141205524.png)

在4.0版本之前，应用坐标：

引用组件maven坐标如下：

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>{<4.0.0版本}</version>
</dependency>
```

自4.0版本开始，maven组件的`artifactId`已经发生了变化。

引用组件maven坐标如下：

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi2-spring-boot-starter</artifactId>
    <version>{maven仓库最新版本}</version>
</dependency>
```

## 引入knife4j

**添加依赖**

```xml
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-spring-boot-starter</artifactId>
            <version>2.0.9</version>
        </dependency>
```

**创建 Swagger 配置依赖**

```yml
springdoc:
  info:
    # 标题
    title: '标题：FastAdmin后台管理系统_接口文档'
    # 描述
    description: '描述：用于管理集团旗下公司的人员信息,具体包括XXX,XXX模块...'
    termsOfServiceUrl: "127.0.0.1:8888"
    # 版本
    version: '版本号: 1.1.0'
    # 作者信息
    contact:
      name: yolo
      email: 2936412130@qq.com
      url: https://gitee.com/huanglei1111
# Knife4j接口文档
knife4j:
  enable: true
  # 开启生产环境屏蔽
  production: false
  basic:
    enable: false
    username: admin
    password: 123456
```

```java
package com.yolo.framework.config.properties;

import io.swagger.v3.oas.models.info.License;
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.NestedConfigurationProperty;
import org.springframework.stereotype.Component;
import springfox.documentation.service.Contact;

@Data
@Component
@ConfigurationProperties(prefix = "springdoc")
public class SpringDocProperties {

    /**
     * 文档基本信息
     */
    @NestedConfigurationProperty
    private InfoProperties info = new InfoProperties();



    /**
     * <p>
     * 文档的基础属性信息
     * </p>
     *
     * @see io.swagger.v3.oas.models.info.Info
     *
     * 为了 springboot 自动生产配置提示信息，所以这里复制一个类出来
     */
    @Data
    public static class InfoProperties {

        /**
         * 标题
         */
        private String title = null;

        /**
         * 描述
         */
        private String description = null;

        /**
         * url服务地址
         */
        private String termsOfServiceUrl = null;

        /**
         * 联系人信息
         */
        @NestedConfigurationProperty
        private Contact contact = null;

        /**
         * 许可证
         */
        @NestedConfigurationProperty
        private License license = null;

        /**
         * 版本
         */
        private String version = null;

    }
}

```

```java
package com.yolo.framework.config;

import com.yolo.framework.config.properties.SpringDocProperties;
import io.swagger.annotations.Api;



import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2WebMvc;

@Configuration
@EnableSwagger2WebMvc
@RequiredArgsConstructor
public class Knife4jConfiguration {

    private final SpringDocProperties springDocProperties;

    @Bean(value = "defaultApi2")
    public Docket defaultApi2() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(new ApiInfoBuilder()
                        .title(springDocProperties.getInfo().getTitle())
                        .description(springDocProperties.getInfo().getDescription())
                        .termsOfServiceUrl(springDocProperties.getInfo().getTermsOfServiceUrl())
                        .contact(springDocProperties.getInfo().getContact())
                        .version("1.0")
                        .build())
                //分组名称
                .groupName(springDocProperties.getInfo().getVersion())
                .select()
                //这里指定Controller扫描包路径
                .apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
                //这里指定Controller扫描包路径
//                .apis(RequestHandlerSelectors.basePackage("com.github.xiaoymin.knife4j.controller"))
                .paths(PathSelectors.any())
                .build();
    }
}
```

## 注解

```java
@Api(tags = {“用户操作”})
加在controller类上
tags表示该类的标签，在页面会独立显示一个菜单
 
@ApiOperation(value = “保存用户”, notes = “保存时，ID由数据库生成，无需填写，有则忽略”, tags = “保存”)
加在相应的请求处理方法上
value表示该方法的说明
notes相当于对该方法的详细说明，也就是更加完整的描述
tags 表示标签，，在页面会独立显示一个菜单
 
@ApiImplicitParam(name = “id”, value = “用户ID”, defaultValue = “1”)
方法只有一个基本类型参数时加在方法上。方法有多个参数时加在@ApiImplicitParams内
name 参数中属性的名字
value 对这个属性的描述
defaultValue 默认值，这个还是有必要填写的，在页面进行请求时，会自动填充
 
@ApiImplicitParams(value = {})
用在请求方法上
这个注解必须和@ApiImplicitParam配合使用
当请求方法中的请求参数很多的时候，例如saveUser(String username, Integer age, Date birthday, String phone)
 
@ApiParam(value = “当前页”, defaultValue = “1”)
加在请求方法的普通参数上
value的值是对该参数的说明
与@ApiImplicitParam使用的效果等同，根据个人喜好进行使用
 
@ApiModel(value = “用户信息”)
加在请求方法的对象类上
value 对该对象参数的描述
例如有一个请求方法save(UserDTO userDTO), 则需要加在UserDTO这个类上面(可以参照下面的示例)
 
@ApiModelProperty(value = “用户ID”, example = “1”)
加在请求方法的参数对象的属性上
value 对该属性的描述
example 属性的示例值，在页面会自动填充该值
 
 
@ApiIgnore：注解类、参数、方法，注解后将不在Swagger UI中显示
```

## 案例

```java
package com.yolo.knife4j.controller;

import cn.hutool.core.collection.ListUtil;
import com.yolo.knife4j.base.ResultVo;
import com.yolo.knife4j.dto.UserAddRequest;
import com.yolo.knife4j.vo.StudentVO;
import com.yolo.knife4j.vo.UserVO;
import io.swagger.annotations.*;
import org.springframework.web.bind.annotation.*;
 
import javax.validation.Valid;
import java.util.Date;

@ApiResponses(value = {
        @ApiResponse(code = 200, message = "接口返回成功状态"),
        @ApiResponse(code = 500, message = "接口返回未知错误，请联系开发人员调试")
})
@Api(tags = "用户")
@RestController
@RequestMapping("/user")
public class UserController {


    @ApiOperation(value = "保存用户", notes = "简单传参")
    @PostMapping("/add")
    public ResultVo<Object> add(@RequestBody @Valid UserAddRequest userAddRequest) {
        return ResultVo.builder().build().setCode(200).setSuccess(true)
                .setTime(new Date()).setMsg("保存用户成功").setData(userAddRequest);
    }

    @ApiOperation(value = "保存用户2", notes = "复杂传参")
    @PostMapping("/add2")
    public ResultVo<Object> add2(@RequestBody @Valid UserAddRequest userAddRequest) {
        return ResultVo.builder().build().setCode(200).setSuccess(true)
                .setTime(new Date()).setMsg("保存用户成功").setData(userAddRequest);
    }


    @GetMapping("/list")
    @ApiOperation(value = "查找用户列表", notes = "根据id查找单个用户")
    public ResultVo<Object> list(@RequestParam @ApiParam(value = "当前页", defaultValue = "1") Integer pageNum,
                          @RequestParam @ApiParam(value = "页大小", defaultValue = "10") Integer pageSize) {

        UserVO vo = new UserVO();
        StudentVO studentVO = StudentVO.builder().build().setAddress("wuhan").setCode(2);
        vo.setStudentVOS(ListUtil.of(studentVO));

        return ResultVo.builder().build().setCode(200).setSuccess(true)
                .setTime(new Date()).setMsg("查找用户列表").setData(vo);
    }

    @DeleteMapping("/{id}")
    @ApiOperation(value = "删除用户", notes = "删除后无法恢复")
    @ApiImplicitParam(name = "id", value = "用户ID", defaultValue = "1")
    public ResultVo<Object> delete(@PathVariable(name = "id") Long id) {
        return ResultVo.builder().build().setCode(200).setSuccess(true)
                .setTime(new Date()).setMsg("删除用户");
    }
}
```

**响应参数**

```java
package com.yolo.knife4j.base;

import com.fasterxml.jackson.annotation.JsonFormat;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.Accessors;

import java.io.Serializable;
import java.util.Date;

@Data
@Builder
@Accessors(chain = true)
@ApiModel("响应参数")
@AllArgsConstructor
@NoArgsConstructor
public class ResultVo<T> implements Serializable {

    private static final long serialVersionUID = -8054007511410819665L;

    @ApiModelProperty(value = "响应状态码", example = "1", dataType = "Integer")
    private int code;

    // 是否成功标识.true表示成功,false表示失败
    @ApiModelProperty("success标识,true表示成功,false表示失败")
    private boolean success;

    // 操作成功时需要响应给客户端的响应数据
    @ApiModelProperty("响应信息")
    private String msg;


    @ApiModelProperty("响应数据")
    private T data;

    @ApiModelProperty("当前时间")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date time;
}
```

> tips:  http://127.0.0.1:8080/doc.html
>
> 这里端口，就是你运行项目的端口

![image-20230331155517709](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230331155517709.png)

## knife4j增强功能

springboot 中 knife4j的完整参数如下：

```yml
knife4j:
  enable: true
  documents:
    -
      group: 2.X版本
      name: 接口签名
      locations: classpath:sign/*
  setting:
    language: zh-CN
    enableSwaggerModels: true
    enableDocumentManage: true
    swaggerModelName: 实体类列表
    enableVersion: false
    enableReloadCacheParameter: false
    enableAfterScript: true
    enableFilterMultipartApiMethodType: POST
    enableFilterMultipartApis: false
    enableRequestCache: true
    enableHost: false
    enableHostText: 192.168.0.193:8000
    enableHomeCustom: true
    homeCustomLocation: classpath:markdown/home.md
    enableSearch: false
    enableFooter: false
    enableFooterCustom: true
    footerCustomContent: Apache License 2.0 | Copyright  2019-[浙江八一菜刀股份有限公司](https://gitee.com/xiaoym/knife4j)
    enableDynamicParameter: false
    enableDebug: true
    enableOpenApi: false
    enableGroup: true
  cors: false
  production: false
  basic:
    enable: false
    username: test
    password: 12313
```

### 接口添加作者

添加作者有俩种方式

1. 在方法上使用注解 `@ApiOperationSupport(author = "yolo-test")`
2. 在controller类上使用注解 `@ApiSupport(author = "yolo-controller")`

如果在方法上使用了注解，并且也在类上使用了注解，那么最后的展示结果以方法上的注解为准

![image-20230331160425656](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230331160425656.png)

### 资源屏蔽

当我们在生成环境的时候不想显示接口文档，由于 Knife4j 基于 Servlet 体系提供了过滤 Filter 功能，所以就不需要我们再去造轮子了，直接使用即可

```yml
knife4j:
  # 开启增强配置
  enable: true
  # 是否开启生产环境屏蔽   true:关闭swagger，false:开启swagger
  production: true
```

然后重启项目

![image-20230331160832987](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230331160832987.png)

### 访问页面加权控制

针对Swagger的资源接口,Knife4j提供了简单的Basic认证功能

简单点说，指定一个用户名和密码，访问 Swagger 文档需要验证登录名和密码，验证通过之后才能正常访问

如果用户开启了 basic （knife4j.basic.enable = true）认证功能，但是没有指定 username 和password，那么 knife4j 提供了一组默认的用户名密码

```java
admin/123321
```

```yml
knife4j:
  # 开启增强配置
  enable: true
  # 是否开启生产环境屏蔽   true:关闭swagger，false:开启swagger
  production: false
  basic:
    # 是否开启认证
    enable: true
    # Basic认证用户名
    username: admin
    # Basic认证密码
    password: 123456
```

> 如果开启生产环境屏蔽了，开启basic认证是不生效的

![image-20230331161440718](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230331161440718.png)

### 接口排序

@ApiOperationSupport注解中增加了 order 字段,用于接口排序。在使用此注解之前需要开启增强功能

### 分组排序

分组，顾名思义，就是多个 controller 之间的排序，开发者可以通过注解实现每个controller 之间的排序，实现这个功能的注解一共有三个，具体如下

**@ApiSupport**

```java
@RestController
@RequestMapping(value = "/test")
@ApiSupport(author = "yolo-controller",order = 999)
@Api(tags = "测试swagger")
public class Knife4jTestController
```

**@ApiSort**

```text
@RestController
@RequestMapping(value = "/test")
@ApiSort(value = 999)
@Api(tags = "测试swagger")
public class Knife4jTestController
```

**@Api**

```text
@RestController
@RequestMapping(value = "/test")
@Api(tags = "测试swagger",position = 999)
public class Knife4jTestController
```

> Tips:
>
> 这三个注解是存在优先级的，也就是说，当同时使用时，只会有一个注解生效，所以在使用的时候需要特别注意。优先级规则如下
>
> @ApiSupport > @ApiSort>@Api

### 请求参数缓存

我们在调试接口的时候，有的接口会有很多参数，当我们好不容易填好了所有的参数，由于我们不小心关闭了页面，下次再调试的时候发现还需要再次将参数输入一遍，心态会爆炸吧，所以 knife4j 在文档管理中增加了一个选项：开启请求参数缓存

![image-20230331162345564](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230331162345564.png)

> Tips:
>
> 俩种情况会失效
>
> 1、 @ApiModelProperty 注解中添加 example （属性的示例值）属性，那么， knife4j 将不会使用缓存，使用的是后端指定的 example
>
> 2、当域名发生改变时，所有缓存将会失效

### 过滤请求参数

我们在开发过程中，经常会遇到这样的一个问题，新增和修改接口，修改接口需要传递修改的记录id，但是新增则不需要，而后端往往会将修改和新增的入参对象设置为一个对象，那么这个对象中必然会存在 id 字段，这就会对新增造成误导

所以，knife4j 支持了请求参数的过滤（忽略），实现方式也是非常的简单，使用自定义增强注解ApiOperationSupport中的ignoreParameters属性,可以强制忽略要显示的参数

这里表单和json格式过滤参数是不一样的

**我们先看表单格式**

```java
@ApiModel("用户信息")
@Getter
@Setter
@ToString
public class UserDTO {
    @ApiModelProperty(value = "用户id")
    private Long id;
    @ApiModelProperty(value = "用户名",example = "李雷")
    private String username;
    @ApiModelProperty(value = "性别",example = "男")
    private String gender;
    @ApiModelProperty(value = "手机号码",example = "18888888888")
    private String phone;
    @ApiModelProperty(value = "用户收货地址信息")
    private UserAddressDTO userAddressDTO;
}

@Getter
@Setter
@ToString
public class UserAddressDTO {
    @ApiModelProperty(value = "收获地址的记录id")
    private Long id;
    
    @ApiModelProperty(value = "省")
    private String province;
    @ApiModelProperty(value = "市")
    private String city;
    @ApiModelProperty(value = "区")
    private String district;
    @ApiModelProperty(value = "详细地址")
    private String addr;
}



    @PostMapping(value = "/saveUser")
    @ApiOperation("新增用户信息-表单")
    @ApiOperationSupport(author = "yolo",ignoreParameters = {"id","userAddressDTO.id"})
    public String saveUser(UserDTO userDTO){
        System.out.println("前端传递的用户信息："+ userDTO);
        return "save success";
    }
    @PostMapping(value = "/updateUser")
    @ApiOperation("编辑用户信息")
    @ApiOperationSupport(author = "yolo")
    public String updateUser( UserDTO userDTO){
        System.out.println("前端传递的用户信息："+ userDTO);
        return "edit success";
    }
```

> 在过滤字段的时候，第一层我们只要填写对象中的属性名即可，但如果需要过滤第二层，根据忽略规则中的第二条，我们在 UserDTO 对象中引入 UserAddressDTO 对象：private UserAddressDTO userAddressDTO; 我们还需要忽略 UserAddressDTO 对象中的 id 属性，那么需要填上 userAddressDTO.id ，其中 userAddressDTO 要与 UserDTO 对象中的 UserAddressDTO 属性名一致

新增操作没有id

![image-20230331163601535](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230331163601535.png)

编辑操作

![image-20230331163636414](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230331163636414.png)

**JSON格式忽略**

专业说法是：实例名.属性名，以新增用户为例，我们需要过滤用户id，那么写法就是：userDTO.id，其中 userDTO 为 saveUser() 的 参数名

```java
    @PostMapping(value = "/saveUser")
    @ApiOperation("新增用户信息")
    @ApiOperationSupport(author = "yolo",ignoreParameters = {"userDTO.id","userDTO.userAddressDTO.id"})
    public String saveUser(@RequestBody UserDTO userDTO){
        System.out.println("前端传递的用户信息："+ userDTO);
        return "save success";
    }
```

### 禁用调试

```yml
knife4j:
  enable: true
  setting:
    enableDebug: false
```

> enableDebug:该属性是一个Boolean值，代表是否启用调试功能,默认值为true(代表开启调试)，如果要禁用调试，该值设为false
> 同样，此操作也需要开发者在创建Docket逻辑分组对象时，通过Knife4j提供的工具对象OpenApiExtensionResolver将扩展属性进行赋值

![image-20230331165449490](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230331165449490.png)

### 禁用搜索框

发者如果想要禁用Ui界面中的搜索功能，需要通过增强属性进行配置，此功能需要开启增强功能

```yml
knife4j:
  enable: true
  setting:
    enableSearch: false
```

> enableSearch:该属性是一个Boolean值，代表是否启用搜索功能,默认值为true(代表开启搜索)，如果要禁用搜索，该值设为false
> 同样，此操作也需要开发者在创建Docket逻辑分组对象时，通过Knife4j提供的工具对象OpenApiExtensionResolver将扩展属性进行赋值。具体的代码实现请参考禁用调试和自定义主页内容，我这里就不重复了。

> [Gitee项目地址（demo-knife4j）](https://gitee.com/huanglei1111/yolo-springboot-demo/tree/master/demo-knife4j)
>
> [knife4j官方文档地址](https://doc.xiaominfo.com/docs/quick-start/start-knife4j-version)

