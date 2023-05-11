# idea-使用easyYapi插件导出yapi接口

## 一、安装插件easyYapi

安装easyYapi插件： 插件拦搜索easyYapi安装并重启idea

![image-20230511142713046](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142713046.png)


## 二、配置easyYapi

- window系统路径：File-Settings --->other Settings --->EasyApi

### Token方式

![image-20230511142727603](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142727603.png)


> server：yapi的地址
>
> tokens：当前要生成的controller文件所在的项目的名称=yapi上要导入项目的token

![image-20230511142742781](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142742781.png)


导出接口：

先选择要导入的controller文件，然后右键选择Export Api

![image-20230511142801125](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142801125.png)


点击绿色钩子后，会弹出一个输入框，让你输入Yapi对应项目下的Token

![image-20230511142812776](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142812776.png)


Export Yapi 出现输入token框，将上面步骤的token直接复制，点击ok，如下图所示：

![](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142812776.png)


控制台提示导入成功：

![image-20230511142846335](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142846335.png)


###  项目id方式

Yapi版本太低导致一直no found token 的解决办法：

在EasyApi配置中选中 loginMode，并将项目token改成项目id，如下图所示： 

![image-20230511142904233](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142904233.png)


项目id是yapi中项目的id，如下图所示：

![image-20230511142916384](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142916384.png)


之后在EasyApi下的Built-in config中加入鉴权逻辑，如下图所示：

> url中ip和端口改成自己yapi的地址
>
> body中的 email 和 password 是你登陆yapi的时候输入的email 和 password

```
yapi.export.before=groovy:```
    httpClient.request().url("http://127.0.0.1:3000/api/user/login")
                .method("POST")
                .contentType("application/json")
                .body(["email":"xxx@xxx.com","password":"*******"])
                .call();
​```
```
![image-20230511142948362](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142948362.png)

之后再次执行上述导出接口步骤，输入项目id，即可成功。

> [EasyYapi文档地址](http://easyyapi.com/documents/login_mode_yapi.html)

