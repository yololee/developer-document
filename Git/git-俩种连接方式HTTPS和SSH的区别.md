# git-俩种连接方式HTTPS和SSH的区别

首先创建仓库，建好仓库以后，进入仓库，会看见有两种连接方式。

![image-20230719151101610](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719151101610.png)


## 一、HTTPS 远程连接

点击右边的复制，复制HTTPS地址。

选用HTTPS的连接方式每次都需要输入码云的用户名和密码。

```shell
首先创建一个文件夹，本地仓库初始化
git init

并创建一个文件(空文件无法提交)
cat hello.md

添加到缓存区
git add hello.md

添加到本地仓库
git commit -m 'first'

添加远端仓库
git remote add origin https://gitee.com/huanglei1111/git-demo.git

使用 git push 命令向远端仓库提交项目
git push -u origin "master"
```

最后一步的时候会出现弹窗

![image-20230719151110921](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719151110921.png)


这里需要输入gitee的账户和密码；

HTTPS的远程连接方式每次都需要输入用户名和密码。

![image-20230719151124139](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719151124139.png)


输入一次账号密码之后这里就会出现凭证，就不用每次都输入密码，如果没有，就自己手动添加

## 二、SSH 远程连接

接下来，改用 SSH 的远程连接方式，先把原来的远端仓库删掉。

```shell
# 这里是把本地的删除
git remote remove origin
```

创建远端仓库

```shell
git remote add origin <SSH地址>
git remote add origin git@gitee.com:huanglei1111/git-demo.git
```

设置密钥 ssh-keygen

```shell
ssh-keygen -t rsa -b 2048 -C "572613158@qq.com"
```

| 参数 | 解释             |
| ---- | ---------------- |
| -t   | 加密算法(rsa)    |
| -b   | 密钥长度         |
| -C   | 说明(邮箱做分割) |

由于我以前配置过一次，所以这里重新配置的密钥会覆盖之前的。
然后一直按 enter 键就完了。

```shell
admin@DESKTOP-VO31VAE MINGW64 /e/Users/gitdemo (master)
$ git remote remove origin

admin@DESKTOP-VO31VAE MINGW64 /e/Users/gitdemo (master)
$ git remote add origin git@gitee.com:huanglei1111/git-demo.git

admin@DESKTOP-VO31VAE MINGW64 /e/Users/gitdemo (master)
$ ssh-keygen -t rsa -b 2048 -C "huanglei@mye.hk"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/admin/.ssh/id_rsa):
/c/Users/admin/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/admin/.ssh/id_rsa.
Your public key has been saved in /c/Users/admin/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:szcGY/9+obIGiK9925vX1VtvZb3zu8VCvRyyhCdyXSo huanglei@mye.hk
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|               . |
|            o o. |
|     . .S. E *..+|
|    . ...*o =.=.X|
|     .  ..=  +.*O|
|     .. .+o+o o=+|
|    .....o=*o. +*|
+----[SHA256]-----+

```

进入 .ssh 文件夹下面，把 .ssh 文件夹下面的公钥文件打开，将其中的内容全部复制。

![image-20230719151140061](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719151140061.png)


打开码云，在设置中将公钥粘贴到指定位置保存。配置免密完成。

![image-20230719151154343](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230719151154343.png)


再次执行push，无需用户名和密码！

```shell
admin@DESKTOP-VO31VAE MINGW64 /e/Users/gitdemo (master)
$ git push -u origin master
Everything up-to-date
Branch 'master' set up to track remote branch 'master' from 'origin'.

admin@DESKTOP-VO31VAE MINGW64 /e/Users/gitdemo (master)
$ git remote -v
origin  git@gitee.com:huanglei1111/git-demo.git (fetch)
origin  git@gitee.com:huanglei1111/git-demo.git (push)
```

