# macos配置环境变量

macOS 的环境变量一般配置在 .bash_profile 或者 .zshrc 文件里，所以只要将下载的安装包路径写入 .bash_profile 或者 .zshrc 并完成保存就好了。

### macOS 下 .bash_profile 和 .zshrc 两者之间的区别

.bash_profile 中修改环境变量只对当前窗口有效，当关闭窗口后再使用可能会报 `zsh: command not found: XXX`; 而且需要 `source ~/.bash_profile` 才能使用;

.zshrc 则相当于 windows 的开机启动的环境变量, 永久有效。

所以建议尽量只使用 .zshrc，然后加一行 `source .bash_profile` 以兼容不小心加入 .bash_profile 文件的变量。

### 编辑 .bash_profile 文件

1. 打开终端
2. vi ~/.bash_profile 或者 open ~/.bash_profile 打开文件
3. 修改后 通过 :wq 保存
4. source ~/.bash_profile 生效

![image-20230728110026135](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230728110026135.png)

### 编辑 .zshrc 文件

只有当在 Mac OS 上使用 zsh shell 时，才会获得 ~/.zshrc 文件，如果你不确定自己使用的是哪个 shell，请打开终端并发出以下命令：

```bash
bash
复制代码echo $SHELL
```

若结果显示为 /bin/zsh， 说明你在 macOS 上使用的是 zsh shell

1. open ~/.zshrc 或者 vim ~/.zshrc
2. 在打开的 .zshrc 文件窗口中进行更改
3. 通过 :wq 保存
4. source ~/.zshrc