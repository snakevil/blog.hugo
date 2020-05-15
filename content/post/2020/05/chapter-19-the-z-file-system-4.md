---
title: "【译】第 19 章：Z 文件系统㈣"
slug: "第19章：Z文件系统四"
date: 2020-05-13T03:35:12+08:00
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

[点击这里阅读原文。](https://www.freebsd.org/doc/handbook/zfs-zfs.html)

---

## 19.4 `zfs` 管理

`zfs` 工具负责创建、销毁和管理池中的 ZFS 数据集。管理池需要使用 `zpool`。

<!--more-->

[委派]: /2020/05/第19章Z文件系统五/
[数据集]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-dataset
[lz4]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-compression-lz4
[快照]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-snapshot
[cow]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-cow
[cron(8)]: https://www.freebsd.org/cgi/man.cgi?query=cron&sektion=8&manpath=freebsd-release-ports
[数据集配额]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-quota
[引用配额]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-refquota
[用户配额]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-userquota
[组配额]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-groupquota
[zfs(1)]: https://www.freebsd.org/cgi/man.cgi?query=zfs&sektion=1&manpath=freebsd-release-ports
[预留]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-reservation
[引用预留]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-refreservation
[压缩]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-compression
[去重]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-deduplication

## 19.4.1 创建和销毁数据集

与传统的磁盘和卷管理器不同，ZFS 中的空间并不会被预分配。在传统的文件系统中，当所有空间都被分区后，除了添加新磁盘外是无法新增文件系统的。而在 ZFS 中可以随时创建新的文件系统。每个[数据集][]都有如压缩、去重、缓存和配额等多项特性设置，也有如只读、大小写敏感、网络文件共享和挂载点等有用的属性。数据集可以相互嵌套，子集会集成父集的属性设置。每个数据集都可以作为一个单元进行管理、[委派][]、[复制](#1947-同步)、[快照](#1945-管理快照)、[监禁](#19412-zfs-和监狱)和销毁。为每种不同的文件类型或文件集创建单独的数据集好处良多。大量数据集的唯一弊端只是某些如 `zfs list` 的指令变得慢了些，而当挂载的数据集成百上千时 FreeBSD 的引导进程也会慢一些。

创建新数据集并启用 [LZ4 压缩][lz4]：

```sh
zfs list
```

```
NAME                  USED  AVAIL  REFER  MOUNTPOINT
mypool                781M  93.2G   144K  none
mypool/ROOT           777M  93.2G   144K  none
mypool/ROOT/default   777M  93.2G   777M  /
mypool/tmp            176K  93.2G   176K  /tmp
mypool/usr            616K  93.2G   144K  /usr
mypool/usr/home       184K  93.2G   184K  /usr/home
mypool/usr/ports      144K  93.2G   144K  /usr/ports
mypool/usr/src        144K  93.2G   144K  /usr/src
mypool/var           1.20M  93.2G   608K  /var
mypool/var/crash      148K  93.2G   148K  /var/crash
mypool/var/log        178K  93.2G   178K  /var/log
mypool/var/mail       144K  93.2G   144K  /var/mail
mypool/var/tmp        152K  93.2G   152K  /var/tmp
```

```sh
zfs create -o compress=lz4 mypool/usr/mydataset
zfs list
```

```
NAME                   USED  AVAIL  REFER  MOUNTPOINT
mypool                 781M  93.2G   144K  none
mypool/ROOT            777M  93.2G   144K  none
mypool/ROOT/default    777M  93.2G   777M  /
mypool/tmp             176K  93.2G   176K  /tmp
mypool/usr             704K  93.2G   144K  /usr
mypool/usr/home        184K  93.2G   184K  /usr/home
mypool/usr/mydataset  87.5K  93.2G  87.5K  /usr/mydataset
mypool/usr/ports       144K  93.2G   144K  /usr/ports
mypool/usr/src         144K  93.2G   144K  /usr/src
mypool/var            1.20M  93.2G   610K  /var
mypool/var/crash       148K  93.2G   148K  /var/crash
mypool/var/log         178K  93.2G   178K  /var/log
mypool/var/mail        144K  93.2G   144K  /var/mail
mypool/var/tmp         152K  93.2G   152K  /var/tmp
```

直接销毁数据集比删除其中的全部文件要快得多，因为省去了扫描全部文件并更新每个文件的元信息的操作。

销毁之前创建的数据集：

```sh
zfs list
```

```
NAME                   USED  AVAIL  REFER  MOUNTPOINT
mypool                 880M  93.1G   144K  none
mypool/ROOT            777M  93.1G   144K  none
mypool/ROOT/default    777M  93.1G   777M  /
mypool/tmp             176K  93.1G   176K  /tmp
mypool/usr             101M  93.1G   144K  /usr
mypool/usr/home        184K  93.1G   184K  /usr/home
mypool/usr/mydataset   100M  93.1G   100M  /usr/mydataset
mypool/usr/ports       144K  93.1G   144K  /usr/ports
mypool/usr/src         144K  93.1G   144K  /usr/src
mypool/var            1.20M  93.1G   610K  /var
mypool/var/crash       148K  93.1G   148K  /var/crash
mypool/var/log         178K  93.1G   178K  /var/log
mypool/var/mail        144K  93.1G   144K  /var/mail
mypool/var/tmp         152K  93.1G   152K  /var/tmp
```

```sh
zfs destroy mypool/usr/mydataset
zfs list
```

```
NAME                  USED  AVAIL  REFER  MOUNTPOINT
mypool                781M  93.2G   144K  none
mypool/ROOT           777M  93.2G   144K  none
mypool/ROOT/default   777M  93.2G   777M  /
mypool/tmp            176K  93.2G   176K  /tmp
mypool/usr            616K  93.2G   144K  /usr
mypool/usr/home       184K  93.2G   184K  /usr/home
mypool/usr/ports      144K  93.2G   144K  /usr/ports
mypool/usr/src        144K  93.2G   144K  /usr/src
mypool/var           1.21M  93.2G   612K  /var
mypool/var/crash      148K  93.2G   148K  /var/crash
mypool/var/log        178K  93.2G   178K  /var/log
mypool/var/mail       144K  93.2G   144K  /var/mail
mypool/var/tmp        152K  93.2G   152K  /var/tmp
```

现代版本的 ZFS 中，`zfs destroy` 是异步操作，空间可能需要几分钟甚至更久才会释放。使用 `zpool get freeing poolname` 可以通过查看 `freeing` 属性判断有多少数据集正在后台释放空间。只有使用 `-r` 才能递归销毁数据集和其全部子集。使用 `-n -v` 可以模拟列举出操作将会被销毁的数据集和[快照][]。包括通过销毁快照可以回收的空间。

## 19.4.2 创建和销毁卷

卷是特殊类型的数据集。其不再作为文件系统挂载，而是作为 `/dev/zvol/poolname/dataset` 形式的块设备。以供其它文件系统使用，如用作虚拟机的磁盘，或通过 iSCSI 或 HAST 等协议导出。

卷可以由任何文件系统格式化，但也可以直接存储原始数据。对用户而言，卷就是普通磁盘。这些 ZFS 卷中的普通文件系统也能获得普通磁盘或文件系统所没有的特性。例如，在一个 250MB 大小的卷上启用压缩，可以创建一个压缩的 FAT 文件系统。

```sh
zfs create -V 250m -o compression=on tank/fat32
zfs list tank
```

```
NAME USED AVAIL REFER MOUNTPOINT
tank 258M  670M   31K /tank
```

```sh
newfs_msdos -F32 /dev/zvol/tank/fat32
mount -t msdosfs /dev/zvol/tank/fat32 /mnt
df -h /mnt | grep fat32
```

```
Filesystem           Size Used Avail Capacity Mounted on
/dev/zvol/tank/fat32 249M  24k  249M     0%   /mnt
```

```sh
mount | grep fat32
```

```
/dev/zvol/tank/fat32 on /mnt (msdosfs, local)
```

销毁卷与销毁常规的文件系统数据集差不多。该操作几乎使瞬时完成，但在后台回收空间可能需要几分钟。

## 19.4.3 数据集改名

`zfs rename` 可以修改数据集的名字。也可以用来修改数据集的父集。此时原本从父集继承的属性设置也会变更为新的父集的设置。改名时会先将数据集卸载，然后重新挂载到新的位置（从新的父集继承而来）。使用 `-u` 可以阻止这一行为。

重命名数据集并移动至别的父集下：

```sh
zfs list
```

```
NAME                   USED  AVAIL  REFER  MOUNTPOINT
mypool                 780M  93.2G   144K  none
mypool/ROOT            777M  93.2G   144K  none
mypool/ROOT/default    777M  93.2G   777M  /
mypool/tmp             176K  93.2G   176K  /tmp
mypool/usr             704K  93.2G   144K  /usr
mypool/usr/home        184K  93.2G   184K  /usr/home
mypool/usr/mydataset  87.5K  93.2G  87.5K  /usr/mydataset
mypool/usr/ports       144K  93.2G   144K  /usr/ports
mypool/usr/src         144K  93.2G   144K  /usr/src
mypool/var            1.21M  93.2G   614K  /var
mypool/var/crash       148K  93.2G   148K  /var/crash
mypool/var/log         178K  93.2G   178K  /var/log
mypool/var/mail        144K  93.2G   144K  /var/mail
mypool/var/tmp         152K  93.2G   152K  /var/tmp
```

```sh
zfs rename mypool/usr/mydataset mypool/var/newname
zfs list
```

```
NAME                  USED  AVAIL  REFER  MOUNTPOINT
mypool                780M  93.2G   144K  none
mypool/ROOT           777M  93.2G   144K  none
mypool/ROOT/default   777M  93.2G   777M  /
mypool/tmp            176K  93.2G   176K  /tmp
mypool/usr            616K  93.2G   144K  /usr
mypool/usr/home       184K  93.2G   184K  /usr/home
mypool/usr/ports      144K  93.2G   144K  /usr/ports
mypool/usr/src        144K  93.2G   144K  /usr/src
mypool/var           1.29M  93.2G   614K  /var
mypool/var/crash      148K  93.2G   148K  /var/crash
mypool/var/log        178K  93.2G   178K  /var/log
mypool/var/mail       144K  93.2G   144K  /var/mail
mypool/var/newname   87.5K  93.2G  87.5K  /var/newname
mypool/var/tmp        152K  93.2G   152K  /var/tmp
```

快照也可以通过相似的方式改名。但特殊之处在于，无法将其重命名至其它父集中。指定 `-r` 可以将快照及所有子数据集中的同名快照递归改名。

```sh
zfs list -t snapshot
```

```
NAME                                USED  AVAIL  REFER  MOUNTPOINT
mypool/var/newname@first_snapshot      0      -  87.5K  -
```

```sh
zfs rename mypool/var/newname@first_snapshot new_snapshot_name
zfs list -t snapshot
```

```
NAME                                   USED  AVAIL  REFER  MOUNTPOINT
mypool/var/newname@new_snapshot_name      0      -  87.5K  -
```

## 19.4.4 设置数据集属性

每个 ZFS 数据集都有许多属性以控制其行为。大部分属性都从父集自动继承而来，并可重设覆盖。使用 `zfs set property=value dataset` 进行设置。大部分属性都只有有限个数的有效值，`zfs get` 可以查看所有可用的属性和有效值。使用 `zfs inherit` 可以将大部分属性值还原至其继承值。

也可以设置用户自定义属性。它们会成为数据集配置的一部分，以提供数据集或其中内容的额外信息。为了便于识别，自定义属性都需要通过 `:` 冒号声明命名空间。

```sh
zfs set custom:costcenter=1234 tank
zfs get custom:costcenter tank
```

```
NAME PROPERTY           VALUE SOURCE
tank custom:costcenter  1234  local
```

删除自定义属性需要使用 `zfs inherit -r`。如果该属性并未定义在任何父集上，其设置就会被彻底删除（当然变更操作还是会被记录在池的历史记录里）。

```sh
zfs inherit -r custom:costcenter tank
zfs get custom:costcenter tank
```

```
NAME    PROPERTY           VALUE              SOURCE
tank    custom:costcenter  -                  -
```

```sh
zfs get all tank | grep custom:costcenter
```

### 19.4.4.1 获取和设置共享属性

NFS 和 SMB 共享选项时两个常用且有用的数据集属性。其规定了 ZFS 数据集是否和如何被共享在网络上的。目前 FreeBSD 只支持 NFS 共享。要获取当前的共享状态，请输入：

```sh
zfs get sharenfs mypool/usr/home
```

```
NAME             PROPERTY  VALUE    SOURCE
mypool/usr/home  sharenfs  on       local
```

```sh
zfs get sharesmb mypool/usr/home
```

```
NAME             PROPERTY  VALUE    SOURCE
mypool/usr/home  sharesmb  off      local
```

开始共享数据集：

```sh
zfs set sharenfs=on mypool/usr/home
```

使用 NFS 共享数据集时还可以设置额外的选项，如：`-alldirs`、`-maproot` 和 `-network`。例如：

```sh
zfs set sharenfs="-alldirs,-maproot=root,-network=192.168.1.0/24" mypool/usr/home
```

## 19.4.5 管理快照

[快照][]是 ZFS 最强大的功能之一。提供了数据集在指定时间的只读副本。基于[写时复制（Copy-On-Write，COW）][cow]，可以在磁盘上保留较旧版本数据的方式快速创建快照。如果没有快照，当数据被重写或删除时将回收空间以供将来使用。快照仅记录当前数据集和先前版本之间的差异，以此节约磁盘空间。快照只生效于整个数据集，而不包括单个文件或目录。从数据集创建快照，会复制数据集中包含的所有内容。包括文件系统属性、文件、目录和权限等。快照在创建时并不占用空间，只有其引用的块发生改变才消耗。使用 `-r` 创建的递归快照会在数据集及其所有子集上创建一个同名的、确保全部文件系统的瞬时一致性的快照。当应用程序在多个相互关联或依赖的数据集上都写有文件时，这个机制就很有用了。如果没有快照，不同数据集的备份对应的时间点是不一致的。

ZFS 快照提供了比其它具有快照功能的文件系统还有多的功能特性。快照的一个典型用途是在执行诸如软件安装或系统升级之类的危险操作前，快速备份文件系统的当前状态。如果操作失败，就可以将系统回滚到创建快照时的状态。升级成功则删除快照释放空间。如果没有快照，失败的升级行为大都需要从备份恢复，系统会长时间无法使用，太过繁琐。而快照的回滚操作之快速，对系统的正常运行几乎没有影响。TB 量级的存储系统可因此节省大量的时间。快照并非完整备份的替代品，但为某些场合提供了快速而简单的创建数据集副本的方法。

### 19.4.5.1 创建快照

通过 `zfs snapshot dataset@snapshotname` 创建快照。`-r` 可以创建包含了全部子数据集的递归快照。

创建整个池的递归快照：

```sh
zfs list -t all
```

```
NAME                                   USED  AVAIL  REFER  MOUNTPOINT
mypool                                 780M  93.2G   144K  none
mypool/ROOT                            777M  93.2G   144K  none
mypool/ROOT/default                    777M  93.2G   777M  /
mypool/tmp                             176K  93.2G   176K  /tmp
mypool/usr                             616K  93.2G   144K  /usr
mypool/usr/home                        184K  93.2G   184K  /usr/home
mypool/usr/ports                       144K  93.2G   144K  /usr/ports
mypool/usr/src                         144K  93.2G   144K  /usr/src
mypool/var                            1.29M  93.2G   616K  /var
mypool/var/crash                       148K  93.2G   148K  /var/crash
mypool/var/log                         178K  93.2G   178K  /var/log
mypool/var/mail                        144K  93.2G   144K  /var/mail
mypool/var/newname                    87.5K  93.2G  87.5K  /var/newname
mypool/var/newname@new_snapshot_name      0      -  87.5K  -
mypool/var/tmp                         152K  93.2G   152K  /var/tmp
```

```sh
zfs snapshot -r mypool@my_recursive_snapshot
zfs list -t snapshot
```

```
NAME                                        USED  AVAIL  REFER  MOUNTPOINT
mypool@my_recursive_snapshot                   0      -   144K  -
mypool/ROOT@my_recursive_snapshot              0      -   144K  -
mypool/ROOT/default@my_recursive_snapshot      0      -   777M  -
mypool/tmp@my_recursive_snapshot               0      -   176K  -
mypool/usr@my_recursive_snapshot               0      -   144K  -
mypool/usr/home@my_recursive_snapshot          0      -   184K  -
mypool/usr/ports@my_recursive_snapshot         0      -   144K  -
mypool/usr/src@my_recursive_snapshot           0      -   144K  -
mypool/var@my_recursive_snapshot               0      -   616K  -
mypool/var/crash@my_recursive_snapshot         0      -   148K  -
mypool/var/log@my_recursive_snapshot           0      -   178K  -
mypool/var/mail@my_recursive_snapshot          0      -   144K  -
mypool/var/newname@new_snapshot_name           0      -  87.5K  -
mypool/var/newname@my_recursive_snapshot       0      -  87.5K  -
mypool/var/tmp@my_recursive_snapshot           0      -   152K  -
```

常规 `zfs list` 操作无法看到快照，得加上 `-t snapshot` 选项才行。只使用 `-t` 会显示全部文件系统的快照。

快照无法直接挂载，因此 `MOUNTPOINT` 列中没有内容。而 `AVAIL` 列也没有内容，是因为快照在创建后是无法写入新内容的。比较一下数据集及生成的快照：

```sh
zfs list -rt all mypool/usr/home
```

```
NAME                                    USED  AVAIL  REFER  MOUNTPOINT
mypool/usr/home                         184K  93.2G   184K  /usr/home
mypool/usr/home@my_recursive_snapshot      0      -   184K  -
```

同时显示数据集和快照可以看出有多少快照以 [COW][] 方式工作。即只保存更改（增量），而非文件系统中的全部内容。也就是说如果几乎不做更改，那么快照就几乎不占空间。将文件复制到数据集后再制作快照，可以更明显地看到空间使用情况：

```sh
cp /etc/passwd /var/tmp
zfs snapshot mypool/var/tmp@after_cp
zfs list -rt all mypool/var/tmp
```

```
NAME                                   USED  AVAIL  REFER  MOUNTPOINT
mypool/var/tmp                         206K  93.2G   118K  /var/tmp
mypool/var/tmp@my_recursive_snapshot    88K      -   152K  -
mypool/var/tmp@after_cp                   0      -   118K  -
```

第二个快照只包含数据集在复制行为之后的变更。这样节约了大量空间。需要注意快照 `mypool/var/tmp@my_recursive_snapshot` 的 `USED` 列也发生了变化，因为新的行为发生了。

### 19.4.5.2 比较快照

ZFS 提供了内置工具了比较两份快照内容的不同。有利于用户从大量的快照中寻找系统是如何变化的。例如，`zfs diff` 可以找出包含了某个被误删文件的最近期的快照。对上一节中创建的两个快照执行此操作，将产生以下输出：

```sh
zfs list -rt all mypool/var/tmp
```

```
NAME                                   USED  AVAIL  REFER  MOUNTPOINT
mypool/var/tmp                         206K  93.2G   118K  /var/tmp
mypool/var/tmp@my_recursive_snapshot    88K      -   152K  -
mypool/var/tmp@after_cp                   0      -   118K  -
```

```sh
zfs diff mypool/var/tmp@my_recursive_snapshot
```

```
M       /var/tmp/
+       /var/tmp/passwd
```

指令列举了指定快照 `mypool/var/tmp@my_recursive_snapshot` 与当前文件系统间的变更。首列用于描述变更的类型：

| 符号 | 变更类型   |
| :--: | :--------- |
|  +   | 新增文件   |
|  -   | 删除文件   |
|  M   | 修改文件   |
|  R   | 重命名文件 |

因此可知在快照 `mypool/var/tmp@my_recursive_snapshot` 创建后新增了 `passwd` 文件，因此挂载于 `/var/tmp` 的父目录也发生了修改。

使用 ZFS 复制功能将数据集传输到其他主机以进行备份时，比较两个快照可能会有所帮助。

通过提供两个数据集的完整数据集名称和快照名称来比较两个快照：

```sh
cp /var/tmp/passwd /var/tmp/passwd.copy
zfs snapshot mypool/var/tmp@diff_snapshot
zfs diff mypool/var/tmp@my_recursive_snapshot mypool/var/tmp@diff_snapshot
```

```
M       /var/tmp/
+       /var/tmp/passwd
+       /var/tmp/passwd.copy
```

```sh
zfs diff mypool/var/tmp@my_recursive_snapshot mypool/var/tmp@after_cp
```

```
M       /var/tmp/
+       /var/tmp/passwd
```

备份管理员可以通过比较收到的两份快照以确定数据集中究竟发生了什么变更。更多详情请查阅[复制]()章节。

### 19.4.5.3 回滚快照

当快照存在时，随时都可以执行回滚。大多数情况下希望将数据集恢复回早期版本时就是这样的操作。如本地研发测试方案出了错，又如系统更新影响了原本的完整功能，再如需要恢复被误删的文件或目录。幸好只需要 `zfs rollback snapshotname` 就可以轻松完成操作。操作耗时视其涉及的变更量而定。在此期间，数据集仍然保持着一致性状态，有些类似于符合 ACID 原则的数据库做回滚操作。只要数据集处于活动状态即可实时完成。回滚完成后，数据集将具有创建快照时的相同状态。未记录在快照中的数据将被丢弃。如果觉得以后可能会用到当前状态中的数据，可在回滚前多创建一个快照。这样就可以在不同的快照之间来回滚动了。

在第一个例子中，因为粗心的 `rm` 操作了很多数据，回滚快照。

```sh
zfs list -rt all mypool/var/tmp
```

```
NAME                                   USED  AVAIL  REFER  MOUNTPOINT
mypool/var/tmp                         262K  93.2G   120K  /var/tmp
mypool/var/tmp@my_recursive_snapshot    88K      -   152K  -
mypool/var/tmp@after_cp               53.5K      -   118K  -
mypool/var/tmp@diff_snapshot              0      -   120K  -
```

```sh
ls /var/tmp
```

```
passwd          passwd.copy     vi.recover
```

```sh
rm /var/tmp/passwd*
ls /var/tmp
```

```
vi.recover
```

此时用户已经意识到删除了太多文件，希望将其取回。ZFS 确实提供了回滚方法，但前提是有定期对重要数据做快照。用从上个快照中取回文件，执行：

```sh
zfs rollback mypool/var/tmp@diff_snapshot
ls /var/tmp
```

```
passwd          passwd.copy     vi.recover
```

回滚照做将数据集还原到最后的快照状态。当然也可以回滚到数个快照之前的更早快照。但此时 ZFS 会警告：

```sh
zfs list -rt snapshot mypool/var/tmp
```

```
NAME                                   USED  AVAIL  REFER  MOUNTPOINT
mypool/var/tmp@my_recursive_snapshot    88K      -   152K  -
mypool/var/tmp@after_cp               53.5K      -   118K  -
mypool/var/tmp@diff_snapshot              0      -   120K  -
```

```sh
zfs rollback mypool/var/tmp@my_recursive_snapshot
```

```
cannot rollback to 'mypool/var/tmp@my_recursive_snapshot': more recent snapshots exist
use '-r' to force deletion of the following snapshots:
mypool/var/tmp@after_cp
mypool/var/tmp@diff_snapshot
```

警告强调数据集在当前状态和回滚目标快照之间还有更多的快照。如果完成回滚，这些快照会被删除。ZFS 无法追踪数据集的不同状态间的全部变更，因为快照是只读的。除非用户使用 `-r` 确认这就是预期操作，否则 ZFS 不会删除受影响的快照。如果一定要这么做，而且也理解了丢失中间阶段的快照的后果，可以执行：

```sh
zfs rollback -r mypool/var/tmp@my_recursive_snapshot
zfs list -rt snapshot mypool/var/tmp
```

```
NAME                                   USED  AVAIL  REFER  MOUNTPOINT
mypool/var/tmp@my_recursive_snapshot     8K      -   152K  -
```

```sh
ls /var/tmp
```

```
vi.recover
```

`zfs list -t snaphost` 的输出也说明中间阶段的快照确实被删除掉了。

### 19.4.5.4 从快照中恢复单个文件

快照其实是挂载在父数据集中的隐藏目录 `.zfs/snapshots/snapshotname` 上。但即便使用 `ls -a` 也无法看到这些目录。不过这并不影响像普通目录一样直接对其进行访问。属性 `snapdir` 控制了这些影响目录的显示与否。将其设为 `visible` 后再执行 `ls` 和其它处理目录内容的指令就可以看到了。

```sh
zfs get snapdir mypool/var/tmp
```

```
NAME            PROPERTY  VALUE    SOURCE
mypool/var/tmp  snapdir   hidden   default
```

```sh
ls -a /var/tmp
```

```
.               ..              passwd          vi.recover
```

```sh
zfs set snapdir=visible mypool/var/tmp
ls -a /var/tmp
```

```
.               ..              .zfs            passwd          vi.recover
```

因此将快照中的数据复制到父数据集中，就可以轻松将单个文件还原到预期的状态了。`.zfs/snapshot` 中的目录结构与创建时一致，以便于识别。下面的例子展示了如何将一个文件从最新的快照中还原：

```sh
rm /var/tmp/passwd
ls -a /var/tmp
```

```
.               ..              .zfs            vi.recover
```

```sh
ls /var/tmp/.zfs/snapshot
```

```
after_cp                my_recursive_snapshot
```

```sh
ls /var/tmp/.zfs/snapshot/after_cp
```

```
passwd          vi.recover
```

```sh
cp /var/tmp/.zfs/snapshot/after_cp/passwd /var/tmp
```

尽管 `snapdir` 属性可能被设置为 `hidden`，但这并不影响 `ls .zfs/snapshot` 显示目录中的内容。管理员可能显示部分数据集的快照，同时不显示另一部分的。从隐藏的 `.zfs/snapshot` 复制文件和目录很简单。但反向操作会出现这样的错误：

```sh
cp /etc/rc.conf /var/tmp/.zfs/snapshot/after_cp/
```

```
cp: /var/tmp/.zfs/snapshot/after_cp/rc.conf: Read-only file system
```

该错误再次提醒用户快照是只读的，无法在创建后更改。而向其中复制文件或从中删除目录，都会导致其对照的数据集状态发生变化。

快照所占空间视其创建后父文件系统发生的变化而定。快照的 `written` 属性记录了其使用了多少空间。

`zfs destroy dataset@snapshot` 可以销毁快照并回收空间。`-r` 可以递归删除父数据集中所有同名的快照。`-n -v` 可以模拟出将删除多少快照并因此回收多少空间。

## 19.4.6 克隆管理

克隆是快照的一种副本，但可以当作常规数据集使用。与快照不同，克隆并非只读，可以挂载，也可以设置属性。通过 `zfs clone` 创建克隆后，源快照就不再可被销毁。克隆与源快照之间的父子关系可以通过 `zfs promote` 调换。将克隆提升，快照就从原本的父级变为子级。这回改变空间的计算方式，但并不影响空间的使用情况。克隆可以挂载到 ZFS 文件系统层级中的任何位置，而不仅仅是快照的原始位置。

使用下面的数据集来演示克隆功能：

```sh
zfs list -rt all camino/home/joe
```

```
NAME                    USED  AVAIL  REFER  MOUNTPOINT
camino/home/joe         108K   1.3G    87K  /usr/home/joe
camino/home/joe@plans    21K      -  85.5K  -
camino/home/joe@backup    0K      -    87K  -
```

克隆的典型用途是在保留快照以备恢复的同时，对特定数据集进行试验。从无法更改的快照中创建可读写的克隆，在获得预期结果后，可以将克隆提升为数据集，并删除旧文件系统。如果不这么做也不要紧，克隆可以和数据集并存。

```sh
zfs clone camino/home/joe@backup camino/home/joenew
ls /usr/home/joe*
```

```
/usr/home/joe:
backup.txz     plans.txt

/usr/home/joenew:
backup.txz     plans.txt
```

```sh
df -h /usr/home
```

```
Filesystem          Size    Used   Avail Capacity  Mounted on
usr/home/joe        1.3G     31k    1.3G     0%    /usr/home/joe
usr/home/joenew     1.3G     31k    1.3G     0%    /usr/home/joenew
```

被创建的副本是数据集在创建快照时状态的精准副本。但可以随意更改而不受源数据集的影响。两者之间唯一的联系就是源快照。ZFS 使用 `origin` 属性记录这层联系。当 `zfs promote` 提升克隆时，其与源快照的依赖关系就消失了，其 `origin` 设置也被删除，变成独立数据集。下面的例子示范了这一过程：

```sh
zfs get origin camino/home/joenew
```

```
NAME                  PROPERTY  VALUE                     SOURCE
camino/home/joenew    origin    camino/home/joe@backup    -
```

```sh
zfs promote camino/home/joenew
zfs get origin camino/home/joenew
```

```
NAME                  PROPERTY  VALUE   SOURCE
camino/home/joenew    origin    -       -
```

假设再在提升过的克隆中做一些变更，如复制写入 `loader.conf`，源目录相对而言就过时了。可以使用该克隆来替换它。 `zfs destroy` 销毁旧数据集后再将克隆 `zfs rename` 改名为旧数据集一样的名字（也可以用别的完全不同的名字）。

```sh
cp /boot/defaults/loader.conf /usr/home/joenew
zfs destroy -f camino/home/joe
zfs rename camino/home/joenew camino/home/joe
ls /usr/home/joe
```

```
backup.txz     loader.conf     plans.txt
```

```sh
df -h /usr/home
```

```
Filesystem          Size    Used   Avail Capacity  Mounted on
usr/home/joe        1.3G    128k    1.3G     0%    /usr/home/joe
```

克隆现在和普通数据集毫无二致。既包含了源快照中的全部数据，还有像 `loader.conf` 这样新增的部分。ZFS 使用者可以在不同场合使用克隆来达到不同目的。例如，监狱就是安装了一些配套应用程序的快照。用户可以克隆并添加自己所需的应用程序。添加完毕后再将克隆提升为真正的数据集已正常使用。通过这些监狱可以节约不少时间和管理成本。

## 19.4.7 同步

只将数据存储在单个池中，有很高的盗窃或别的天灾人祸的风险。定期备份整个池至关重要。ZFS 内置的序列化功能可以将数据以流的形式发送到标准输出中。因此数据不仅可以通过本地系统存储到另一个池中，也能通过网络发送到别的系统中。快照是实现同步的基础（详见 [19.4.5 管理快照](#1945-管理快照)）。同步数据所用指令是 `zfs send` 和 `zfs receive`。

下例演示了两个池之间的 ZFS 同步：

```sh
zpool list
```

```
NAME    SIZE  ALLOC   FREE   CKPOINT  EXPANDSZ   FRAG   CAP  DEDUP  HEALTH  ALTROOT
backup  960M    77K   896M         -         -     0%    0%  1.00x  ONLINE  -
mypool  984M  43.7M   940M         -         -     0%    4%  1.00x  ONLINE  -
```

`mypool` 池是会定期读写数据的主池。`backup` 池则是前者不可用时的备用池。请注意，此故障转移操作需要由管理员手动完成，ZFS 无法自动处理。快照用于提供待同步的文件系统的一致性版本。每当 `mypool` 的快照被创建时，就会复制到 `backup` 中。可以同步复制的只有快照。因此最新的快照之后的变更是无法被同步的。

```sh
zfs snapshot mypool@backup1
zfs list -t snapshot
```

```
NAME                    USED  AVAIL  REFER  MOUNTPOINT
mypool@backup1             0      -  43.6M  -
```

接下来就可以使用 `zfs send` 创建快照的内容流了。流可以存储成单个文件，也可以被另一个池接收。虽然流会被写入标准输出，但必须使用转向符或者管道进行处理，否则就会报错：

```sh
zfs send mypool@backup1
```

```
Error: Stream can not be written to a terminal.
You must redirect standard output.
```

再将其转向到挂载的备份池中的文件。要确保池中有足够的可用空间来容纳发送的快照的全部数据，而非仅仅是与更前一个快照之间的变更差。

```sh
zfs send mypool@backup1 > /backup/backup1
zpool list
```

```
NAME    SIZE  ALLOC   FREE   CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
backup  960M  63.7M   896M         -         -     0%     6%  1.00x  ONLINE  -
mypool  984M  43.7M   940M         -         -     0%     4%  1.00x  ONLINE  -
```

`zfs send` 将 `backup1` 快照中的全部数据传输到了 `backup` 池中。可以使用 [cron(8)][] 的任务来实现创建和发送快照的自动化操作。

除了将备份保存为文件，ZFS 还可以使用实时文件系统接收，使备份数据可以直接访问。`zfs receive` 用于将流中的数据再回流到文件和目录中。下面的例子通过管道组合使用了 `zfs send` 和 `zfs receive` 来将数据从一个池复制到另一个中。传输结束即可直接使用接收池中的数据。数据集只能同步到另一空数据集中。

```sh
zfs snapshot mypool@replica1
zfs send -v mypool@replica1 | zfs receive backup/mypool
```

```
send from @ to mypool@replica1 estimated size is 50.1M
total estimated size is 50.1M
TIME        SENT   SNAPSHOT
```

```sh
zpool list
```

```
NAME    SIZE  ALLOC   FREE   CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
backup  960M  63.7M   896M         -         -     0%     6%  1.00x  ONLINE  -
mypool  984M  43.7M   940M         -         -     0%     4%  1.00x  ONLINE  -
```

### 19.4.7.1 增量备份

`zfs send` 也可以检查两个快照之间的差别并只发送这些差异之处。以节约磁盘空间和传输时间。例如：

```sh
zfs snapshot mypool@replica2
zfs list -t snapshot
```

```
NAME                    USED  AVAIL  REFER  MOUNTPOINT
mypool@replica1         5.72M      -  43.6M  -
mypool@replica2             0      -  44.1M  -
```

```sh
zpool list
```

```
NAME    SIZE  ALLOC   FREE   CKPOINT  EXPANDSZ   FRAG   CAP  DEDUP  HEALTH  ALTROOT
backup  960M  61.7M   898M         -         -     0%    6%  1.00x  ONLINE  -
mypool  960M  50.2M   910M         -         -     0%    5%  1.00x  ONLINE  -
```

然后创建第二个快照 `replica2`。其中只包含了上个快照 `replica1` 之后的文件系统的变更。使用 `zfs send -i` 指定成对的快照就会生成一个只包含了变更数据的增量同步流。但只有接收方也有初始快照，才能执行成功。

```sh
zfs send -v -i mypool@replica1 mypool@replica2 | zfs receive /backup/mypool
```

```
send from @replica1 to mypool@replica2 estimated size is 5.02M
total estimated size is 5.02M
TIME        SENT   SNAPSHOT
```

```sh
zpool list
```

```
NAME    SIZE  ALLOC   FREE   CKPOINT  EXPANDSZ   FRAG  CAP  DEDUP  HEALTH  ALTROOT
backup  960M  80.8M   879M         -         -     0%   8%  1.00x  ONLINE  -
mypool  960M  50.2M   910M         -         -     0%   5%  1.00x  ONLINE  -
```

```sh
zfs list
```

```
NAME                         USED  AVAIL  REFER  MOUNTPOINT
backup                      55.4M   240G   152K  /backup
backup/mypool               55.3M   240G  55.2M  /backup/mypool
mypool                      55.6M  11.6G  55.0M  /mypool
```

```sh
zfs list -t snapshot
```

```
NAME                                         USED  AVAIL  REFER  MOUNTPOINT
backup/mypool@replica1                       104K      -  50.2M  -
backup/mypool@replica2                          0      -  55.2M  -
mypool@replica1                             29.9K      -  50.0M  -
mypool@replica2                                 0      -  55.0M  -
```

增量流传输成功。只有变更的数据被同步了，而不是完整的 `replica1`。因为只发送了差异之处，所用时间减少了，磁盘空间也节约了。在网络很慢或传输成本昂贵时，这很有用。

新的文件系统 `backup/mypool` 包含了 `mypool` 池中的全部文件和数据。如果使用 `-P` 选项，那么数据集属性也会被复制，包括压缩设置、配额和挂载点信息。而使用 `-R` 就会连子数据集和其属性设置一起复制。通过将发送和接收自动化处理，在第二个池中定期备份是很容易的。

### 19.4.7.2 通过 SSH 发送加密备份

通过网络发送流可以实现远程备份，但这并不完美。通过网络链接发送出的数据是未加密的，其他人可以在用户不知情的情况瞎拦截流并转化回数据。这让人无法容忍，尤其是通过互联网的情况。SSH 自身能很安全地加密通过网络链接发出的数据。而 ZFS 可以将流从标准数据再做重定向，很容易就使用管道传递到 SSH 中。如果希望文件系统的内容无论在传输过程中、亦或在远端系统中都是加密的，请考虑使用 [PEFS](https://wiki.freebsd.org/PEFS)。

开始之前请先完成一些设置和安全预防措施。这里只展示了 `zfs send` 操作所必需的步骤，其它内容请参阅 [13.8 OpenSSH](https://www.freebsd.org/doc/handbook/openssh.html)。

需要配置：

-   发送和接收主机之间使用 SSH 密钥完成无密码化的 SSH 访问；

-   一般情况下是需要 `root` 用户的权限来发送和接收流。即作为 `root` 登录接收方系统。但默认情况下出于安全考量是禁止使用 `root` 登录的。可以使用 ZFS [委派][]系统让两边系统中 `root` 之外的用户执行各自的发送和接收操作。

-   在发送方系统中：

    ```sh
    zfs allow -u someuser send,snapshot mypool
    ```

-   为了挂载池，非特权用户必须是目录所有者，且普通用户可以挂载文件系统。在接受方系统中：

    ```sh
    sysctl vfs.usermount=1
    ```

    ```
    vfs.usermount: 0 -> 1
    ```

    ```sh
    sysrc -f /etc/sysctl.conf vfs.usermount=1
    zfs create recvpool/backup
    zfs allow -u someuser create,mount,receive recvpool/backup
    chown someuser /recvpool/backup
    ```

至此非特权用户可以接收和挂载数据集，`home` 数据集也就可以同步到远端系统：

```sh
zfs snapshot -r mypool/home@monday
zfs send -R mypool/home@monday | ssh someuser@backuphost zfs recv -dvu recvpool/backup
```

将 `mypool` 池中的 `home` 数据集的文件系统生成递归快照 `monday`。再通过 `zfs send -R` 将数据集、全部子数据集、快照、克隆和设置都以流的形式发送。输出结果通过管道传导到 SSH 的远端主机 `backuphost` 中等候的 `zfs receive` 指令中。主机名称推荐使用全量域名或者 IP 地址。接收方机器将数据写入 `recvpool` 池的 `backup` 数据集中。在 `zfs recv` 时配合以 `-d` 可以在接收方用快照名称覆盖池的名称。`-u` 可以不在接收方挂载文件系统。`-v` 可以显示更多的传输信息，包括耗时和数据传输量。

## 19.4.8 数据集、用户和组的配额

[数据集配额][]用于限制单个数据集可消耗的空间总量。[引用配额][]工作方式与之类似，但只统计数据集自身的空间占用，不包括快照和子集。同样的，[用户配额][]和[组配额][]用于防止用户或组使用池或数据集的太多空间。

给 `storage/home/bob` 分配 10GB 的数据集配额：

```sh
zfs set quota=10G storage/home/bob
```

给 `storage/home/bob` 分配 10GB 的引用配额：

```sh
zfs set refquota=10G storage/home/bob
```

去重 `storage/home/bob` 的配额限制：

```sh
zfs set quota=none storage/home/bob
```

用户配额通用格式为 `userquote@user=size`，且用户名必须符合以下格式之一：

-   POSIX 兼容名称，如 `joe`

-   POSIX 数字编号，如 `789`

-   SID 名称，如 `joe.bloggs@example.com`

-   SID 数字编号，如 `S-1-123-456-789`

例如，给用户 `joe` 分配 50GB 的用户配额：

```sh
zfs set userquota@joe=50G
```

去重配额限制：

```sh
zfs set userquota@joe=none
```

> **注意：**
>
> `zfs get all` 无法显示用户配额属性。非 `root` 用户除非被授予 `userquota` 权限，否则只能看到自己的配额。具有这个权限的用户可以看到和设置每个用户的配额。

组配额的通用格式为 `groupquota@group=size`。

给 `firstgroup` 组分配 50GB 配额：

```sh
zfs set groupquota@firstgroup=50G
```

去重 `firstgroup` 的配额：

```sh
zfs set groupquota@firstgroup=none
```

与用户配额属性类似，非 `root` 用户只能看到其所在的各个群的配额。而 `root` 或具有 `groupquota` 权限的用户可以看到和设置全部组的配额。

使用 `zfs userspace` 可以查看每个用户在有配额限制的文件系统或快照中所占用的空间大小。要查看组信息，可以使用 `zfs groupspace`。关于可用选项和如何只显示特定内容的更多信息，请查看 [zfs(1)][] 帮助文档。

具有相应权限的用户和 `root` 可以使用下面的指令查看 `storage/home/bob` 相关的配额：

```sh
zfs get quota storage/home/bob
```

## 19.4.9 预留

[预留][]可以确保数据集中始终有最小量的可用空间。这些预留空间无法用于其它数据集。这个特性可以确保某个重要的数据集或日志文件能够有充足的可用空间。

`reservation` 属性的通用格式是 `reservation=size`，给 `storage/home/bob` 预留 10GB：

```sh
zfs set reservation=10G storage/home/bob
```

清除预留：

```sh
zfs set reservation=none storage/home/bob
```

基于同样的原理，也可以通过 `refreservation` 属性来设置[引用预留][]。通用格式为 `refreservation=size`。

查看 `storage/home/bob` 的预留和引用预留：

```sh
zfs get reservation storage/home/bob
zfs get refreservation storage/home/bob
```

## 19.4.10 压缩

ZFS 支持透明压缩。块级别的数据压缩除了节省空间，还可以提高磁盘吞吐量。当数据压缩比为 25% 时，以同样的速度写入压缩数据和非压缩数据，意味着 125% 的写入速度的提升。压缩也可以替代[去重](#19411-去重)，因为它不需要额外的内存。

ZFS 支持若干种不同的压缩算法，每种的开销各有不同。在 ZFS v5000 引入的 LZ4 压缩算法，可以比其它算法更低的性能开销对整个池启用压缩。LZ4 最大的优势在于其 _早期中止_ 特性。当 LZ4 发现数据的起始部分无法达到 12.5% 以上的压缩比时，就会认为数据要么已经被压缩过、要么无法被压缩，直接写入以免无用计算。关于 ZFS 中各压缩算法的更多详情，请参见术语部分中的[压缩][]条目。

管理员可以通过数据集属性的组合来监控压缩效率：

```sh
zfs get used,compressratio,compression,logicalused mypool/compressed_dataset
```

```
NAME        PROPERTY          VALUE     SOURCE
mypool/compressed_dataset  used              449G      -
mypool/compressed_dataset  compressratio     1.11x     -
mypool/compressed_dataset  compression       lz4       local
mypool/compressed_dataset  logicalused       496G      -
```

数据集目前使用了 449GB 空间（`used` 属性）。如不压缩应该会占用 496GB 空间（`logicalused` 属性）。压缩比为 1.11:1。

压缩与[用户配额][]组合工作时会产生一个意想不到的效果。因为用户配额是以 _压缩后_ 的数据大小数值来约束其对数据集空间的使用。因此一个拥有 10GB 配额的用户，在写入 10GB 可压缩的数据之后，仍然可以保存其它数据。但假如他再更新了某些文件，如数据库，造成压缩结果变化，其可用的剩余空间的大小就也会随之变化。极端情况下，用户虽然并没有新增数据（`logicalused` 属性），但压缩的变化致使达到了配额限制。

压缩还可能与备份发生类似的意外的相互干扰。配额常用于限制可以存储多少数据，来保障足够的可用备份空间。 但由于配额不考虑压缩，因此与未压缩的备份相比，写入的数据可能更多。

## 19.4.11 去重

在启用时，[去重][]机制使用每个块的效验值来寻找重复块。当新快与某个既有块重复时，ZFS 只会写入指向该既有块的引用信息。如果数据中囊括了许多雷同文件或重复信息，则可以节约大量空间。警告：去重需要极大的内存，启用压缩即可节约大部分的空间且损耗更小。

在目标池上激活去重机制需要设置 `dedup` 属性：

```sh
zfs set dedup=on pool
```

只有新写入池中的数据才会被去重。原有数据即便激活机制也不会被处理。新启用去重机制的池可能看起来是这个样子：

```sh
zpool list
```

```
NAME  SIZE ALLOC  FREE   CKPOINT  EXPANDSZ   FRAG   CAP   DEDUP   HEALTH   ALTROOT
pool 2.84G 2.19M 2.83G         -         -     0%    0%   1.00x   ONLINE   -
```

`DEDUP` 列展示了池的真实去重比例。`1.00x` 的值说明数据中没有重复。下例中，端口树会分三次复制到去重池的不同目录中。

```sh
for d in dir1 dir2 dir3; do
    mkdir $d && cp -R /usr/ports $d &
done
```

将冗余数据去重后：

```sh
zpool list
```

```
NAME SIZE  ALLOC  FREE   CKPOINT  EXPANDSZ   FRAG  CAP   DEDUP   HEALTH   ALTROOT
pool 2.84G 20.9M 2.82G         -         -     0%   0%   3.00x   ONLINE   -
```

`DEDUP` 列显示的因子值变成了 `3.00x`。端口树的不同副本被检测并去重，因此只用掉了三分之一的空间。这种节约空间的潜力是巨大的，但代价就是需要足够大的内存来保持追踪重复块。

去重对于没有冗余数据的池来说毫无益处。ZFS 可以在现有池中模拟展示去重机制的潜在的空间节省效果：

```sh
zdb -S pool
```

```
Simulated DDT histogram:

bucket              allocated                       referenced
______   ______________________________   ______________________________
refcnt   blocks   LSIZE   PSIZE   DSIZE   blocks   LSIZE   PSIZE   DSIZE
------   ------   -----   -----   -----   ------   -----   -----   -----
     1    2.58M    289G    264G    264G    2.58M    289G    264G    264G
     2     206K   12.6G   10.4G   10.4G     430K   26.4G   21.6G   21.6G
     4    37.6K    692M    276M    276M     170K   3.04G   1.26G   1.26G
     8    2.18K   45.2M   19.4M   19.4M    20.0K    425M    176M    176M
    16      174   2.83M   1.20M   1.20M    3.33K   48.4M   20.4M   20.4M
    32       40   2.17M    222K    222K    1.70K   97.2M   9.91M   9.91M
    64        9     56K   10.5K   10.5K      865   4.96M    948K    948K
   128        2   9.50K      2K      2K      419   2.11M    438K    438K
   256        5   61.5K     12K     12K    1.90K   23.0M   4.47M   4.47M
    1K        2      1K      1K      1K    2.98K   1.49M   1.49M   1.49M
 Total    2.82M    303G    275G    275G    3.20M    319G    287G    287G

dedup = 1.05, compress = 1.11, copies = 1.00, dedup * compress / copies = 1.16
```

`zdb -S` 对池的分析结果展示了在启用去重机制时的空间节约比例。例中的 `1.16` 是一个很差的空间节约比例，而且还主要通过压缩实现。在该池上激活去重并不会有显著的空间节约，甚至还不及所需的内存开销。通过 `ratio = dedup * compress / copies` 公式，系统管理员可以规划存储分配，确定工况中的重复块是否抵得上内存需求。如果数据可以合理压缩，空间节约情况就会很好。这种情况下推荐首先启用压缩，而且压缩还能大大提高性能。只有额外的节省效果也值得衡量、且可用于去重的内存又足够多时，才应该启用去重。

## 19.4.12 ZFS 和监狱

`zfs jail` 和对应的 `jailed` 属性用于将 ZFS 数据集委派成[监狱](https://www.freebsd.org/doc/handbook/jails.html)。`zfs jail jailed` 会关联数据集到指定监狱中，而 `zfs unjail` 则会反向分离。要在监狱中控制某个数据集，则必须设置数据集的 `jailed` 属性。关联到监狱的数据集无法在挂载到宿主机上，因其挂载点可能会损害宿主机的安全性。
