---
title: "【译】Baseimage-docker，胖容器和“容器也是虚拟机”"
slug: "Baseimage-docker、胖容器和容器也是虚拟机"
date: 2016-09-29T02:45:27Z
lastmod: 2016-09-29T04:25:47Z
toc: true
category:
    - Docker
tag:
    - Docker
    - 自主翻译
    - Ubuntu
---

作者 Hongli Lai 看着像是华人，可惜没找到文章的中文版本，于是就很土鳖地手翻了一遍。文章的措辞很口语化，因此意译为主。[点击这里可以阅读原文。](https://blog.phusion.nl/2015/01/20/baseimage-docker-fat-containers-treating-containers-vms/)

---

[Baseimage-docker](http://phusion.github.io/baseimage-docker/) 是针对 Docker 优化地 Ubuntu 最小化基础镜像。我们可以[从 Docker 仓库中拉取回来](https://index.docker.io/u/phusion/baseimage/)作为基础镜像用在自己的镜像中。

<!--more-->

笔者也算是 Docker 的早期使用者了，早在其 1.0 版本发布前，就已经将其用来做持续集成和构建开发环境了。因而，笔者才研发了 Baseimage-docker 以解决 Docker 工作模式中的一些问题，主要是[子进程的“僵尸化”问题](https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/)。

我们认为：

1. Baseimage-docker 所致力解决地问题适用于很多人；
2. 大多数人并没有意识到这些问题，因此在创建自己的 Docker 镜像时会出现各种诡异地问题；
3. 避免每个人去重复性地解决同样地问题是有价值的。

因此笔者在闲暇时间将解决方案制作成了每个人都能使用的 [Baseimage-docker](https://github.com/phusion/baseimage-docker) 基础镜像，以避免社区里反复地去造同样的轮子。这个解决方案看起来反响不错：在 Docker 仓库中，Baseimage-docker 是仅次于 Ubuntu 和 CentOS 官方镜像的，排名第三的，最受欢迎的第三方镜像。

![](/2016/09/28/popularity.png)

## 胖容器，“也是虚拟机”

一直以来，很多人对 Baseimage-docker 的印象都是“胖容器”，或者干脆“就是虚拟机”。Docker 开发者们执着地追求着那些小而轻的，一次只解决一个问题的容器方案。Baseimage-docker 的多进程复合方案似乎违背了这种哲学。

但是，Docker 开发者们真正所期望的，其实是每个容器只跑一个*逻辑服务*。或者说，每个容器只承担一种*责任*。Baseimage-docker 对此并无异议，因为**一个逻辑服务也可以是由多个系统进程组成**。Baseimage-docker 同样不赞同胖容器或者“容器也是虚拟机”的做法。

那么 Baseimage-docker 是否倡导在一个容器中跑多个逻辑服务呢？见仁见智。尽管 Docker 哲学倡导瘦（slim）容器，但有些情况下我们还是需要在单一容器中跑多个服务。

## 为何需要多个进程？

Baseimage-docker 使用多个操作系统进程的最根本的原因，在于需要解决[子进程的“僵尸化”问题](https://blog.phusion.nl/2015/01/20/baseimage-docker-fat-containers-treating-containers-vms/)。

其次是将逻辑服务分割到多个操作系统进程中会更**安全**。在使用不同系统帐号启动不同进程再进而组合成服务时，安全隐患可以局限在更小的范围内。Baseimage-docker 提供了包括 `setuser` 在内的很多工具，以更好地使用不同的系统帐号来启动进程。

第三则在于能够自动重启崩溃地进程。我们很多人会使用 [Supervisord](http://supervisord.org/) 来达到这一目的，但在 Baseimage-docker 中使用地是 [Runit](http://smarden.org/runit/) 。因为在笔者看来，它更易于使用，效率更高，对资源的需求也更少。在 Docker 1.2 版本以前，如果主进程崩溃了，那么容器就会自动停止。1.2 版本新增的容器自动重启机制，变相地实现了这一目标。但对于使用不同系统帐号启动不同的进程，Runit 仍然是一个好工具。另外在某些情况下，也存在着只重启部分服务的需求。

## Baseimage-docker is about freedom

虽说遵循 Docker 哲学是好事情，但我们坚信最重要的还是在于实际需要解决地问题是什么。Docker 归根到底还是一套通用工具，和 FreeBSD jails、Solaris zones 并无区别。对我们来说，Docker 主要应用场景包括：

1. 持续集成；
2. 构建便携开发环境（如：取代 Vagrant）；
3. 构建用于编译软件的可控环境（如：[移植 Ruby](http://phusion.github.io/traveling-ruby/) 和 [passenger_rpm_automation](https://github.com/phusion/passenger_rpm_automation)）。

出于这些原因，Baseimage-docker 鼓励大家尽可能地遵循 Docker 哲学，但不做强制要求。

## Baseimage-docker 是如何遵循 Docker 哲学的？

我们说 Baseimage-docker “针对 Docker 优化”了 Ubuntu，并“接受了 Docker 哲学”，究竟是指什么？下面举例说明。

### 环境变量

使用环境变量传参给 Docker 容器，这就“很 Docker”。但如果容器中有多个进程，那么环境变量的原始值就很容易丢失。比如，sudo 会出于安全考虑重置环境变量。Nginx 之类的软件也会因为安全而重置环境变量。

Baseimage-docker [实现了一套访问环境变量原始值的机制](https://github.com/phusion/baseimage-docker#environment_variables)，以使特定被允许的进程可以访问到特定的环境变量。

### 更好地集成 `docker logs`

Baseimage-docker 努力将日志集成到 `docker logs` 中。服务程序会更倾向于将日志写入 syslog 或文件中，但 Docker 却更倾向于直接在 stdout 和 stderr 中输出日志（`docker logs` 捕获）。

下一版本的 Baseimage-docker 会尽量将所有的 syslog 日志转向 `docker logs` ，以更好地遵循 Docker 哲学。

### 使用 `docker exec` 替代 SSH

Baseimage-docker 使用 SSH 来访问容器。这也是为什么人们会认为 Baseimage-docker 是胖容器。

但这并非镜像中包含 SSH 服务地理由。其目的在于提供一种调试、检查和维护容器的渠道。在 Docker 1.4 版本以前，也就是还没有 `docker exec` 的时候，我们是没有任何内置方案来访问容器的，所以才有了 SSH 服务。

部分人认为容器应该当作黑盒使用，如果需要访问容器，那么容器的设计就不够好。Baseimage-docker 并不反对这种观念。包含 SSH 服务并非就是为了访问容器，而是能够更好地处理突发问题。无论我们是怎么在设计容器，但只要是用于生产环境，那么我们必然会有一天需要在其内部去调试某些问题。Baseimage-docker 只是提前为这一天做准备而已。

尽管如此，SSH 服务仍然广受批评。在 Docker 1.4 版本以前，主流方法是使用 lxc-attach 和 nsenter 。但因为 Docker 从 0.7 版本开始，就不再使用 LXC 作为后端，lxc-attach 就无法使用了。nsenter 是一个不错的替代方案，但也有[其自己的问题](https://github.com/phusion/baseimage-docker/tree/rel-0.9.15#login-to-the-container-or-running-a-command-inside-it-via-nsenter)，比如没有被收录到我们常用 Linux 发行版本的包仓库里，又比如需要 root 权限才能使用。当然，SSH 也有其自身的问题。但世上没有什么万能解决方案。所以 [Baseimage-docker 同时支持 SSH 和 nsenter](https://github.com/phusion/baseimage-docker/tree/rel-0.9.15#run_inside_existing_container)，也[对各种方式的优缺点做了详细说明](https://github.com/phusion/baseimage-docker/tree/rel-0.9.15#login-to-the-container-or-running-a-command-inside-it-via-nsenter)。

Docker 在 1.4 版本实现了 `docker exec` 命令。这个命令很像是一份被 Docker 略微改造过，包含在 Docker 内部的 nsenter 。这意味着在大多数场合里，我们不再额外需要 SSH 或 nsenter 。但某些场合里，nsenter 仍然是个好选择。比如，`docker exec` 需要访问 Docker 服务程序，而具备这一权限的系统帐号一般也具有 root 权限。

但是，无论如何，`docker exec` 肯定“更 Docker”。下一版本的 Baseimage-docker 会[将 `docker exec` 作为访问容器的默认方式](https://github.com/phusion/baseimage-docker/issues/168)。然后为了避免 `docker exec` 的问题，SSH 服务仍然会作为一种替代方案包含在镜像中，只是默认不启用。两种方式各自的优缺点，也会一如既往地加以详细说明，以避免用户盲目随大流跳进坑里。

## 结语

Baseimage-docker 既不是胖容器，也不是以虚拟机的方式去解决问题。其使用多进程方案也没有违背 Docker 的哲学。而且，Docker 哲学并没有非白即黒，而是一种引导和思路。所以 Baseimage-docker 也会随着版本更新而更贴合 Docker 哲学。

### Baseimage-docker 是唯一的正确方案吗？

当然不是。Baseimage-docker 致力于：

1. 让人们意识到 Docker 容器的一些潜在问题和风险；
2. 提供他人无需深入关注也能安全使用的预构建方案。

这也意味着只要是解决了我们所描述地问题的方案，都是可行的。可以随意使用 C、Go、Ruby 或者什么别的语言来实现相应的方案。但我们既然已经有了现成的好方案，为什么还要再重新做轮子呢？

或许我们使用地基础镜像并不是 Ubuntu，或者是 CentOS。但 Baseimage-docker 同样可以为我们所用。比如笔者公司的 [passenger_rpm_automation](https://github.com/phusion/passenger_rpm_automation) 项目就基于 CentOS，直接将 Baseimage-docker 的 [my_init](https://github.com/phusion/baseimage-docker/blob/master/image/bin/my_init) 移植了过去。

因此即便我们没有、也不想用 Baseimage-docker ，我们在前文中描述的问题仍然值得关注，也仍然值得仔细思考如何去解决。

Happy Dockering.
