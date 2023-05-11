# idea-远程调试服务器代码

## 一、前提

- 服务器代码和本地代码保持一致
- 服务器网络跟本地网络相通

## 二、IDEA设置

![image-20230511141736623](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511141736623.png)


![image-20230511141753202](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511141753202.png)


## 三、java-jar 启动测试

- 上传服务器

![image-20230511141809548](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511141809548.png)


- 服务器启动

![image-20230511141826507](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511141826507.png)


- 本地启动

![image-20230511141842820](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511141842820.png)


- postman发请求

![image-20230511141909674](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511141909674.png)

- 查看本地代码

![image-20230511141937138](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511141937138.png)


## 四、docker 启动测试

### 开启docker配置远程访问

系统配置：

系统：centos 7
Docker version 19.03.6   

```shell
# 修改docker.service支持远程访问
vim /usr/lib/systemd/system/docker.service
```

![image-20230511142011233](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142011233.png)


```shell
# 通知docker服务做出的修改
systemctl daemon-reload
# 重启docker
systemctl restart docker
```

测试：

```shell
# 输入下面命令，出现如下图显示则配置正确
curl http://localhost:2375/version
```

![image-20230511142033896](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142033896.png)


### 构建镜像,启动容器

编写dockerFile文件

```shell
FROM java:8
ADD debug-test-0.0.1-SNAPSHOT.jar debug-test.jar
EXPOSE 8085
EXPOSE 5005
ENTRYPOINT ["java","-jar","-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005","/debug-test.jar"]

# Add 这一行，后面必须取别名，不然会失败
# EXPOSE 暴露的端口 一个项目启动的端口，一个远程debug的端口
```

构建镜像

```shell
docker build -f ./debug-test-file -t debug-test .

# 注意最后的 . 表示 debug-test-file 文件在当前目录下
# debug-test 镜像的名称
```

启动容器

```shell
docker run  -d --restart=always --name debug-test -p 8085:8080 -p 5005:5005 debug-test

# 这里5005必须要暴露出去，不然远程debug会连接失败
```

![image-20230511142107262](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142107262.png)


### 测试

本地项目启动

![image-20230511142121411](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142121411.png)

postman发送请求

![image-20230511142222620](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142222620.png)
查看本地代码



查看本地代码

![image-20230511142333289](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511142333289.png)

