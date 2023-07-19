# git-基本使用教程



## 一、Git简介

### 1.1 Git与SVN对比

SVN是集中式版本控制系统，版本库是集中放在中央服务器的，而开发人员工作的时候，用的都是自己的电脑，所以首先要从中央服务器下载最新的版本，然后开发，开发完后，需要把自己开发的代码提交到中央服务器。

> 集中式版本控制工具缺点：服务器单点故障，容错性差

![image-20230719150451097](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150451097.png)


Git是分布式版本控制系统（Distributed Version Control System，简称 DVCS） ，分为两种类型的仓库：本地仓库（是在开发人员自己电脑上的Git仓库）和远程仓库（是在远程服务器上的Git仓库）

> Clone：克隆，就是将远程仓库复制到本地
> Push：推送，就是将本地仓库代码上传到远程仓库
> Pull：拉取，就是将远程仓库代码下载到本地仓库

![image-20230719150505644](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150505644.png)


### 1.2 Git工作流程

1. 从远程仓库中克隆代码到本地仓库
2. 从本地仓库中checkout代码然后进行代码修改
3. 在提交前先将代码提交到暂存区
4. 提交到本地仓库。本地仓库中保存修改的各个历史版本
5. 修改完成后，需要和团队成员共享代码时，将代码push到远程仓库

## 二、Git下载与安装

Git 各平台安装包下载地址为：http://git-scm.com/downloads

windows安装包下载地址：https://gitforwindows.org/

官网慢，可以用国内的镜像：https://npm.taobao.org/mirrors/git-for-windows/。

![image-20230719150519317](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150519317.png)


下载完成后可以得到如下安装文件：

![image-20230719150530783](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150530783.png)


这里默认下载的是64位的软件，双击下载的安装文件来安装Git。一直下一步直到安装完成即可

安装完成后在电脑桌面（也可以是其他目录）点击右键，如果能够看到如下两个菜单则说明Git安装成功。

![image-20230719150539438](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150539438.png)


**Git GUI：Git提供的图形界面工具
Git Bash：Git提供的命令行工具**

## 三、Git常见命令

### 3.1 环境配置

<b>步骤：</b>

1. 设置Git账号
2. 生成SSH公钥
3. 设置账户公钥
4. 公钥测试

| 命令                                              | 介绍                  |
| ------------------------------------------------- | --------------------- |
| git config user.name                              | 查看git账号           |
| git config user.email                             | 查看git邮箱           |
| git config --global user.name “账户名”            | 设置全局账户名        |
| git config --global user.email “邮箱名”           | 设置全局邮箱          |
| cd ~/.ssh                                         | 查看是否生成过SSH公钥 |
| ssh-keygen -t rsa -C '邮箱' (注意中间敲击3次回车) | 生成公钥              |
| cat ~/.ssh/id_rsa.pub                             | 查看公钥              |
| ssh -T git@gitee.com                              | 测试公钥              |
| git config --list                                 | 查看配置信息          |

![image-20230719150558031](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150558031.png)


### 3.2 获取Git仓库

<b>在本地初始化一个Git仓库</b>

步骤：

1. 在电脑的任意位置创建一个空目录（例如repo1）作为我们的本地Git仓库
2. 进入这个目录中，点击右键打开Git bash窗口
3. 执行命令git init

如果在当前目录中看到.git文件夹（此文件夹为隐藏文件夹）则说明Git仓库创建成功

![image-20230719150612382](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150612382.png)


<b>从远程仓库克隆</b>

命令形式为：git clone 远程Git仓库地址

### 3.3 工作目录、暂存区以及版本库概念

- 工作目录(Working Tree)：代码存放位置
- 缓存区(Index)：代码提交到仓库之前的临时存储空间
- 本地历史仓库(Repository)：存放不同版本的代码

![image-20230719150620973](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150620973.png)


### 3.4 本地仓库操作

| 命令                    | 介绍                                      |
| ----------------------- | ----------------------------------------- |
| git init                | 初始化，创建git仓库                       |
| git status              | 查看git状态(文件是否进行了添加，提交操作) |
| git add 文件名          | 添加，指定文件添加到缓存区                |
| git commit -m’文件说明’ | 提交，将缓冲区文件提交到本地历史仓库      |
| git log                 | 查看日志(git提交的历史日志)               |
| cat 文件名              | 查看文件内容                              |
| git reset 文件名称      | 将缓存区的文件取消暂存                    |
| git rm 文件名称         | 删除文件                                  |

### 3.5 远程仓库操作

| 命令                                | 介绍                                            |
| ----------------------------------- | ----------------------------------------------- |
| git remote                          | 查看远程仓库                                    |
| git remote add 仓库名称 远程仓库URL | 自定义本地仓库名称                              |
| git push -u 仓库名称 分支名         | 推送                                            |
| git clone 仓库地址                  | 从远程仓库克隆                                  |
| git pull 远程仓库名 分支名          | 从远程仓库获取最新版本并merge到本地仓库         |
| git fetch远程仓库名 分支名          | 从远程仓库获取最新版本到本地仓库，不会自动merge |

### 3.6 Git分支

| 命令                 | 介绍                       |
| -------------------- | -------------------------- |
| git branch           | 列出所有本地分支           |
| git branch -r        | 列出所有远程分支           |
| git branch -a        | 列出所有本地分支和远程分支 |
| git branch 分支名    | 创建新分支                 |
| git checkout 分支名  | 切换新分支                 |
| git merge 分支名     | 合并分支                   |
| git branch -d 分支名 | 删除分支                   |

### 3.7 Git版本管理

- 准备工作
  - 查看本地仓库的log日志(git reflog)
- 需求：将代码切换到，第二次修改的版本
  - 指令：git reset --hart 版本唯一索引值

| 命令                            | 介绍                                                     |
| ------------------------------- | -------------------------------------------------------- |
| git reflog                      | 查看所有分支的所有记录(包括已经被删除的commit记录的操作) |
| git reset --hart 版本唯一索引值 | 根据版本唯一索引值切换版本                               |

## 四、在IDEA中使用Git

### 4.1 在IDEA中配置Git

![image-20230719150644916](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150644916.png)


选择git的安装目录后可以点击“Test”按钮测试是否正确配置


### 4.2 在IDEA中生成本地仓库

![image-20230719150704324](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150704324.png)


### 4.3 在IDEA中进行commit和push

![image-20230719150716037](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150716037.png)


### 4.4 在IDEA中进行clone

![image-20230719150729027](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150729027.png)


### 4.5 在IDEA中进行分支操作

![image-20230719150749385](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150749385.png)


![image-20230719150800622](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150800622.png)


![image-20230719150810772](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719150810772.png)