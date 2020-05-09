---
title: "使用 GnuPG2 为 Git 版本签名"
slug: "使用GnuPG2为Git版本签名"
date: 2016-10-24T09:10:02Z
category:
    - 开发技巧
tag:
    - Linux
    - Git
    - GnuPG2
---

从很久很久以前开始，我就一直用 [GnuPG][] 对自己提交的 Git 代码签名。最初地时候是为了装Ｘ，后来则是为了确保代码的可靠性——毕竟 `git config` 都是随便写的。今天因为更换 [Homebrew][]，顺路就更新到了 GnuPG2。但是，问题来了…

[gnupg]: http://baike.baidu.com/view/1657408.htm
[homebrew]: http://brew.sh

<!--more-->

`git commit -veS` 报错：

```
You need a passphrase to unlock the secret key for
user: "Snakevil Zen <zsnakevil@gmail.com>"
1024-bit RSA key, ID 37B86419, created 2015-02-02

gpg-agent[19813]: command get_passphrase failed: Inappropriate ioctl for device
gpg: problem with the agent: Inappropriate ioctl for device
gpg: skipped "37B86419": Operation cancelled
gpg: signing failed: Operation cancelled
error: gpg failed to sign the data
fatal: failed to write commit object
```

这是什么鬼？之前用 [GnuPG][] 地时候可从来没有碰到过这种问题。

搜索，按照[攻略](http://www.zsh.org/mla/users/2015/msg01176.html)添加一个环境变量搞定。

```bash
export GPG_TTY=$( tty )
```

因为我肯定用地是自己的 [Bashrc.X](https://github.com/snakevil/bashrc.x) 嘛（无耻打一波广告），所以实际上是添加了一个配置脚本 `~/.bashrc.x/bashrc.d/01-env.sh`。无脑代码如下：

```bash
echo 'export GPG_TTY=$( tty )' >> ~/.bashrc.x/bashrc.d/01-env.sh
```
