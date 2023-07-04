# mac中Go安装和环境配置

## 一、安装

安装地址：https://golang.google.cn/dl/

![image-20230629165326408](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230629165326408.png)

下载好之后，傻瓜式安装

## 二、添加环境变量

```shell
vim ~/.bash_profile
```

```shell
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH:.
```


添加后需要执行：

```shell
source ~/.bash_profile
```

检查是否安装成功

```shell
go version
```

![image-20230629165552432](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230629165552432.png)