# idea-热部署插件JRebel的激活和使用

## 一、安装JRebel

1、在IDEA中一次点击 File->Settings->Plugins->Brows Repositories
2、在搜索框中输入JRebel进行搜索
3、找到JRebel for intellij
4、install
5、安装好之后需要restart IDEA

![image-20230511144126816](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511144126816.png)

## 二、激活JRebel

> Windows

JRebel并非免费的插件，需要激活之后才能使用。
1、首先到github上去下载一个反向代理软件，我下载的是windows x64版本

[下载地址](https://github.com/ilanyu/ReverseProxy/releases/tag/v1.4)

![image-20230511144148208](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511144148208.png)

2、双击运行我们下载的程序

![image-20230511144205646](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511144205646.png)

> mac

搭建本地license server服务器。我直接使用别人提供的dokcer 镜像进行搭建，两条命令就搞定，首先得安装docker

```shell
docker pull ilanyu/golang-reverseproxy
docker run -d -p 8888:8888 ilanyu/golang-reverseproxy
```

> 按快捷键command + shift + fn + f9 就可以自动

3、在IDEA中一次点击 File->Settings->JRebel 并找到激活界面

![image-20230511144219683](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511144219683.png)


4、选择JRebel activated中的 connect to online licensing service
第一行输入 http://127.0.0.1:8888/自己生成的uuid
这里必须是uuid才可以通过验证
第二行输入正确的邮箱格式，例如：2936412130@qq.com
再点击以下change liense 按钮验证激活


> 这里的uuid需要自己生成
>
> uuid生成网站：https://www.uuidgenerator.net/version1

5、最后别忘了把JRebel设置为offline模式 点一下**work offline**

![image-20230511144249485](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511144249485.png)


## 三、相关设置
### 1.开启IDEA的自动编译（静态）
![image-20230511144301759](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511144301759.png)
### 2.开启IDEA的自动编译（动态）
设置 compiler.automake.allow.when.app.running
ctrl+shift+A 或者 help->find action…打开
搜索registry
找到 compiler.automake.allow.when.app.running 并✔

![image-20230511144324136](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511144324136.png)





