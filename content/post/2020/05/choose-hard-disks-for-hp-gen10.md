---
title: "将 HPE ProLiant Gen10 X3421 打造成加强版 NAS"
slug: "将HPE-ProLiant-Gen10-X3421打造成加强版NAS"
date: 2020-05-10T14:32:58+08:00
toc: true
category:
    - 家庭组网
    - 杂谈日记
tag:
    - HDD
    - SSD
    - HPE
    - ProLiant
    - NAS
    - 我的世界
    - Docker
    - FreeNAS
    - ZFS
    - Nextcloud
    - ownCloud
    - Seafile
    - Syncthing
    - Stow
    - RAID
    - Samba
    - Gitolite
    - Plex
---

这两天 [HPE ProLiant MicroServer Gen10 X3421](https://buy.hpe.com/us/en/servers/proliant-microserver/proliant-microserver/proliant-microserver/hpe-proliant-microserver-gen10/hpe-proliant-microserver-gen10-x3421-1p-8gb-u-4lff-nhp-sata-200w-ps-soln-server/p/P04923-S01) 就要到货了，得赶紧提前备好硬盘——一块 SSD 做系统，四块 HDD 做 RAID10 仓库。

<!--more-->

[docker]: /tag/Docker
[zfs]: /tag/ZFS
[debian]: /tag/Debian

## 一 用途规划

本来一直对[群晖 Synology](https://www.synology.com/zh-cn) 蛮有好感，差点就下了单。结果还是接受了一直在用 [HPE ProLiant MicroServer Gen8](https://support.hpe.com/hpesc/public/docDisplay?docId=emr_na-c03793258) 的朋友的建议，自己动手，丰衣足食。

自己动手首先明确用途：

### 1.1 家庭照片和视频多媒体库

目前全家的庭照片、视频和录音，分散在不同的笔记本电脑和手机中，一直都在勉勉强强地维持。笔记本电脑存放着相对久远的数据，剩余空间有限。手机一直记录，又一直在删除一些零三杂乱无用的部分，却无法备份到电脑上。这让我一直有些提心掉胆，生怕某台设备一坏，若干年的生活记忆就消失殆尽。

这一次首先就的解决这个问题。方便备份，能够通过浏览器或者应用程序快速查看，能够组织管理，如果还能定制不同设备的同步，那就更完美了。

### 1.2 个人工作资料库

然后就是个人工作相关的文档、报表、演示等资料数据。虽然说起来在办公电脑里也可以整理得有条不紊，但对那些需要领导反复帮着批改的重要材料，邮件、QQ 传送文件、U 盘等等，桌面、下载文件夹、工作资料目录等等…稍微忙忘了过两天再回来，就不知道电脑里能找到的 27 个不同版本，哪个才是最后该备档的了！

如果能有同步的个人工作资料库，没准能督促我再忙也要尽快把废旧的 26 个文件版本删掉，该备档的备档？没准呢！

### 1.3 Git 版本仓库

除了常见的工作文档外，我个人工作成果里更多是大量的以 Git 版本库形式存在的项目资料（代码为主）。毕竟个人持有公司项目的代码并不违法，相反如果未经公司许可同步到如 [GitHub][] 或 [Gitee][] 之类的云平台中，哪怕是私有库，也存在泄密可能，违法的风险更高一些。

[github]: https://github.com
[gitee]: https://gitee.com

而且，云平台更不靠谱！我从 2006 年就开始在 Microsoft Space Live! 里写博客，中间这家停止服务，那家倒闭，搬家来搬家去，以前写的文章啥都没了！真正免费的才最昂贵！

### 1.4 流媒体库

优先级再次，就是我攒了好些年的电影和音乐专辑，这些都是一张 CD 一张 DVD 或 BD 慢点攒回来的。为了更好的抓出来，当初还苦学了不久压片相关的知识。

各大视频平台、音频平台可能确实都提供线上资源，甚至免费的。但电影动不动剪掉一部分，甚至哪天突然就 404。太糟心了。

### 1.5 [我的世界][minecraft]家庭服务器

[minecraft]: https://www.minecraft.net/en-us/

[我的世界][minecraft] 从 2013 年开始一直就是我的心头好，我也一直坚定地认为应该将它分类成数字玩具，而非电子游戏。疲惫的时候进去挖挖矿、做做红石、或者建建房子，颇有小时候霸占一大片荒地过家家的味道，放松得很。

之前在 [N3150][] 软路由上架设服务器。在[《处理器 CPU 性能排名》](https://itianti.sinaapp.com/index.php/cpu)查到的 [N3150][] 跑分 1656，X3421 有 4928，应该够使。

[n3150]: https://ark.intel.com/content/www/us/en/ark/products/87258/intel-celeron-processor-n3150-2m-cache-up-to-2-08-ghz.html

### 1.6 MacOS 时光机器

MacOS 系统的稳定性相对非常不错，而且对第三方软件的约束也很严格，没啥糟心事。所以就用默认的一个分区也挺开心。而且还提供了时光机器这种更省事的整机备份方案，能上就直接上算了。

### 1.7 [Docker][] 宿主机

多年使用软路由的经验，在一台需要长时间运行、但维护管理极少的系统里，如果用些五花八门的工具或服务，安装的时候挺好，过几个月再来调整时，真的会一头雾水折磨死人。

所以非核心的工具或服务，直接全部 [Docker][] 好了。反正现在一时半会也想不到需要上 Windows 虚拟机的理由，[Docker][] 够轻。

## 二 技术选型

### 2.1 [FreeNAS][]

[freenas]: https://www.freenas.org

[FreeNAS][] 是自从我留心 NAS 起，时不时就会从我耳边飘过的一个名字。去官网翻了翻特性，嗯，口舌莲花，把我唬得一愣一愣的。还好官方展示了比较完整的截图，这就方便我快速拼凑出 [FreeNAS][] 的真实骨架了——

> 一个把控制版面 B/S 化的操作系统。基于 [FreeBSD](https://www.freebsd.org) 改造，使用 [ZFS][] 文件系统来实现诸多 NAS 相关特性。有一些定制的服务程序（插件 plugin），支持虚拟机和 [Docker][]（但据说硬件直穿比较磨人），还支持沙盒（监狱 jail）。

如此说来对我就用不上了，拿出我心爱的 [Debian][] 再插上 [ZFS][] 它就不香么？更广泛的生态环境下寻找各式各样的定制解决方案，理论上也会更容易一些。

### 2.2 [Nextcloud][]、[ownCloud][]、[Seafile][] 和 [Syncthing][]

[nextcloud]: https://nextcloud.com
[owncloud]: https://owncloud.org
[seafile]: https://www.seafile.com/home
[syncthing]: https://syncthing.net

因为这四款产品从来都是结伴出现，但谁家更强就众说纷纭。所以我也只能也全搁一起来比较。

[Nextcloud][] 和 [ownCloud][] 两家首页首屏的 H5 动画几次加载不出来…让我心都凉了半截，什么毛病这是！经费短缺到网站都是设计师来维护的吗？！

#### [Nextcloud][]

首页顶部导航里就讲了自家有多少多少平台的客户端，好评！还提供演示站点，再好评！等等…点进去又回首页了，差评！基本没啥直观的截屏，差评！

文件共享、私有多媒体会议、团队协作…不对！这不是我需要的东西，撤…再等等，还有一个家用版！

家用版简单很多，但还是基于文件共享，然后再多了点社交？嗯，确实不是我需要的东西，走了。

#### [ownCloud][]

演示入口在首页大屏里，不过试用还得先注册呀？注册还得填完整的信用卡信息啊？惹不起惹不起…我去找视频云体验一下算了。

[B 站搜 `ownCloud` 关键字](https://search.bilibili.com/all?keyword=ownCloud)，总共 17 个视频。有点少呀。果不其然，还是基于文件共享的团队协作产品。再撤！

#### [Seafile][]

这个产品倒是我们国产的，据说还借鉴文件系统模型里的一些概念，实现了以数据区块为单元的高性能的存储和同步机制。

唉，核心功能还是文件备份和同步啊…撤！

#### [Syncthing][]

这回更好，名字就充分说明了其设计目的。官网首页的一张截屏，也将这个特点一览无遗。溜了溜了。

四款产品全挂，看来是我自己思路错了。顺着 [FreeNAS][] 的常见伙伴关键词找，有点缘木求鱼。得更针对性地找找相册备份之类的产品。

## 三 方案定型

### 3.1 [Debian][]

操作系统还是上 [Debian][]。节约，稳定，全都是包，没有法律风险。

再祭出我珍藏多年的 [stow][]——所有支持符号链接的配置，定制结果和版本，全部交给 [stow][] 管理——这样过几个月再进系统，看看 `/usr/local/include` 里的目录，就晓得自己之前干过啥了。

[stow]: https://www.gnu.org/software/stow/

### 3.2 [ZFS][]

Debian 官方百科里说自 [9 Stretch](https://wiki.debian.org/DebianStretch) 开始，[使用 Linux 内核时可以通过动态内核模块支持（Dynamic Kernel Module Support，DKMS）加载使用](https://wiki.debian.org/ZFS#Status)。

那就果断给数据存储部分上 ZFS，正好 X3421 支持 4 LFF HDD 盘位，可以组 [RAID10](<https://wiki.archlinux.org/index.php/RAID_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%B5%8C%E5%A5%97_RAID_%E7%BA%A7%E5%88%AB>)。

### 3.3 [Samba][]

[samba]: https://www.samba.org

所有跨硬件设备文件的共享传输行为都需要以 [Samba][] 作为基础。

[家庭相片和视频多媒体库（1.1）](#11-家庭照片和视频多媒体库)可以继续复用 [MacOS Photos](https://www.apple.com/macos/photos/)。只是需要将分散在多台机器中的库合并，然后存储在特定的共享卷中，作为外部库进行操作和管理。

而且[有文章实践确认](https://nick.bouwhuis.io/2020/01/03/time-machine-server-on-debian-10-copy/)， [Samba][] 自 4.9.1 版本以后，开始支持 [MacOS 时光机器（1.6）](#16-macos-时光机器)功能。因此[个人工作资料库（1.2）](#12-个人工作资料库)也只需要考量 Windows 机器即可，无论自动同步机制如何设计，快速浏览和操作总是需要的。

### 3.4 [Gitolite](https://gitolite.com/gitolite/index.html)

多年的老朋友，简单、干净、明晰。对只需要存储版本库即可的我来说，恰到好处。[Git 版本仓库（1.3）](#13-git-版本仓库)，搞定！

### 3.5 [Plex][]

[plex]: https://www.plex.tv

我在探究 [FreeNAS][] 是如何管理[流媒体库（1.4）](#14-流媒体库)时发现了 [Plex][]，然后就发现似乎这款产品已经统治了家庭 NAS 网络领域，甚至更多！它居然还有 PS4 和 XBox 的客户端，简直了！

至于是否能够针对不同用户（设备）进行内容分级或者屏蔽？如何编辑视频的元信息和标签？对于在这个细分领域一枝独秀的 [Plex][] 来说，我们似乎并无选择。

### 3.6 [Docker][]

仔细再三思考，[我的世界 Minecraft 家庭服务器（1.5）](#15-我的世界minecraft家庭服务器)其实是一个部署之后几乎无更改、只会阶段性启动的服务，完全可以自己做一个 [Docker][] 镜像来跑容器！

正好和 [Docker 宿主机（1.7）](#17-docker-宿主机)一起满足，事半功倍。

## 四 稳定性控制

虽然没看过 [FreeNAS][] 源代码，不过我琢磨着沙盒（jail）功能应该就是基于 [Docker][] 实现的。

即便不是，我也决定了就用 [Docker][] 来实现这个效果！除了以上列出的几项，其余任何变动都必须通过不同的 [Docker][] 容器实现。

唯一可能例外的，应该就剩下 [QEMU](https://www.qemu.org) 这样的大型虚拟化方案了。以后真要用的时候再说。
