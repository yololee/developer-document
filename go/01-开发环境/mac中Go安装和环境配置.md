# mac中Go安装和环境配置

## 一、安装

安装地址：https://golang.google.cn/dl/

![image-20230629165326408](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230629165326408.png)

下载好之后，傻瓜式安装

## 二、建立Go的工作空间

GO代码必须在工作空间内。工作空间是一个目录，其中包含三个子目录：

src —- 里面每一个子目录，就是一个包。包内是Go的源码文件

pkg —- 编译后生成的，包的目标文件

bin —- 生成的可执行文件

这里，我们在`/Users/huanglei/Desktop/work/code/myGoCode`目录下, 建立一个名为go(可以不是go, 任意名字都可以)的文件夹，
然后再建立三个子文件夹(子文件夹名必须为src、pkg、bin)

## 三、配置GOPATH

把上面我们创建的go工作空间，添加到go开发环境配置中

```go
# 查看go环境配置
go env 
# 修改go环境配置
export GOPATH="/Users/huanglei/Desktop/work/code/myGoCode"
```

![image-20230720142607565](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230720142607565.png)

## 四、添加环境变量

```shell
vim ~/.bash_profile
```

```shell
# golang安装目录
export GOROOT=/usr/local/go
# golang项目目录
export GOPATH=/Users/huanglei/Desktop/work/code/myGoCode
export PATH=$GOROOT/bin:$PATH
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

## 五、go的项目目录

在进行`Go`语言开发的时候，我们的代码总是会保存在`$GOPATH/src`目录下。在工程经过`go build`、`go install`或`go get`等指令后，会将下载的第三方包源代码文件放在`$GOPATH/src`目录下， 产生的二进制可执行文件放在 `$GOPATH/bin`目录下，生成的中间缓存文件会被保存在 `$GOPATH/pkg` 下。

如果我们使用版本管理工具（`Version Control System`，`VCS`。常用如`Git`）来管理我们的项目代码时，我们只需要添加`$GOPATH/src`目录的源代码即可。`bin` 和 `pkg` 目录的内容无需版本控制。

### 适合个人开发者

我们知道源代码都是存放在`GOPATH`的`src`目录下，那我们可以按照下图来组织我们的代码

![image-20230720141911045](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230720141911045.png)

### 目前流行的项目结构

Go语言中也是通过包来组织代码文件，我们可以引用别人的包也可以发布自己的包，但是为了防止不同包的项目名冲突，我们通常使用顶级域名来作为包名的前缀，这样就不担心项目名冲突的问题了。

因为不是每个个人开发者都拥有自己的顶级域名，所以目前流行的方式是使用个人的github用户名来区分不同的包

![image-20230720142037295](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230720142037295.png)

### 适合企业开发者

![image-20230720142134771](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230720142134771.png)

## 六、入门程序

创建一个main.go文件

```go
package main

import "fmt"

func main() {
	fmt.Println("hello world")
}

```

在该文件目录下执行` go run main.go `。输出结果

```
hello world
```



