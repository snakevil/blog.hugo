---
title: "【译】第 19 章：Z 文件系统㈢"
slug: "第19章：Z文件系统三"
date: 2020-05-12T10:45:39+08:00
toc: true
category:
    - Linux
    - 博闻广记
tag:
    - ZFS
    - FreeBSD
    - Linux
    - 自主翻译
---

[点击这里阅读原文。](https://www.freebsd.org/doc/handbook/zfs-zpool.html)

---

## 19.3 `zpool` 管理

ZFS 管理主要通过两个工具实现。`zpool` 工具负责操作池和添加、删除、替换、管理磁盘。[`zfs`](/2020/05/第19章Z文件系统四/) 工具负责创建、销毁和管理数据集，包括[文件系统][]和[卷][]。

<!--more-->

[文件系统]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-filesystem
[卷]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-volume
[虚拟设备类型]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-vdev
[副本]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-copies
[清理]: https://www.freebsd.org/doc/handbook/zfs-zpool.html#zfs-zpool-scrub
[降级]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-degraded
[重构]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-resilver
[联机]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-online
[故障]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-faulted
[fsck(8)]: https://www.freebsd.org/cgi/man.cgi?query=fsck&sektion=8&manpath=freebsd-release-ports
[替换]: https://www.freebsd.org/doc/handbook/zfs-zpool.html#zfs-zpool-replace
[zpool(8)]: https://www.freebsd.org/cgi/man.cgi?query=zpool&sektion=8&manpath=freebsd-release-ports
[gpart(8)]: https://www.freebsd.org/cgi/man.cgi?query=gpart&sektion=8&manpath=freebsd-release-ports

## 19.3.1 创建和销毁存储池

创建 ZFS 存储池（zpool）意味着需要做出许多近乎永久的决策，因为池在创建后是无法更改的。最重要的决策便是使用何种虚拟设备来对物理磁盘分组。全部可选的[虚拟设备类型][]请通过点击链接查看列表。绝大多数的虚拟设备类型是无法在池创建后继续添加磁盘的。可以这么操作的有镜像（能够添加磁盘到该虚拟设备）和带区（stripe，可以通过添加磁盘到虚拟设备的方式升级成镜像）。尽管可以通过添加更多的虚拟设备的方式来扩展池，其布局在创建后仍然是无法更改的。相反只能先备份数据，然后销毁并重新创建池。

可以直接创建一个简单的镜像池：

```sh
zpool create mypool mirror /dev/ada1 /dev/ada2
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada1    ONLINE       0     0     0
            ada2    ONLINE       0     0     0

errors: No known data errors
```

也可以在不同分组的磁盘间添加虚拟设备类型关键词，如 `mirror`，来同时创建多个虚拟设备：

```sh
zpool create mypool mirror /dev/ada1 /dev/ada2 mirror /dev/ada3 /dev/ada4
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada1    ONLINE       0     0     0
            ada2    ONLINE       0     0     0
          mirror-1  ONLINE       0     0     0
            ada3    ONLINE       0     0     0
            ada4    ONLINE       0     0     0

errors: No known data errors
```

使用磁盘分区也可以构建池。将 ZFS 放进单独分区里，这样磁盘就可以将其它分区用作它用。特别是制作引导分区和引导所需的文件系统。因此从池中的某块成员磁盘启动也是可以做到的。使用分区并不会损失 FreeBSD 的性能。而且使用分区还能让管理员只配置使用磁盘的部分容量，如果将来出现替换的磁盘容量较当前磁盘较小、但满足分区大小要求的情况，仍然可以替换成功。

下面的例子就是使用分区来创建 [RAID-Z2]() 池：

```sh
zpool create mypool raidz2 /dev/ada0p3 /dev/ada1p3 /dev/ada2p3 /dev/ada3p3 /dev/ada4p3 /dev/ada5p3
zpool status
```

不再需要的池可以尽早销毁，以释放并复用磁盘。不过要记得先卸载所有的数据集。如果数据集在被使用中，卸载操作就会失败，池也不会被销毁。使用 `-f` 参数可以强制完成池的销毁，但这会导致打开了数据集中文件的应用程序出现不可预料的后果。

## 19.3.2 添加和删除设备

有两种方法可以向池中添加磁盘：使用 `zpool attach` 向已有的虚拟设备中关联磁盘，或者使用 `zpool add` 将虚拟设备添加到池中。但只有部分[虚拟设备类型][]允许在虚拟设备创建之后继续添加磁盘。

单个磁盘创建的池缺少冗余。即便检测出损坏，也会因缺少更多的数据副本而无法修复。[副本][]特性有助于恢复诸如扇区损坏之类的小故障，但无法提供与镜像或 RAID-Z 同等级别的保护。对于单块磁盘虚拟设备创建的池，`zpool attach` 可以添加更多的磁盘到虚拟设备中，从而创建镜像。`zpool attach` 还可以向镜像组中添加更多的磁盘，以增加冗余并提高读取性能。如果要用于构建池的磁盘分了区，那么将第一块磁盘的布局复制到第二块中，再使用 `gpart backup` 和 `gpart restore` 会让构建操作变得更简单些。

下面的例子里，我们通过关联 `ada1p3` 使单磁盘（带区）虚拟设备 `ada0p3` 升级成镜像：

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          ada0p3    ONLINE       0     0     0

errors: No known data errors
```

```sh
zpool attach mypool ada0p3 ada1p3
```

```
Make sure to wait until resilver is done before rebooting.

If you boot from pool 'mypool', you may need to update
boot code on newly attached disk 'ada1p3'.

Assuming you use GPT partitioning and 'da0' is your new boot disk
you may use the following command:

        gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 da0
```

```sh
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada1
```

```
bootcode written to ada1
```

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Fri May 30 08:19:19 2014
        527M scanned out of 781M at 47.9M/s, 0h0m to go
        527M resilvered, 67.53% done
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada1p3  ONLINE       0     0     0  (resilvering)

errors: No known data errors
```

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: resilvered 781M in 0h0m with 0 errors on Fri May 30 08:15:58 2014
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada1p3  ONLINE       0     0     0

errors: No known data errors
```

当无法将磁盘添加到现有的虚拟设备中（如 RAID-Z 一样）时，可以变通为将另一个虚拟设备添加到池中。更多的虚拟设备可以提供更高的性能，将写操作分散到不同的虚拟设备中。每个虚拟设备负责自己的冗余。将不同的虚拟设备类型混合使用，如镜像和 RAID-Z 并用，虽然能做到但并不建议这么做。将无冗余的虚拟设备添加到包含了镜像或 RAID-Z 虚拟设备的池中，会对整个池内的数据都带来风险。写操作是分布式的，所以非冗余磁盘中的故障会导致一同写入池中的每一块数据都出现问题。

数据是分散在不同的虚拟设备中的。举个例子，由两个镜像虚拟设备组成的池等效于 RAID10，数据是写入在不同的镜像中的。空间应该做好分配以做到每个虚拟设备都同时写满。如果虚拟设备的剩余空间大小不一，那么更空的虚拟设备就会被写入更多的数据，导致性能受损。

如果要将其它设备关联到引导池，别忘了更新引导代码。

下例关联了第二个镜像组（`ada2p3` 和 `ada3p3`）到现有镜像中：

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: resilvered 781M in 0h0m with 0 errors on Fri May 30 08:19:35 2014
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada1p3  ONLINE       0     0     0

errors: No known data errors
```

```sh
zpool add mypool mirror ada2p3 ada3p3
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada2
```

```
bootcode written to ada2
```

```sh
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada3
```

```
bootcode written to ada3
```

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: scrub repaired 0 in 0h0m with 0 errors on Fri May 30 08:29:51 2014
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada1p3  ONLINE       0     0     0
          mirror-1  ONLINE       0     0     0
            ada2p3  ONLINE       0     0     0
            ada3p3  ONLINE       0     0     0

errors: No known data errors
```

目前无法从池中删除虚拟设备，从镜像中删除磁盘也必须由足够的剩余冗余。当镜像组中只剩一块磁盘时，其将自动变更为带区，一旦磁盘发生故障，整个池都会出问题。

从三向镜像组中删除磁盘：

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: scrub repaired 0 in 0h0m with 0 errors on Fri May 30 08:29:51 2014
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada1p3  ONLINE       0     0     0
            ada2p3  ONLINE       0     0     0

errors: No known data errors
```

```sh
zpool detach mypool ada2p3
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: scrub repaired 0 in 0h0m with 0 errors on Fri May 30 08:29:51 2014
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada1p3  ONLINE       0     0     0

errors: No known data errors
```

## 19.3.3 检查池状态

池状态非常重要。当某个设备离线时，当出现读、写或效验错误时，响应的错误计数就会累加。`status` 的输出内容在呈现了配置、池中的每个设备的状态和整个池的状态之外，还会告知最后一次[清理][]的详情和操作建议。

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: scrub repaired 0 in 2h25m with 0 errors on Sat Sep 14 04:25:50 2013
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          raidz2-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada1p3  ONLINE       0     0     0
            ada2p3  ONLINE       0     0     0
            ada3p3  ONLINE       0     0     0
            ada4p3  ONLINE       0     0     0
            ada5p3  ONLINE       0     0     0

errors: No known data errors
```

## 19.3.4 清除错误

错误发生时，读、写或效验计数是会累加的。使用 `zpool clear mypool` 可以清除错误消息，并重置计数器。这对发生错误时就会向管理员发送警报的自动脚本来说很重要。如果旧的错误尚未清除，新的错误是无法查看到的。

## 19.3.5 替换工作中的设备

在许多情况下都需要替换新盘。在替换读写中的磁盘时，我们需要让被替换的磁盘保障在线状态，避免池进入[降级][]状态，进而增加数据丢失的风险。`zpool replace` 会将原磁盘中的全部数据都复制到新磁盘中，然后原磁盘就会从虚拟设备中断开（替换成新盘）。如果新磁盘比原磁盘要大，可以对池进行扩容以使用新增的空间。详见[池的扩容](#1939-池的扩容)。

下例就替换了池里工作中的设备：

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada1p3  ONLINE       0     0     0

errors: No known data errors
```

```sh
zpool replace mypool ada1p3 ada2p3
```

```
Make sure to wait until resilver is done before rebooting.

If you boot from pool 'zroot', you may need to update
boot code on newly attached disk 'ada2p3'.

Assuming you use GPT partitioning and 'da0' is your new boot disk
you may use the following command:

        gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 da0
```

```sh
gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada2
zpool status
```

```
  pool: mypool
 state: ONLINE
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Mon Jun  2 14:21:35 2014
        604M scanned out of 781M at 46.5M/s, 0h0m to go
        604M resilvered, 77.39% done
config:

        NAME             STATE     READ WRITE CKSUM
        mypool           ONLINE       0     0     0
          mirror-0       ONLINE       0     0     0
            ada0p3       ONLINE       0     0     0
            replacing-1  ONLINE       0     0     0
              ada1p3     ONLINE       0     0     0
              ada2p3     ONLINE       0     0     0  (resilvering)

errors: No known data errors
```

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: resilvered 781M in 0h0m with 0 errors on Mon Jun  2 14:21:52 2014
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada2p3  ONLINE       0     0     0

errors: No known data errors
```

## 19.3.6 处理故障设备

当池中的磁盘出现故障时，其所属的虚拟设备会进入[降级][]状态。虽然所有的数据还是可用，但性能会因丢失的数据需要从冗余即时计算而下降。要将虚拟设备恢复为全功能状态，就需要替换掉故障的物理设备。然后指示 ZFS 做[重构（resilver）][重构]操作。从可用冗余中重新计算故障设备上的数据，并将其写入替换设备。完成后，虚拟设备重返[联机][]状态。

如果虚拟设备没有任何冗余，或因多个设备发生故障没有足够的冗余以作补偿，则池会进入[故障][]状态。如果池中丢失的设备越来越多，数据最终会完全损坏，只能从备份恢复。

替换故障磁盘时，需要使用设备的 GUID 而为磁盘名称。如果新设备的名字不变，那么在 `zpool replace` 时可以省略这一参数。

下例就使用 `zpool replace` 替换故障磁盘：

```sh
zpool status
```

```
  pool: mypool
 state: DEGRADED
status: One or more devices could not be opened.  Sufficient replicas exist for
        the pool to continue functioning in a degraded state.
action: Attach the missing device and online it using 'zpool online'.
   see: http://illumos.org/msg/ZFS-8000-2Q
  scan: none requested
config:

        NAME                    STATE     READ WRITE CKSUM
        mypool                  DEGRADED     0     0     0
          mirror-0              DEGRADED     0     0     0
            ada0p3              ONLINE       0     0     0
            316502962686821739  UNAVAIL      0     0     0  was /dev/ada1p3

errors: No known data errors
```

```sh
zpool replace mypool 316502962686821739 ada2p3
zpool status
```

```
  pool: mypool
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Mon Jun  2 14:52:21 2014
        641M scanned out of 781M at 49.3M/s, 0h0m to go
        640M resilvered, 82.04% done
config:

        NAME                        STATE     READ WRITE CKSUM
        mypool                      DEGRADED     0     0     0
          mirror-0                  DEGRADED     0     0     0
            ada0p3                  ONLINE       0     0     0
            replacing-1             UNAVAIL      0     0     0
              15732067398082357289  UNAVAIL      0     0     0  was /dev/ada1p3/old
              ada2p3                ONLINE       0     0     0  (resilvering)

errors: No known data errors
```

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: resilvered 781M in 0h0m with 0 errors on Mon Jun  2 14:52:38 2014
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada2p3  ONLINE       0     0     0

errors: No known data errors
```

## 19.3.7 池清理

建议定期[清理][]池，最好每月至少一次。`scrub` 操作非常占用磁盘空间并且在运行时会降低性能。制定清理的计划任务时应避免使用需求大的时段，或使用 [`vfs.zfs.scrub_delay`](/2020/05/第19章Z文件系统六/#1961-调优) 调节清理的相对优先级以免干扰其它工作负荷。

```sh
zpool scrub mypool
zpool status
```

```
  pool: mypool
 state: ONLINE
  scan: scrub in progress since Wed Feb 19 20:52:54 2014
        116G scanned out of 8.60T at 649M/s, 3h48m to go
        0 repaired, 1.32% done
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          raidz2-0  ONLINE       0     0     0
            ada0p3  ONLINE       0     0     0
            ada1p3  ONLINE       0     0     0
            ada2p3  ONLINE       0     0     0
            ada3p3  ONLINE       0     0     0
            ada4p3  ONLINE       0     0     0
            ada5p3  ONLINE       0     0     0

errors: No known data errors
```

如需取消清理操作，请使用 `zpool scrub -s mypool`。

## 19.3.8 自愈

与数据块一同存储的效验值使文件系统可以 _自愈_。这个特性在发现存储池中不同设备里的同一数据效验值不一致时，会尝试自动修复。例如，两个磁盘的镜像中有一个设备开始出现故障，无法再正确存储数据。当数据长时间不访问（长期存档性存储）时，情况就更糟了。传统的文件系统需要运行如 [fsck(8)][] 之类的特定算法检查和修复数据。这些指令都极耗时间。而在更严重的情况下，管理员只能自己想办法该如何修复。在 ZFS 中只要发现有数据块的效验值不一致，就会尝试从镜像磁盘中读取数据。从磁盘中读取到的正确数据，不仅会提供给调用数据的应用程序，还会修正另一块磁盘上的错误数据。这一机制会在池的正常操作间自动实施，无需系统管理员的任何干预。

下面演示 `/dev/ada0` 和 `/dev/ada1` 创建的镜像池是如何自愈的。

```sh
zpool create healer mirror /dev/ada0 /dev/ada1
zpool status healer
```

```
  pool: healer
 state: ONLINE
  scan: none requested
config:

    NAME        STATE     READ WRITE CKSUM
    healer      ONLINE       0     0     0
      mirror-0  ONLINE       0     0     0
       ada0     ONLINE       0     0     0
       ada1     ONLINE       0     0     0

errors: No known data errors
# zpool list
NAME     SIZE  ALLOC   FREE   CKPOINT  EXPANDSZ   FRAG   CAP  DEDUP  HEALTH  ALTROOT
healer   960M  92.5K   960M         -         -     0%    0%  1.00x  ONLINE  -
```

首先将某些需要通过自愈特性保护的重要数据复制到池中，并手动生成池的效验值以用于后继的比较。

```sh
cp /some/important/data /healer
zfs list
```

```
NAME     SIZE  ALLOC   FREE    CAP  DEDUP  HEALTH  ALTROOT
healer   960M  67.7M   892M     7%  1.00x  ONLINE  -
```

```sh
sha1 /healer > checksum.txt
cat checksum.txt
```

```
SHA1 (/healer) = 2753eff56d77d9a536ece6694bf0a82740344d1f
```

再通过向镜像的一块磁盘的头部写入随机数据来模拟错误。同样，为了避免 ZFS 会在发现错误时自愈，先将池导出，操作后再导回。

> **警告：**
>
> 此危险操作可能会破坏重要数据。仅用作演示，请勿在存储池的正常操作期间尝试使用。也不要在任何有文件系统的磁盘上执行。更不要使用不在池中的磁盘设备的名称。请确保在运行指令前已创建池的备份！

```sh
zpool export healer
dd if=/dev/random of=/dev/ada1 bs=1m count=200
```

```
200+0 records in
200+0 records out
209715200 bytes transferred in 62.992162 secs (3329227 bytes/sec)
```

```sh
zpool import healer
```

池的状态显示其中一个设备出现了错误。请注意应用程序并未从池中读取到错误的数据。因为 ZFS 从 `ada0` 设备中提供的是有正确效验值的数据。`CKSUM` 列的值不为零的设备就是出现了错误效验值的。

```sh
zpool status healer
```

```
    pool: healer
   state: ONLINE
  status: One or more devices has experienced an unrecoverable error.  An
          attempt was made to correct the error.  Applications are unaffected.
  action: Determine if the device needs to be replaced, and clear the errors
          using 'zpool clear' or replace the device with 'zpool replace'.
     see: http://illumos.org/msg/ZFS-8000-4J
    scan: none requested
  config:

      NAME        STATE     READ WRITE CKSUM
      healer      ONLINE       0     0     0
        mirror-0  ONLINE       0     0     0
         ada0     ONLINE       0     0     0
         ada1     ONLINE       0     0     1

errors: No known data errors
```

错误是使用不受影响的 `ada0` 镜像磁盘中存在的冗余来检测和处理的。然后再与原始效验值比较，以确认池是否恢复一致。

```sh
sha1 /healer >> checksum.txt
cat checksum.txt
```

```
SHA1 (/healer) = 2753eff56d77d9a536ece6694bf0a82740344d1f
SHA1 (/healer) = 2753eff56d77d9a536ece6694bf0a82740344d1f
```

故意篡改池数据前后两次生成的效验值是一致的。这说明 ZFS 是能够发现不匹配的效验值并自动校正的。需要注意这得有足够的冗余才能实现。仅由单个设备构成的池没有自愈能力。效验值在 ZFS 是如此的重要，无论如何也不该禁用。即便出现错误也不影响池的可用性，更无需 [fsck(8)][] 之类的文件系统一致性检查程序。现在需要执行清理操作来覆盖 `ada1` 中得错误数据。

```sh
zpool scrub healer
zpool status healer
```

```
  pool: healer
 state: ONLINE
status: One or more devices has experienced an unrecoverable error.  An
            attempt was made to correct the error.  Applications are unaffected.
action: Determine if the device needs to be replaced, and clear the errors
            using 'zpool clear' or replace the device with 'zpool replace'.
   see: http://illumos.org/msg/ZFS-8000-4J
  scan: scrub in progress since Mon Dec 10 12:23:30 2012
        10.4M scanned out of 67.0M at 267K/s, 0h3m to go
        9.63M repaired, 15.56% done
config:

    NAME        STATE     READ WRITE CKSUM
    healer      ONLINE       0     0     0
      mirror-0  ONLINE       0     0     0
       ada0     ONLINE       0     0     0
       ada1     ONLINE       0     0   627  (repairing)

errors: No known data errors
```

清理操作从 `ada0` 中读取数据并覆写 `ada1` 中所有效验值错误的数据。注意 `zpool status` 的输出中的 `(reparing)` 标志。操作完成后池状态会变更为：

```sh
zpool status healer
```

```
  pool: healer
 state: ONLINE
status: One or more devices has experienced an unrecoverable error.  An
        attempt was made to correct the error.  Applications are unaffected.
action: Determine if the device needs to be replaced, and clear the errors
             using 'zpool clear' or replace the device with 'zpool replace'.
   see: http://illumos.org/msg/ZFS-8000-4J
  scan: scrub repaired 66.5M in 0h2m with 0 errors on Mon Dec 10 12:26:25 2012
config:

    NAME        STATE     READ WRITE CKSUM
    healer      ONLINE       0     0     0
      mirror-0  ONLINE       0     0     0
       ada0     ONLINE       0     0     0
       ada1     ONLINE       0     0 2.72K

errors: No known data errors
```

清理操作完成且数据从 `ada0` 都同步到 `ada1` 后，执行 `zpool clear` 可以[清除](#1934-清除错误)池状态中的错误消息。

```sh
zpool clear healer
zpool status healer
```

```
  pool: healer
 state: ONLINE
  scan: scrub repaired 66.5M in 0h2m with 0 errors on Mon Dec 10 12:26:25 2012
config:

    NAME        STATE     READ WRITE CKSUM
    healer      ONLINE       0     0     0
      mirror-0  ONLINE       0     0     0
       ada0     ONLINE       0     0     0
       ada1     ONLINE       0     0     0

errors: No known data errors
```

这时池已经恢复到全功能状态，所有的错误也被清除掉了。

## 19.3.9 池的扩容

每个虚拟设备的冗余池的可用大小受限于其中单个设备的最小容量。将最小的设备更换为较大的设备，等[替换][]或[重构][]操作完成后，池就扩容可以使用新设备的容量了。例如，一个由 1TB 设备和 2TB 设备组成的镜像，其可用空间是 1TB。当使用另一个 2TB 设备替换 1TB 设备时，重构流程会将既有数据复制到新设备里。而且因为现在两个设备都是 2TB 容量，镜像的可用空间就扩容到了 2TB。

分区扩容（expansion）是通过在每台设备上使用 `zpool online -e` 执行的。当池中的所有设备都完成分区扩容后，整个池的可用空间也会得到增长。

## 19.3.10 导入和导出池

在将池移至另一个系统之前，先将其 _导出_ 。 所有数据集均已卸载，每个设备都标记为已导出，但仍处于锁定状态，因此其他磁盘子系统无法使用它。 这允许将池 _导入_ 到支持 ZFS 的其它计算机、其它操作系统，甚至是不同的硬件体系结构（有一些警告，请参阅 [zpool(8)][] 帮助文档）。使用 `zpool export -f` 可以在数据集打开文件时强行将池导出。但请注意，强制卸载数据集，可能会导致打开了其中文件的应用程序出现意外行为。

导出未使用的池：

```sh
zpool export mypool
```

导入池会自动挂载数据集。 如果这不是您想要的行为，可以通过 `zpool import -N` 来阻止。 `zpool import -o` 会为导入只设置临时属性。`zpool import altroot=` 允许将池导入到文件系统根目录以外的挂载点。如果一个池之前用在其他系统中且没有正确导出，则需通过 `zpool import -f` 强制导入。`zpool import -a` 会导入所有未被其它系统使用的池。

列举所有可导入的池：

```sh
zpool import
```

```
   pool: mypool
     id: 9930174748043525076
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        mypool      ONLINE
          ada2p3    ONLINE
```

将池导入到另外的根目录：

```sh
zpool import -o altroot=/mnt mypool
zfs list
```

```
zfs list
NAME                 USED  AVAIL  REFER  MOUNTPOINT
mypool               110K  47.0G    31K  /mnt/mypool
```

## 19.3.11 升级存储池

在升级 FreeBSD 后，或将池从一个 ZFS 版本更陈旧的系统中导入时，可以手动将池升级到最新的 ZFS 版本以支持更新的特性。但在升级前请慎重考量该池是否还会用回到其它陈旧的系统中。升级是一件单向操作。旧的池可以升级，但新池无法降级。

升级一个 v28 版本的池以支持特性标记：

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
status: The pool is formatted using a legacy on-disk format.  The pool can
        still be used, but some features are unavailable.
action: Upgrade the pool using 'zpool upgrade'.  Once this is done, the
        pool will no longer be accessible on software that does not support feat
        flags.
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
	    ada0    ONLINE       0     0     0
	    ada1    ONLINE       0     0     0

errors: No known data errors
```

```sh
zpool upgrade
```

```
This system supports ZFS pool feature flags.

The following pools are formatted with legacy version numbers and can
be upgraded to use feature flags.  After being upgraded, these pools
will no longer be accessible by software that does not support feature
flags.

VER  POOL
---  ------------
28   mypool

Use 'zpool upgrade -v' for a list of available legacy versions.
Every feature flags pool has all supported features enabled.
```

```sh
zpool upgrade mypool
```

```
This system supports ZFS pool feature flags.

Successfully upgraded 'mypool' from version 28 to feature flags.
Enabled the following features on 'mypool':
  async_destroy
  empty_bpobj
  lz4_compress
  multi_vdev_crash_dump
```

在完成 `zpool upgrade` 前无法使用 ZFS 的新特性的。`zpool upgrade -v` 可以查看升级会新增哪些特性，以及当前以及支持了哪些特性。

升级池以支持更多的特性标记：

```sh
zpool status
```

```
  pool: mypool
 state: ONLINE
status: Some supported features are not enabled on the pool. The pool can
        still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        mypool      ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
	    ada0    ONLINE       0     0     0
	    ada1    ONLINE       0     0     0

errors: No known data errors
```

```sh
zpool upgrade
```

```
This system supports ZFS pool feature flags.

All pools are formatted using feature flags.


Some supported features are not enabled on the following pools. Once a
feature is enabled the pool may become incompatible with software
that does not support the feature. See zpool-features(7) for details.

POOL  FEATURE
---------------
zstore
      multi_vdev_crash_dump
      spacemap_histogram
      enabled_txg
      hole_birth
      extensible_dataset
      bookmarks
      filesystem_limits
```

```sh
zpool upgrade mypool
```

```
This system supports ZFS pool feature flags.

Enabled the following features on 'mypool':
  spacemap_histogram
  enabled_txg
  hole_birth
  extensible_dataset
  bookmarks
  filesystem_limits
```

> **警告：**
>
> 从池引导的系统的引导代码也必须升级，以兼容池的新版本。在包含引导代码的分区执行 `gpart bootcode`。针对不同的系统引导方式，有两种引导代码类型：GPT（最常见的选项）和 EFI（更现代的系统）。
>
> 对使用 GPT 的旧引导，使用：
>
> ```sh
> gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada1
> ```
>
> 要引导使用 EFI 的系统，执行：
>
> ```sh
> gpart bootcode -p /boot/boot1.efifat -i 1 ada1
> ```
>
> 池中所有可引导的磁盘都会更新引导代码。详情请查看 [gpart(8)][] 的参考手册。

## 19.3.12 显示池的历史记录

所有修改池的指令都会被记录下来。包括创建数据集、更改属性或替换磁盘。有助于检索池于何时创建，又曾被哪个用户在何时做过什么操作。这些记录并非在某个日志文件中，而是池本身信息的一部分。查看历史记录的命令就是 `zpool history`：

```sh
zpool history
```

```
History for 'tank':
2013-02-26.23:02:35 zpool create tank mirror /dev/ada0 /dev/ada1
2013-02-27.18:50:58 zfs set atime=off tank
2013-02-27.18:51:09 zfs set checksum=fletcher4 tank
2013-02-27.18:51:18 zfs create tank/backup
```

输出显示了在某个时间点执行过 `zpool` 和 `zfs` 指令。只有改变过池的指令才会被记录。并不包括诸如 `zfs list` 之类的指令。如果未指明池的名称，所有池的所有历史记录都会显示。

`zpool history` 再配合以 `-i` 或 `-l` 选项还可以呈现更多信息。`-i` 显示内部记录的 ZFS 用户初始化事件。

```sh
zpool history -i
```

```
History for 'tank':
2013-02-26.23:02:35 [internal pool create txg:5] pool spa 28; zfs spa 28; zpl 5;uts  9.1-RELEASE 901000 amd64
2013-02-27.18:50:53 [internal property set txg:50] atime=0 dataset = 21
2013-02-27.18:50:58 zfs set atime=off tank
2013-02-27.18:51:04 [internal property set txg:53] checksum=7 dataset = 21
2013-02-27.18:51:09 zfs set checksum=fletcher4 tank
2013-02-27.18:51:13 [internal create txg:55] dataset = 39
2013-02-27.18:51:18 zfs create tank/backup
```

使用 `-l` 则可以呈现更多细节。历史记录会以长格式显示，包括执行指令的用户名信息和发生变更时的机器名称。

```sh
zpool history -l
```

```
History for 'tank':
2013-02-26.23:02:35 zpool create tank mirror /dev/ada0 /dev/ada1 [user 0 (root) on :global]
2013-02-27.18:50:58 zfs set atime=off tank [user 0 (root) on myzfsbox:global]
2013-02-27.18:51:09 zfs set checksum=fletcher4 tank [user 0 (root) on myzfsbox:global]
2013-02-27.18:51:18 zfs create tank/backup [user 0 (root) on myzfsbox:global]
```

输出显示 `root` 用户使用 `/dev/ada0` 和 `/dev/ada1` 磁盘创建了镜像池。之后的指令中也显示了机器名称 `myzfsbox`。当池曾经从一个系统中导出、再导回到另一系统时，机器名称信息就很重要了。每条指令记录的及其名称能清楚地区分不同的指令都发自哪个系统。

如有需要还可以将 `zpool history` 的两个选项组合起来使用以获取最详细的信息。当追踪执行过的操作时，当需要更详尽的信息以便调试时，历史记录就能帮上忙了。

## 19.3.13 性能监控

内置的监控系统可以实时显示池的 I/O 指标。包括池的已用和剩余空间，每秒的读写操作数，和当前占用的 I/O 带宽。系统中的所有池默认都会被监控并显示。如果只想查看一个池那么请指定池名称。例如：

```sh
zpool iostat
```

```
               capacity     operations    bandwidth
pool        alloc   free   read  write   read  write
----------  -----  -----  -----  -----  -----  -----
data         288G  1.53T      2     11  11.3K  57.1K
```

如需持续监控 I/O 动态，设置一个数字作为最后的参数，即会以其秒数为单位周期性地更新监控。每次显示新的一行。通过 `Ctrl-C` 可以中止持续监控。又或者再设置一个数字作为最最后的参数，可以只持续监控该数量的周期。

`-v` 可以呈现更详尽的 I/O 指标。池中的每个设备的信息显示为一行。这有助于查看每个设备的读写操作数，以帮助判断是否有某个设备拖慢了整个池。下例展示了两个设备组件的镜像池的信息：

```sh
zpool iostat -v
```

```
                            capacity     operations    bandwidth
pool                     alloc   free   read  write   read  write
-----------------------  -----  -----  -----  -----  -----  -----
data                      288G  1.53T      2     12  9.23K  61.5K
  mirror                  288G  1.53T      2     12  9.23K  61.5K
    ada1                     -      -      0      4  5.61K  61.7K
    ada2                     -      -      1      4  5.04K  61.7K
-----------------------  -----  -----  -----  -----  -----  -----
```

## 19.3.14 拆分存储池

囊括了一个或多个镜像虚拟设备的池是可以被拆分为两个池的。除非另行说明，否则每个镜像的最后一个成员将被分离出来，用于创建包含了同样数据的新池。真正操作前可以通过 `-n` 模拟查看操作的细节。这有助于确认该操作是否符合预期。
