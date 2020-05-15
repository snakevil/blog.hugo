---
title: "【译】第 19 章：Z 文件系统㈠"
slug: "第19章：Z文件系统一"
date: 2020-05-11T22:36:35+08:00
category:
    - Linux
    - 博闻广记
tag:
    - ZFS
    - FreeBSD
    - Linux
    - 自主翻译
---

[点击这里阅读原文。](https://www.freebsd.org/doc/handbook/zfs.html)

---

由 Tom Rhodes、Allan Jude、Benedict Reuschling 和 Warren Block 编写。

> **吐槽：**
>
> 这帮大佬肯定都是技术宅！文档写得干干巴巴、麻麻咧咧。翻译起来脑壳疼！

<!--more-->

Z 文件系统，或 ZFS，是一种旨在克服过往设计中诸多已发现问题的高级文件系统。

最初由 Sun™ 开发，现在 ZFS 的开源开发已变更为 [OpenZFS 项目](http://open-zfs.org/)。

ZFS 有以下三个主要设计目标：

数据完整性：所有的数据都有一个[效验值](https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-checksum)。写入数据时，同时计算并写入效验值。当稍后重读该数据时，也会重算效验值。如果效验值不匹配，就认为是数据错误。ZFS 会在数据冗余可用时尝试自动更正错误。

池存储：物理存储设备是被添加到一个池中，并通过该共享池分配存储空间。空间对所有文件系统均可用，并能够通过向池中添加更多存储设备来扩容。

性能：提供多种缓存机制以提高性能。ARC 是一种基于内存的高级读缓存。使用 [L2ARC](https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-l2arc) 可以实现基于磁盘的二级读缓存，而使用 [ZIL](https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-zil) 则可以实现基于磁盘的同步写缓存。

功能特性和术语的完整列表，详见 [19.8 ZFS 的功能特性与术语](https://www.freebsd.org/doc/handbook/zfs-term.html)。

## 19.1 ZFS 为何与众不同

ZFS 与以往任何文件系统最显著的区别在于其不仅仅是一种文件系统。将传统意义上相互独立的卷管理和文件系统结合起来，让 ZFS 具备了独特的优势。文件系统开始知晓所用磁盘的基础结构。传统的文件系统一次只能在单个磁盘上进行创建，如果有两个磁盘就必须创建两个独立的文件系统。在传统的硬件级 RAID 配置中，可以通过主文件系统中的操作系统所实现的单块逻辑磁盘，将多个物理磁盘拼接以避免这一问题。即便是在诸如由 GEOM 提供的软件级 RAID 中，UFS 文件系统也工作在由 RAID 转换来的单一设备中。ZFS 的卷管理和文件系统的功能组合解决了这一问题，并让多个文件系统可以共享可用存储池而随意创建。ZFS 了解磁盘的物理布局的最大优势之一在于，当向池中追加磁盘时现有文件系统还可以自动扩容。并且该新增空间是对所有文件系统可用的。ZFS 还可以对不同的文件系统应用不同的特性，与创建单个文件系统相比，创建多个不同的文件系统和数据集具有更多的优势。
