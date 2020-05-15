---
title: "【译】第 19 章：Z 文件系统㈡"
slug: "第19章：Z文件系统二"
date: 2020-05-11T22:57:35+08:00
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

[点击这里阅读原文。](https://www.freebsd.org/doc/handbook/zfs-quickstart.html)

---

## 19.2 快速体验指南

有一种启动机制可以让 FreeBSD 在系统初始化阶段加载 ZFS 池。将下行添加至 `/etc/rc.conf`：

<!--more-->

[zpool(8)]: https://www.freebsd.org/cgi/man.cgi?query=zpool&sektion=8&manpath=freebsd-release-ports
[zfs(8)]: https://www.freebsd.org/cgi/man.cgi?query=zfs&sektion=8&manpath=freebsd-release-ports
[periodic(8)]: https://www.freebsd.org/cgi/man.cgi?query=periodic&sektion=8&manpath=freebsd-release-ports
[联网]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-online
[离线]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-offline

```ini
zfs_enable="YES"
```

然后启动服务，可以激活这一机制。

```sh
service zfs start
```

## 19.2.1 单磁盘池

使用单磁盘设备可以创建一个简单的非冗余池：

```sh
zpool create example /dev/da0
```

通过 `df` 的输出我们可以看到这个新池：

```sh
df
```

```
Filesystem  1K-blocks    Used    Avail Capacity  Mounted on
/dev/ad0s1a   2026030  235230  1628718    13%    /
devfs               1       1        0   100%    /dev
/dev/ad0s1d  54098308 1032846 48737598     2%    /usr
example      17547136       0 17547136     0%    /example
```

`example` 这个池就已经被创建并挂在了。现在可以以文件系统的方式进行操作。如创建并查看文件：

```sh
cd /example
ls
touch testfile
ls -al
```

```
total 4
drwxr-xr-x   2 root  wheel    3 Aug 29 23:15 .
drwxr-xr-x  21 root  wheel  512 Aug 29 23:12 ..
-rw-r--r--   1 root  wheel    0 Aug 29 23:15 testfile
```

此时池还未开启任何 ZFS 的特性。在池中创建一个数据集并开启压缩：

```sh
zfs create example/compressed
zfs set compression=gzip example/compressed
```

`example/compressed` 数据集现在就是一个 ZFS 压缩文件系统了。可以试着拷贝一些大文件进去看看效果。

要关闭压缩可以：

```sh
zfs set compression=off example/compressed
```

使用 `zfs umount` 可以卸载一个文件系统，然后通过 `df` 来确认操作：

```sh
zfs umount example/compressed
df
```

```
Filesystem  1K-blocks    Used    Avail Capacity  Mounted on
/dev/ad0s1a   2026030  235232  1628716    13%    /
devfs               1       1        0   100%    /dev
/dev/ad0s1d  54098308 1032864 48737580     2%    /usr
example      17547008       0 17547008     0%    /example
```

而再次使用 `zfs mount` 可以重新再挂载回来：

```sh
zfs mount example/compressed
df
```

```
Filesystem         1K-blocks    Used    Avail Capacity  Mounted on
/dev/ad0s1a          2026030  235234  1628714    13%    /
devfs                      1       1        0   100%    /dev
/dev/ad0s1d         54098308 1032864 48737580     2%    /usr
example             17547008       0 17547008     0%    /example
example/compressed  17547008       0 17547008     0%    /example/compressed
```

`mount` 指令的输出中也包含了池和文件系统的信息：

```sh
mount
```

```
/dev/ad0s1a on / (ufs, local)
devfs on /dev (devfs, local)
/dev/ad0s1d on /usr (ufs, local, soft-updates)
example on /example (zfs, local)
example/compressed on /example/compressed (zfs, local)
```

ZFS 数据集可以在创建后以任意文件系统的方式进行操作。但还有很多其它特性是针对每个数据集设置的。如下面的例子，`data` 文件系统需要存储重要文件，因此被配置为每个数据块同时保持两份副本：

```sh
zfs create example/data
zfs set copies=2 example/data
df
```

```
Filesystem         1K-blocks    Used    Avail Capacity  Mounted on
/dev/ad0s1a          2026030  235234  1628714    13%    /
devfs                      1       1        0   100%    /dev
/dev/ad0s1d         54098308 1032864 48737580     2%    /usr
example             17547008       0 17547008     0%    /example
example/compressed  17547008       0 17547008     0%    /example/compressed
example/data        17547008       0 17547008     0%    /example/data
```

请注意，一个池中的每个文件系统都有相同的可用空间。 这也是使用 `df` 演示的原因，以表明文件系统仅使用其所需的空间大小，且所有空间均来源于同一个池。 ZFS 消除了诸如卷和分区之类的概念，并允许多个文件系统占用同一个池。

最后是销毁文件系统和不再需要的池：

```sh
zfs destroy example/compressed
zfs destroy example/data
zpool destroy example
```

## 19.2.2 RAID-Z

磁盘会出错。避免磁盘故障造成数据丢失的一种方法是 RAID。 ZFS 在池设计中也支持此功能。 RAID-Z 池需要三块以上的磁盘，但可以比镜像池提供更多的可用空间。

指定要添加的磁盘来创建 RAID-Z 池：

```sh
zpool create storage raidz da0 da1 da2
```

> **注意：**
>
> Sun™ 推荐一个 RAID-Z 配置中使用三到九块磁盘。如果需要十块以上的磁盘来创建一个池，请考量将其拆分成多个小池。如果只有两块硬盘但又需要冗余，请考量使用 ZFS 镜像。更多细节请参阅 [zpool(8)][] 帮助文档。

这样就创建了 `storage` 池，接下来是在池中创建 `home` 文件系统：

```sh
zfs create storage/home
```

也开启压缩和多副本：

```sh
zfs set copies=2 storage/home
zfs set compression=gzip storage/home
```

将用户数据复制进来并符号链接回去，这样这个目录就成了新的用户目录：

```sh
cp -rp /home/* /storage/home
rm -rf /home /usr/home
ln -s /storage/home /home
ln -s /storage/home /usr/home
```

可以尝试创建一个新用户，其数据应该也是存储在 `/storage/home` 中。

创建一个可以回滚的文件系统快照：

```sh
zfs snapshot storage/home@08-30-08
```

只有文件系统才能创建快照，目录和文件是做不到的。

`@` 字符之后是快照的名称。如果不小心删掉了某个重要文件夹，也可以在备份文件系统后将其回滚至删除前的快照：

```sh
zfs rollback storage/home@08-30-08
```

使用 `ls` 查看文件系统的 `.zfs/snapshot` 目录，可以列举全部可用的快照：

```sh
ls /storage/home/.zfs/snapshot
```

通过编写脚本可以自动定期给用户数据生成快照。但久而久之，大量的磁盘空间都会被快照占用。这样就可以删除一些快照：

```sh
zfs destroy storage/home@08-30-08
```

完成测试后，可以直接将 `/storage/home` 挂载至 `/home`：

```sh
zfs set mountpoint=/home storage/home
```

这时系统会将其视作真正的 `/home`：

```sh
mount
```

```
/dev/ad0s1a on / (ufs, local)
devfs on /dev (devfs, local)
/dev/ad0s1d on /usr (ufs, local, soft-updates)
storage on /storage (zfs, local)
storage/home on /home (zfs, local)
```

```sh
df
```

```
Filesystem   1K-blocks    Used    Avail Capacity  Mounted on
/dev/ad0s1a    2026030  235240  1628708    13%    /
devfs                1       1        0   100%    /dev
/dev/ad0s1d   54098308 1032826 48737618     2%    /usr
storage       26320512       0 26320512     0%    /storage
storage/home  26320512       0 26320512     0%    /home
```

这样就完成了 RAID-Z 的配置。将下面的配置添加到 `/etc/periodic.conf`，可以在每晚自动生成的 [periodic(8)][] 报告中加上 ZFS 文件系统的每日状态更新的信息：

```ini
daily_status_zfs_enable="YES"
```

## 19.2.3 恢复 RAID-Z

每种软件级 RAID 都有各自的方法来监控中泰。RAID-Z 设备的状态可以这样查看：

```sh
zpool status -x
```

如果所有的池都是[联网][]状态，也没有任何问题，就应该输出：

```
all pools are healthy
```

如果某个磁盘成了[离线][]状态，那么池的状态信息应该看起来是这个样子：

```
  pool: storage
 state: DEGRADED
status: One or more devices has been taken offline by the administrator.
	Sufficient replicas exist for the pool to continue functioning in a
	degraded state.
action: Online the device using 'zpool online' or replace the device with
	'zpool replace'.
 scrub: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	storage     DEGRADED     0     0     0
	  raidz1    DEGRADED     0     0     0
	    da0     ONLINE       0     0     0
	    da1     OFFLINE      0     0     0
	    da2     ONLINE       0     0     0

errors: No known data errors
```

原来是管理员之前通过这个指令将设备离线了：

```sh
zpool offline storage da1
```

以在系统关机后替换掉 `da1`。然后等系统重新开机后，就可以在池中替换掉故障的磁盘：

```sh
zpool replace storage da1
```

再次检查状态，不带 `-x` 参数时会呈现全部池的状态：

```sh
zpool status storage
```

```
 pool: storage
 state: ONLINE
 scrub: resilver completed with 0 errors on Sat Aug 30 19:44:11 2008
config:

	NAME        STATE     READ WRITE CKSUM
	storage     ONLINE       0     0     0
	  raidz1    ONLINE       0     0     0
	    da0     ONLINE       0     0     0
	    da1     ONLINE       0     0     0
	    da2     ONLINE       0     0     0

errors: No known data errors
```

恢复正常工作了。

## 19.2.4 数据效验

ZFS 使用效验值来验证存储数据的完整性。这个机制在创建文件系统时便会自动启用。

> **警告：**
>
> 虽然可以禁用效验值，但我们非常不推荐这种行为！效验值只需要极少的存储空间便足以提供数据的完整性。而且很多其它的 ZFS 特性也依赖于效验值机制。禁用效验值并不会显著地提高性能。

效验值验证的行为被成为 _清理_。使用以下指令可以验证 `storage` 池的数据完整性：

```sh
zpool scrub storage
```

清理的耗时视数据存储量而定。数据量大时会按比例花费更长的时间进行验证。清理非常消耗 I/O，一次只能运行一次。清理完成后，池状态应该是这个样子：

```sh
zpool status storage
```

```
 pool: storage
 state: ONLINE
 scrub: scrub completed with 0 errors on Sat Jan 26 19:57:37 2013
config:

	NAME        STATE     READ WRITE CKSUM
	storage     ONLINE       0     0     0
	  raidz1    ONLINE       0     0     0
	    da0     ONLINE       0     0     0
	    da1     ONLINE       0     0     0
	    da2     ONLINE       0     0     0

errors: No known data errors
```

可以根据上次清理操作的完成时间来判断是否需要再次清理。例行清理有助于保护数据遭受无提示性破坏，保障池的完整性。

其它 ZFS 选项请参阅 [zfs(8)][] 和 [zpool(8)][] 的参考文档。
