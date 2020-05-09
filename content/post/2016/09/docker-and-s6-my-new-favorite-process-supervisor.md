---
title: "【译】Docker 和 S6 ——我的进程管理器新欢"
slug: "Docker和S6我的进程管理器新欢"
date: 2016-09-29T05:50:04Z
category:
    - Docker
tag:
    - Docker
    - S6
    - 自主翻译
---

[点击这里阅读原文。](https://blog.tutum.co/2014/12/02/docker-and-s6-my-new-favorite-process-supervisor/)

---

在[我之前的博文](https://blog.tutum.co/2014/10/28/the-5-most-important-things-ive-learned-from-using-docker/)中，我描述了我是如何在容器中使用进程管理器的，并做了一些展开。但我还是觉得有必要针对 [Laurent Bercot 开发的 S6](http://skarnet.org/software/s6/) 做更详尽的说明。

<!--more-->

## 什么用 S6 而不是 Supervisor？

我知道 [Supervisor](http://supervisord.org/) 是一套很不错的系统，既便于学习，又有很强大的功能，很多人都在自己的容器中使用它。[Phusion](http://phusion.github.io/baseimage-docker/) 也发布了一套基于 Ubuntu 和 Supervisor（纠正：Runit） 的非常流行而实用的基础镜像。

**2014 年 12 月 9 日更新**：我以为 Phusion 使用地是 Supervisor，但实际上是 Runit 。

当 Docker 启动容器时，ENTRYPOINT 进程会作为初始进程（PID 1）启动，和 Linux 系统类似（无论是物理系统、虚拟机，还是容器）。初始进程有一个特殊的职责，就是当任何「孤儿进程」结束时，都需要初始进程对其“收割”清理。

Supervisor 专门说明了其不适合作为初始进程使用。如果某些子进程会将自身分入（fork）后台，Supervisor 无法对其进行清理。Phusion 是靠编写了自己的[初始进程](https://github.com/phusion/baseimage-docker/blob/master/image/bin/my_init)程序来解决这一问题。但这太夸张了，直接选用一套可以作为初始进程的进程管理器不就好了么。

S6 就可以作为初始进程运行，因此我觉得它是一种简单省事的好方法。而除此之外，S6 是使用 C 编写的，我们可以很容易地在任何镜像中使用其静态编译程序，即便是 [busybox](https://registry.hub.docker.com/_/busybox/) 镜像。

## 在镜像中添加 S6

往镜像中添加 S6 的方法有两种。要么就是直接在 Dockerfile 中编译，要么就是在外部静态编译完成之后使用 COPY 或者 ADD 指令拷进去。我个人倾向于后一种，因为既能减少镜像的构建时间，又能让镜像保持更小的尺寸。

我现在很喜欢使用 Dcoker 去构建“编译类镜像”，也就是专门用一个镜像来编译代码并打包。[在 Github 上](https://github.com/jprjr/docker-misc/tree/s6-builder/dockerfiles/arch-s6-builder)就有一个这样只用来静态编译 S6 的镜像。大家感兴趣地话可以参考看看如何编译 S6。

一般来说，我会使用自己的基础镜像来开始后面的工作。基础镜像中除了 S6，也有一些我常用的程序（比如 curl，我是真的没它就不能活）。然后在项目文件夹中创建一个名为 `root` 的文件夹，将需要的文件放在其中，再在 Dockerfile 的最后通过 `COPY root /` 添加到镜像里。这样做的好处在于使 S6 和其它文件是在同一 Docker 镜像层里。

`root` 文件夹的简化版本是长成这样：

```
root
|-- etc
|   |-- s6
|       |-- cron
|       |   |-- finish
|       |   |-- run
|       |-- syslog
|           |-- finish
|           |-- run
|       |-- .s6-svscan
|           |-- finish
|-- usr
|       |-- bin
|           |-- (s6 程序)
```

正如我刚才所说，当我执行 `docker build` 时，这些文件会被复制到同一 Docker 镜像层里。

## 使用 S6 启动服务

S6 中类初始化系统的主程序是 s6-svscan。当其启动时，会扫描某个目录以寻找“服务文件夹”，并对每个结果调用 s6-supervise 。在我上面的例子里，`/etc/s6` 是用于扫描的“根”目录，因此 `cron` 和 `syslog` 都是“服务文件夹”。注意 `.s6-svscan` 不是“服务文件夹”，它是被 s6-svscan 所使用的。

每个“服务文件夹”里都有两个文件 `run` 和 `finish` 。s6-supervisor 程序首先会调用 `run` 程序，然后当 `run` 结束时，就会调用 `finish` 程序，再然后重新调用 `run` 程序，如此往复。`run` 和 `finish` 程序可以是任何内容，shell 脚本也好，不带参数的其它程序也好，指向别的程序的符号链接也好。当我的某个“服务文件夹” `run` 程序就可以正常结束而用不上 `finish` 程序时，我就会直接把 `finish` 做成指向 `/bin/true` 的符号链接。

在运行实际程序时，S6 和 Supervisor、Upstart 或 Systemd 很相似，“保持”这个程序的运行，而不是像 SysV init 那样记录 PID 信息。所以需要确保 `run` 程序是以前台或非守护模式启动程序。

要实现这个目标并不难——比如我的 `cron` 的 `run` 脚本：

```bash
#!/bin/sh
exec cron -f
```

又比如 `syslog` 的 `run` 脚本：

```sh
#!/bin/sh
exec rsyslogd -f /etc/rsyslog.conf -n
```

对应的 Dockerfile 中 ENTRYPOINT 和 CMD 指令是：

```dockerfile
ENTRYPOINT ["/usr/bin/s6-svscan","/etc/s6"]
CMD []
```

轻松搞定！接下来我们细说如何通过 S6 在一个容器里跑多个进程。

## 处理 `docker stop`

前文中我提到过 `.s6-svscan` 文件夹，其实它很重要。当 Docker 停止一个容器时，它会发送 SIGTERM 信号给初始进程，在本文中也就是 s6-svscan 。当 s6-svscan 接受到信号，它会向所有运行中的“服务”发送 SIGTERM 信号，然后执行 `.s6-svscan/finish` 。

**2015 年 3 月 1 日更新**：Laurent 告诉我，当 s6-svscan 接受到 SIGTERM 信号时，它会：

-   发送 SIGTERM 信号给每个 s6-supervise 实例（每个监视进程都会有一个相应的 s6-supervise 进程）；
-   s6-supervise 会发送 SIGTERM 信号给监视进程，然后执行相应的 `finish` 脚本；
-   再然后，s6-svscan 会执行 `.s6-svscan/finish` 脚本。

当 s6-supervisor 接受到 SIGTERM 信号时，它会以 stdin 和 stdout 指向 /dev/null 地方式执行 `finish` ，这就意味着我们可以看到 `finish` 脚本的任何输出。但这并不影响脚本的正常执行。

Laurent 正在尝试编写一套范例，以解释这块比较容易把人弄混地逻辑。

## 正确地应用到 Docker 生态系统中

在我之前的文章里，我曾说过我喜欢将一些进程视作“关键”进程，当其结束时，容器也也应该停止。我这样做的原因在于 Docker 容器就是这样工作地——运行单一进程，然后在该进程结束时随之停止。

举个例子，我需要跑一套 NodeJS 程序（比如 Ghost），辅以 `cron` 和 `syslog` 。我并不在意 `cron` 和 `syslog` 是否会结束，因为 s6 会自动将它们重启。但如果 Ghost 结束了，我就需要容器也停止，这样宿主机才能及时向我报警。因此 Ghost 的 `finish` 脚本应该是：

```sh
#!/bin/sh
s6-svscanctl -t /etc/s6
```

这就使 s6-svscan 将所有服务都停止然后结束。

## 以后项目的一些想法

以下是一些我想在以后实现地内容。

### 基于依赖地启动

我相信 S6 可以做到，但我还没找到方法。

### 高效日志

S6 有一种很有趣的日志处理方法——当我创建一个名为 `log` 的“服务文件夹”，也给出相应的 `run` 脚本，其它服务的输出会以管道（pipe）的形式传递给 `run` 脚本。此外，s6-log 程序也可以用于将日志管道传递给其它进程，处理日志切割，等等。

我看到大量的镜像会把所有的日志都直接输出，让 Docker 来进行处理。但我想应该可以通过这些工具找到更好的做法——目前我还不太确定怎样算“更好”，但我会就此继续思考。

## 结语

我认为 S6 是一套有趣且有效的 Supervisor 的替代方案，而尤为让我高兴地是，我可以在任何镜像里使用它，即便是 busybox 镜像也是如此。我真切地希望这篇文章对你有益——如果你有什么好的想法，或者有类似的经验，请通过评论与我分享。非常感谢！
