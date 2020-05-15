---
title: "【译】第 19 章：Z 文件系统㈤"
slug: "第19章：Z文件系统五"
date: 2020-05-13T22:41:27+08:00
category:
    - Linux
    - 博闻广记
tag:
    - ZFS
    - FreeBSD
    - Linux
    - 自主翻译
---

[点击这里阅读原文。](https://www.freebsd.org/doc/handbook/zfs-zfs-allow.html)

> 我打赌 100 块钱这四位文档撰写人员没一个的母语是英语！通篇翻译下来都有一股拉丁语系机翻英语的味道。

---

## 19.5 委派管理

<!--more-->

完备的权限委派系统能让非特权用户也能执行 ZFS 的管理功能。例如，如果每个用户的家目录都是一个数据集，则可以授权用户对各自的家目录创建和销毁快照。备份用户则被授权使用同步功能。而用况统计脚本却只应能访问全部用户的空间使用率数据。甚至可以将委派授权的能力也委派出去。每个子命令和大多数属性都可以进行权限委派。

## 19.5.1 委派创建数据集

`zfs allow someuser crete mydataset` 授予指定用户在特定数据集中创建子集的权限。需要注意的是：创建新数据集后还需要将其挂载。因此还需要在 FreeBSD 中使用 [sysctl(8)](https://www.freebsd.org/cgi/man.cgi?query=sysctl&sektion=8&manpath=freebsd-release-ports) 设置 `vfs.usermount` 为 1 以允许非 `root` 用户也可以挂载文件系统。此外还有一个防止骚扰的限制：非 `root` 用户必须是挂载点的所有者。

## 19.5.2 委派权限委派

`zfs allow someuser allow mydataset` 授予指定用户将其在目标数据集及子集上的权限赋予其它用户的能力。如果该用户具有 `snapshot` 和 `allow` 权限，他就可以将 `snapshot` 权限授权给其它用户。
