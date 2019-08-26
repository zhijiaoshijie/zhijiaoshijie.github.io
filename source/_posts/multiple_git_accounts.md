---
title: 如何管理同一电脑上的多个git账号 - 步骤&解释
date: 2019-08-25 12:17:02
tags: Git
categories: 杂记
---

欢迎！

相信当你因为某种原因，创建自己的第二个 Github 账号时，boom。Permission denied。git 不知道哪个仓库属于哪个账号了。别急，只需n步即可为你的新仓库配好账号，让你后续的 push 等操作都和一个账号一样顺滑。

* 注：简单但不一定成功的方法：

  1. 打开你旧账号的 SSH 公钥（例如 ~/.ssh/id_rsa.pub，也就是C:\Users\你的电脑名\\.ssh\id_rsa.pub）
  2. 提交到新的 Github 账号里（还记得吗？右上角头像 > Settings > 左导航栏SSH and GPG keys > 右上角 New SSH key，把id_rsa.pub的内容复制进去。）

  亲测可以用，不过可能有别的奇奇怪怪的问题，或者 Github 不允许这样做（例如[这里](https://stackoverflow.com/questions/3860112/multiple-github-accounts-on-the-same-computer)和[这里](https://stackoverflow.com/questions/48121614/how-to-use-same-ssh-public-key-again-for-another-on-github-account-same-laptop-c)）。如果你不愿意这样做，那就看看下面的**正规方法**吧。

### 第一步 创建新的 SSH 密钥

就像第一次用 git 一样，打开git bash，输入：

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

把`your_email@example.com`替换成你的Github邮箱。

（不是其实也没关系啦，不要跟别的账号重复就好。那样生成的公钥密钥都是完全一样的。这个邮箱会出现在xxx.pub文件的末尾。）

然后在显示出`Enter file in which to save the key`字样时，输入一个新的路径和文件名。如果和第一个账号一样用默认名字（`~/.ssh/id_rsa`），就会被 Overwrite 掉。换成例如（`~/.ssh/id_rsa_1`）就好，不需要后缀名。

下面的`Enter passphrase`则可以回车跳过。

（passphrase是对你的SSH Key设密码，如果你设置了它，你每次使用SSH，都要输一遍这个密码。）

之后，像原来一样，在浏览器登陆 Github，把公钥（.pub文件）提交到云端（右上角头像 > Settings > 左导航栏SSH and GPG keys > 右上角 New SSH key）。

### 第二步 修改 SSH 的 Host 设置:

打开~/.ssh文件夹，也就是C:\Users\你的电脑名\\.ssh文件夹。

看看里边有没有一个叫`config`的文件（没有后缀名哦）。没有就创建一个。

然后往里写这样几句话：

```
Host 你的GitHub账号.github.com
  Hostname github.com
  PreferredAuthentications publickey
  IdentityFile 刚才写路径和文件名
  User 你的Github用户名或邮箱
```

（这里Host只是个任起的名字而已。本段话的意思是，如果某个 Git 仓库的Host 是 `你的GitHub账号.github.com`，就用这些`Hostname`和`IdentityFile`等设置。）

### 第三步 修改本地仓库设置:

打开你希望推到新账号的本地 Git 仓库，找到 .git 文件夹下的 config 文件（这个应该有），找到形如

```
[remote "origin"]
	url = https://github.com/你的Github账号/本Github仓库名.git
```

这两行，把第二行替换为

```
	url = git@你的Github账号.github.com:你的Github账号/本Github仓库名.git 
```

即可。

（这里是定义 Github 在向云端推送时的仓库地址的。Github 一般用 Https 方式，这就是原先的 URL 是一个 https 开头的网址，而我们修改为了 SSH 格式（Host : 用户名 / 仓库地址），这样才能应用上之前的 Host 设置。也就是，其实第2步 Host 行和第3步 url 行输入的“你的Github账号”改成别的什么都行，只要二者相同即可。）

### （可选）第四步 修改作者：

如果你现在进行 commit 和 push 的话，你很有可能会发现**作者**显示的依然是最早的账号。

在你的本地仓库目录下，输入`git config user.email "新账号的邮箱"`，即可更改**仅限本仓库**的后续commit的作者。

（注意，本地`git log`命令可能仍然会显示原作者的用户名和新作者的邮箱，但实际上，邮箱才是认证标识。去浏览器看一看，**云端**显示的作者已经改过来了。）

（提示：输入`git config user.email`，即可显示当前仓库的作者邮箱。）

（解释：你或许之前使用过形如`git config --global user.email "email@example.com"`这种命令。git 是用本地的配置文件判定作者邮箱，并用邮箱判定作者的。而仓库内的配置文件可以override全局配置。当然，如果这个账号将是你新的常用账号，不妨加上`--global`，更改所有账户的作者。）

---

大功告成！

如果你依然有疑问，请务必在下方留言哦！新人博主，请多多指教！



