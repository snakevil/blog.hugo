---
title: "【译】ZFS RAID-Z 的性能、容量和完整性"
slug: "ZFS-RAIDZ的性能、容量和完整性"
date: 2020-05-17T20:18:04+08:00
toc: true
category:
    - Linux
    - 博闻广记
    - 系统架构
tag:
    - ZFS
    - FreeBSD
    - Linux
    - RAID
    - 自主翻译
---

[点击这里阅读原文。](https://calomel.org/zfs_raid_speed_capacity.html)

---

比较每种 RAID-Z 类型的速度、空间和安全性。

ZFS 包含数据完整性校验、防止数据损坏、支持高存储容量、出色的性能、同步、快照与写时复制（Copy-On-Write，COW）克隆，和自愈功能，这些功能都使其成为数据存储时的自然选择。

<!--more-->

在讨论 RAID 时，最常被问到的问题是“哪种 RAID 是最好的？”这其实取决于您要实现的目标和您愿意放弃的东西。我们应该问自己的问题是：“我们的数据究竟有多重要？”

无论如何您最终都会失去一块驱动器，此时 RAID 方案就决定了其中的数据是否会丢失。如果 RAID 只损坏了单块硬盘却导致您的所有数据都丢失了，那将是一场灾难。您可能需要看看更安全的 RAID 配置，以保障 RAID 可以损坏两块以上的驱动器。而更高的数据安全性会导致的问题，便是您可能会放弃速度或者容量，甚至两者都放弃。

RAID 有三大主要好处：性能、容量和完整性。性能是 RAID 的读写速度有多快，以兆字节每秒和延迟毫秒数为度量单位。容量是 RAID 能容纳多少数据。完整性是多少块磁盘出现故障后才会丢失全部数据。问题在于您可能无法同时取得全部的三项好处。

下面的 ASCII 三角形展示了全部的三项属性。当您将鼠标光标放在三角形的中心，然后尝试移向一个属性，您会发现其它的属性正在远离。例如，RAID0 既快也有着最高的容量，但完全没有数据完整性。另一方面，RAID1 有着出色的完整性和快速读取能力，但写入很慢（多份）且容量受限。

```
                容量
                 /\
                /  \
               /    \
              /      \
        性能 /________\ 完整性
```

## 每种 RAID 类型的优劣何在？

-   **RAID0 或条带阵列**没有冗余，但有着最好的性能和附加存储。任何设备故障都将摧毁整个阵列，因此 RAID0 根本不安全。如果您需要非常快的暂存空间来进行视频编辑，RAID0 是个好选择。
-   **RAID1 或镜像**将同样的数据在阵列的每块驱动器上都做简单镜像。极好的冗余让您可以丢失任意多的驱动器，只要留有一块，数据就依然可访问。好处是这种 RAID 的读取速度受到阵列中每块磁盘的加成。但大坏处则是容量很低，写入速度也很慢。无论阵列中有多少块磁盘最终都只能使用单块的容量。性能受损则是因为每个驱动器都需要将相同的文件再完整存一份。镜像一般用于两块驱动器，而非其它测试中的 12 块或 24 块，因为这纯粹在浪费空间。
-   **RAID2、RAID3 和 RAID4** 因不再被 IT 工业事业就不做测试。RAID2 使用同样数量的磁盘作为专用的 ECC 驱动器。RAID3 和 RAID4 单独使用一块驱动器做奇偶校验。因为可怕的随机读写的性能影响，这些 RAID 都不再用于生产。
-   **RAID5 或 RAIDZ** 将奇偶校验与数据一起存储，因此即便丢失一块物理驱动器也不会造成 RAID 故障。因为需要计算奇偶校验，RAID5 要比 RAID0 慢，但安全得多。RAID5 至少需要三块硬盘，其中相当于一整块磁盘容量的空间会用于奇偶校验。
-   **RAID6 或 RAIDZ2** 将奇偶校验与数据一起存储，但可以丢失两块物理驱动器而非 RAID5 的一块。因为需要计算更多的奇偶校验，RAID6 比 RAID5 更慢，当然也更安全。RAIDZ2 需要至少四块磁盘，其中两块容量大小的空间用作奇偶校验。
-   **RAID7 或 RAIDZ3** 如 RAID5 和 RAID6 一般存储奇偶校验，但可以损失三块物理驱动器。因为要计算三倍的奇偶校验，RAID7 比 RAID5 或 RAID6 都要慢，但也是三种之中最安全的。RAIDZ3 需要至少四块磁盘，但建议使用不少于五块，因为其中三块容量的空间要用作奇偶校验。
-   **RAID10 或 RAID1+0** 是数据的镜像带区。最简单的 RAID10 阵列有四块磁盘并作为两组镜像处理。第一块和第二块是镜像，而第三块和第四块是另一个镜像。然后数据像 RAID0 一样跨镜像形成带区。您可以在每个镜像中都损失一块驱动器而不影响数据。但不能损失组成一个镜像的两块硬盘，比如不能第一块和第二块就不能同时损坏。RAID10 的优势在于读取数据很快。而劣势则是写入慢（多镜像）且容量低。
-   **RAID60 或 RAID6+0** 是两个以上 RAID6 卷的带区。您可以从中获取 RAID6 的安全性（每个 RAID6 阵列可损失两块驱动器）和 RAID0 带区读取速度。缺点和 RAID10 一样。
-   **RAID70 或 RAID7+0** 是两个以上 RAID7 卷的带区。和 RAID6 一样，您可以享有 RAID7 的安全性和 RAID0 的带区读取速度，但失去容量。

**安全第一，不留遗憾！** 在选择 RAID 配置时，您可能会觉得 RAID5 或 RAIDZ 的速度和容量都不错而决心选用。但真实世界的经验总结强烈建议我们不要使用 RAID5。只因为还不过安全。我们推荐的是 RAID1 镜像或者 RAID6（两份奇偶校验）甚至 RAID7（三份）。RAID5 的问题在于只有一个驱动器有奇偶校验。一旦某个驱动器挂了，RAID5 阵列就会降级，如果再出问题，整个阵列全挂，数据全丢！大多数情况下，挂了一个驱动器，更换，然后开始重构或重建。但在此过程中，别的磁盘也有几率（大约 8%）因为受到压力而损坏。更多信息请认真阅读这篇写于 2007 年的[《网络应用压垮磁盘》](http://storagemojo.com/2007/02/26/netapp-weighs-in-on-disks/)，驱动器技术并未改变很多，因此时至今日文章依然有效。

## 测试 RAID 的框架和环境指标

所有的测试于同一天运行在同一台机器上。我们的目标是消除硬件带来的变量，从而确定所有性能上的差异都是源自不同的 ZFS RAID 配置本身。

-   FreeBSD 10.2，更新至文章发表日
-   基于我们自己的[《FreeBSD 调校与优化》](https://calomel.org/freebsd_network_tuning.html)修改性能
-   CPU：Intel E5-2630 单槽六核
-   RAM：Kingston 16GB DDR3 1600
-   HBA：PCI-E 3.0 x16 槽上的 Avago Technologies (LSI) SAS2308 9207-8i
-   机箱：SuperMicro 4U, SuperChassis 846BE16-R1K28B
-   系统驱动器：Samsung 850 PRO 256GB
-   RAID 驱动器：Western Digital Black 4TB 7200rpm SAS (WD4001FYYG) 共 24 块
-   主板：SuperMicro X9SRE
-   SAS 扩展器：SuperMicro Back Plane, BPN-SAS2-846EL1
-   机房室温：主机高度 21C, 71F, 294K
-   机房适度：机柜前约 40%
-   机房风量：每小时约 240 立方米
-   服务器声压：机柜前一英尺 73dB
-   服务器机柜震动：机箱启动器背板小于 0.01m/s2
-   跑分工具：Bonnie++ v1.97

## Bonnie++ 跑分

Bonnie++ 是一个专注于硬盘和文件系统性能的多轮简单测试的跑分工具。跑分测试以数据库的方式操作单个文件，模拟创建、读取和删除很多小文件。

创建的 ZFS 池都禁用 LZ4 压缩，因此 Bonnie++ 的测试数据应该会直接写入磁盘。测试文件大小为 16GB，这样无法使用内存或者 ZFS ARC 进行缓存。Bonnie++ 使用四个并发线程以更好地模拟真实世界的服务器负载。使用脚本循环执行 Bonnie++ 三次然后再采用中位（中间）性能指标。每次运行后系统都会休眠 30 秒，以减轻负载。

Bonnie++ 可以实现异步 I/O，这意味着驱动器的 4K 本地磁盘缓存会被大量使用，每隔 30 秒 ZFS 提交就会导致清除一次。为了避免磁盘缓存对结果的干扰，我们选择只使用 Bonnie++ 的同步测试模式以完全禁用磁盘缓存。每次写入后的同步都会导致偏低的跑分值，但这些数字更接近于服务器负载很高、内存也被吃满的表现。

```sh
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

快速摘要：ZFS 的速度和容量

下表展示了每轮测试和所用磁盘数量、RAID 类型、容量和性能指标的摘要信息以方便简易比较。速度项“w”表示写入，“rw”表示改写，“r”表示读取，单位为每秒兆字节 MBps。

ZFS 使用最大可至 1024KB 的可变块大小。如果启用了数据压缩（LZJB 或 ZL4），就会使用可变块大小。当一个块在压缩后可以适应更小的块大小，就会在磁盘上使用较小的大小以节约存储并提高 I/O 吞吐量，代价是增加了用于压缩和解压操作的 CPU 用量。请在后文中查看我们对压缩的研究。

构建 RAID 的通常做法是使用“额外多两块奇偶校验”的 RAID 基础结构来最大化奇偶校验的带区、速度和容量。但使用 ZFS 时这些 RAID 标准规则并不适用，特别是启用 LZ4 压缩时。ZFS 可以改变每个磁盘上的带区大小，而压缩又会让带区变得不可预测。创建 ZFS RAID 的经验准则是：

-   镜像（RAID1）使用两到四块以上的磁盘
-   RAIDZ-1（RAID5）使用五块以上的磁盘
-   RAIDZ-2（RAID6）使用六块以上的磁盘
-   RAIDZ-3（RAID7）使用十一块以上的磁盘

## 机械硬盘 RAID（Spinning platter hard drive raids）

服务器使用一张 Avago LSI 的主机总线适配器（Host Bus Adapter，HBA）而非 RAID 卡搭建。使用一条多通道线缆将 HBA 连接到 SAS 扩展器，再控制所有 24 个驱动器。禁用了 LZ4 压缩，这样 Bonnie++ 的产生的全部写入都是实时同步的。我们想测试的是 RAID 配置而非驱动器本身，因此既要禁用 LZ4 压缩，也要避免内存大到使用 ZFS ARC。当启用了 LZ4 再去存储可压缩数据，同时内存也达到读写数据集的两倍时，速度可以轻松增加两倍。

| 驱动器数（Nx 4TB） | 配置                                                      | 容量（TB） | 写（MB/s） | 改写（MB/s） | 读（MB/s） |
| ------------------ | --------------------------------------------------------- | ---------- | ---------- | ------------ | ---------- |
| 1                  | [单盘](#单盘)                                             | 3.7        | 108        | 50           | 204        |
| 2                  | [镜像（RAID1）](#2-盘镜像raid1)                           | 3.7        | 106        | 50           | 488        |
| 2                  | [带区（RAID0）](#2-盘带区raid0)                           | 7.5        | 237        | 73           | 434        |
| 3                  | [镜像（RAID1）](#3-盘镜像raid1)                           | 3.7        | 106        | 49           | 589        |
| 3                  | [带区（RAID0）](#3-盘带区raid0)                           | 11.3       | 392        | 86           | 474        |
| 3                  | [RAIDZ-1（RAID5）](#3-盘-raidz-1raid5)                    | 7.5        | 225        | 53           | 644        |
| 4                  | [2 组镜像带区](#4-盘-2-组镜像带区)                        | 7.5        | 226        | 53           | 644        |
| 4                  | [RAIDZ-2（RAID6）](#4-盘-raidz-2raid6)                    | 7.5        | 204        | 54           | 183        |
| 5                  | [RAIDZ-1（RAID5）](#5-盘-raidz-1raid5)                    | 15.0       | 469        | 79           | 598        |
| 5                  | [RAIDZ-3（RAID7）](#5-盘-raidz-3raid7)                    | 7.5        | 116        | 45           | 493        |
| 6                  | [3 组镜像带区](#6-盘-3-组镜像带区)                        | 11.3       | 389        | 60           | 655        |
| 6                  | [RAIDZ-2（RAID6）](#6-盘-raidz-2raid6)                    | 15.0       | 429        | 71           | 488        |
| 10                 | [2 组 5 盘 RAIDZ-1 带区](#10-盘-2-组-5-盘-raidz-1-带区)   | 30.1       | 675        | 109          | 1012       |
| 11                 | [RAIDZ-3（RAID7）](#11-盘-raidz-3raid7)                   | 30.2       | 552        | 103          | 963        |
| 12                 | [6 组镜像带区](#12-盘-6-组镜像带区)                       | 22.6       | 643        | 83           | 962        |
| 12                 | [2 组 6 盘 RAIDZ-2 带区](#12-盘-2-组-6-盘-raidz-2-带区)   | 30.1       | 638        | 105          | 990        |
| 12                 | [RAIDZ-1（RAID5）](#12-盘-raidz-1raid5)                   | 41.3       | 689        | 118          | 993        |
| 12                 | [RAIDZ-2（RAID6）](#12-盘-raidz-2raid6)                   | 37.4       | 317        | 98           | 1065       |
| 12                 | [RAIDZ-3（RAID7）](#12-盘-raidz-3raid7)                   | 33.6       | 452        | 105          | 840        |
| 22                 | [2 组 11 盘 RAIDZ-3 带区](#22-盘-2-组-11-盘-raidz-3-带区) | 60.4       | 567        | 162          | 1139       |
| 23                 | [RAIDZ-3（RAID7）](#23-盘-raidz-3raid7)                   | 74.9       | 440        | 157          | 1146       |
| 24                 | [12 组镜像带区](#24-盘-12-组镜像带区)                     | 45.2       | 696        | 144          | 898        |
| 24                 | [RAIDZ-1（RAID5）](#24-盘-raidz-1raid5)                   | 86.4       | 567        | 198          | 1304       |
| 24                 | [RAIDZ-2（RAID6）](#24-盘-raidz-2raid6)                   | 82.0       | 434        | 189          | 1063       |
| 24                 | [RAIDZ-3（RAID7）](#24-盘-raidz-3raid7)                   | 78.1       | 405        | 180          | 1117       |
| 24                 | [带区（RAID0）](#24-盘带区raid0)                          | 90.4       | 692        | 260          | 1377       |

以下是 [zpool](/2020/05/第19章Z文件系统三/) 指令和 Bonnie++ 的原始输出。

### 单盘

```sh
zpool create storage da0
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           108834  25 50598  11           204522   9 393.0   4
Latency                        1992ms    2372ms              1200ms     289ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          da0       ONLINE       0     0     0
```

### 2 盘镜像（RAID1）

```sh
zpool create storage mirror da0 da1
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           106132  25 50303  10           488762  23 445.7   4
Latency                        9435ms    4173ms               220ms     195ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
```

### 2 盘带区（RAID0）

```sh
zpool create storage da0 da1
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           237933  43 73012  14           434041  20 513.1   5
Latency                         505ms    4059ms               212ms     197ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          da0       ONLINE       0     0     0
          da1       ONLINE       0     0     0
```

### 3 盘镜像（RAID1）

```sh
zpool create storage mirror da0 da1 da2
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           106749  24 49659  11           589971  27 457.3   5
Latency                       12593ms    4069ms               134ms     191ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
```

### 3 盘带区（RAID0）

```sh
zpool create storage da0 da1 da2
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           392378  59 86799  16           474157  22 829.3   8
Latency                         315us    4038ms               129ms     141ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          da0       ONLINE       0     0     0
          da1       ONLINE       0     0     0
          da2       ONLINE       0     0     0
```

### 3 盘 RAIDZ-1（RAID5）

```sh
zpool create storage raidz da0 da1 da2
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           225143  40 56590  11           619315  30 402.9   5
Latency                        2204ms    4052ms               896ms     177ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz1-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
```

### 4 盘 2 组镜像带区

```sh
zpool create storage mirror da0 da1 mirror da2 da3
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           226933  42 53878   9           644043  31 427.8   4
Latency                        1356ms    4066ms              1638ms     221ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
          mirror-1  ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
```

### 4 盘 RAIDZ-2（RAID6）

```sh
zpool create storage raidz2 da0 da1 da2 da3
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           204686  36 54769  13           183487   9 365.5   6
Latency                        5292ms    6356ms               449ms     233ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz2-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
```

### 5 盘 RAIDZ-1（RAID5）

```sh
zpool create storage raidz1 da0 da1 da2 da3 da4
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           469287  69 79083  15           598770  29 561.2   7
Latency                        1202us    4047ms               137ms     157ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz1-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
```

### 5 盘 RAIDZ-3（RAID7）

```sh
zpool create storage raidz3 da0 da1 da2 da3 da4
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           116395  21 45652   9           493679  25 397.7   5
Latency                       27836ms    6418ms             95509us     152ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz3-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
```

### 6 盘 3 组镜像带区

```sh
zpool create storage mirror da0 da1 mirror da2 da3 mirror da4 da5
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           389316  62 60493  11           655534  32 804.4   9
Latency                         549us    2473ms              1997ms     186ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
          mirror-1  ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
          mirror-2  ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
```

### 6 盘 RAIDZ-2（RAID6）

```sh
zpool create storage raidz2 da0 da1 da2 da3 da4 da5
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           429953  67 71505  14           488952  25 447.7   6
Latency                         358us    4057ms               197ms     181ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz2-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
```

### 10 盘 2 组 5 盘 RAIDZ-1 带区

```sh
zpool create storage raidz da0 da1 da2 da3 da4 raidz da5 da6 da7 da8 da9
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           675195  93 109169  21           1012768  50 817.6   9
Latency                       11619us    4471ms             84450us     110ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz1-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
          raidz1-1  ONLINE       0     0     0
            da5     ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
```

### 11 盘 RAIDZ-3（RAID7）

```sh
zpool create storage raidz3 da0 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           552264  79 103947  20           963511  48 545.6   8
Latency                        7373us    4045ms             84226us     144ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz3-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
            da10    ONLINE       0     0     0
```

### 12 盘 6 组镜像带区

```sh
zpool create storage mirror da0 da1 mirror da2 da3 mirror da4 da5 mirror da6 da7 mirror da8 da9 mirror da10 da11
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           643886  91 83717  15           962904  47  1257  13
Latency                       17335us    4040ms              1884ms     175ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
          mirror-1  ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
          mirror-2  ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
          mirror-3  ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
          mirror-4  ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
          mirror-5  ONLINE       0     0     0
            da10    ONLINE       0     0     0
            da11    ONLINE       0     0     0
```

### 12 盘 2 组 6 盘 RAIDZ-2 带区

```sh
zpool create storage raidz2 da0 da1 da2 da3 da4 da5 raidz2 da6 da7 da8 da9 da10 da11
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           638918  89 105631  21           990167  49 773.7  10
Latency                       15398us    6170ms               104ms     113ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz2-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
          raidz2-1  ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
            da10    ONLINE       0     0     0
            da11    ONLINE       0     0     0
```

### 12 盘 RAIDZ-1（RAID5）

```sh
zpool create storage raidz da0 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10 da11
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           689424  96 118573  23           993618  52 647.3   9
Latency                       14466us    3700ms               127ms     141ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz1-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
            da10    ONLINE       0     0     0
            da11    ONLINE       0     0     0
```

### 12 盘 RAIDZ-2（RAID6）

```sh
zpool create storage raidz2 da0 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10 da11
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           317741  45 98974  22           1065349  56 495.5   8
Latency                       11742ms    4062ms             90593us     150ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz2-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
            da10    ONLINE       0     0     0
            da11    ONLINE       0     0     0
```

### 12 盘 RAIDZ-3（RAID7）

```sh
zpool create storage raidz3 da0 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10 da11
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           452865  66 105396  24           840136  44 476.2   7
Latency                         706us    4069ms              2050ms     165ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz3-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
            da10    ONLINE       0     0     0
            da11    ONLINE       0     0     0
```

### 22 盘 2 组 11 盘 RAIDZ-3 带区

```sh
zpool create storage raidz3 da0 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10 raidz3 da11 da12 da13 da14 da15 da16 da17 da18 da19 da20 da21
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           567667  83 162462  32           1139088  59 770.3  13
Latency                        4581us    2700ms             78597us     116ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz3-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
            da10    ONLINE       0     0     0
          raidz3-1  ONLINE       0     0     0
            da11    ONLINE       0     0     0
            da12    ONLINE       0     0     0
            da13    ONLINE       0     0     0
            da14    ONLINE       0     0     0
            da15    ONLINE       0     0     0
            da16    ONLINE       0     0     0
            da17    ONLINE       0     0     0
            da18    ONLINE       0     0     0
            da19    ONLINE       0     0     0
            da20    ONLINE       0     0     0
            da21    ONLINE       0     0     0
```

### 23 盘 RAIDZ-3（RAID7）

```sh
zpool create storage raidz3 da0 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10 da11 da12 da13 da14 da15 da16 da17 da18 da19 da20 da21 da22
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           440656  64 157066  34           1146275  76 408.8   8
Latency                       21417us    2324ms               154ms     195ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz3-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
            da10    ONLINE       0     0     0
            da11    ONLINE       0     0     0
            da12    ONLINE       0     0     0
            da13    ONLINE       0     0     0
            da14    ONLINE       0     0     0
            da15    ONLINE       0     0     0
            da16    ONLINE       0     0     0
            da17    ONLINE       0     0     0
            da18    ONLINE       0     0     0
            da19    ONLINE       0     0     0
            da20    ONLINE       0     0     0
            da21    ONLINE       0     0     0
            da22    ONLINE       0     0     0
```

### 24 盘 12 组镜像带区

```sh
zpool create storage mirror da0 da1 mirror da2 da3 mirror da4 da5 mirror da6 da7 mirror da8 da9 mirror da10 da11 mirror da12 da13 mirror da14 da15 mirror da16 da17 mirror da18 da19 mirror da20 da21 mirror da22 da23
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           696733  94 144526  27           898137  51  1520  16
Latency                       17247us    2850ms              2014ms   79930us
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME         STATE     READ WRITE CKSUM
        storage      ONLINE       0     0     0
          mirror-0   ONLINE       0     0     0
            da0      ONLINE       0     0     0
            da1      ONLINE       0     0     0
          mirror-1   ONLINE       0     0     0
            da2      ONLINE       0     0     0
            da3      ONLINE       0     0     0
          mirror-2   ONLINE       0     0     0
            da4      ONLINE       0     0     0
            da5      ONLINE       0     0     0
          mirror-3   ONLINE       0     0     0
            da6      ONLINE       0     0     0
            da7      ONLINE       0     0     0
          mirror-4   ONLINE       0     0     0
            da8      ONLINE       0     0     0
            da9      ONLINE       0     0     0
          mirror-5   ONLINE       0     0     0
            da10     ONLINE       0     0     0
            da11     ONLINE       0     0     0
          mirror-6   ONLINE       0     0     0
            da12     ONLINE       0     0     0
            da13     ONLINE       0     0     0
          mirror-7   ONLINE       0     0     0
            da14     ONLINE       0     0     0
            da15     ONLINE       0     0     0
          mirror-8   ONLINE       0     0     0
            da16     ONLINE       0     0     0
            da17     ONLINE       0     0     0
          mirror-9   ONLINE       0     0     0
            da18     ONLINE       0     0     0
            da19     ONLINE       0     0     0
          mirror-10  ONLINE       0     0     0
            da20     ONLINE       0     0     0
            da21     ONLINE       0     0     0
          mirror-11  ONLINE       0     0     0
            da22     ONLINE       0     0     0
            da23     ONLINE       0     0     0
```

### 24 盘 RAIDZ-1（RAID5）

```sh
zpool create storage raidz da0 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10 da11 da12 da13 da14 da15 da16 da17 da18 da19 da20 da21 da22 da23
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           567164  82 198696  42           1304530  81 620.7  11
Latency                       36614us    2252ms             70871us     141ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz1-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
            da10    ONLINE       0     0     0
            da11    ONLINE       0     0     0
            da12    ONLINE       0     0     0
            da13    ONLINE       0     0     0
            da14    ONLINE       0     0     0
            da15    ONLINE       0     0     0
            da16    ONLINE       0     0     0
            da17    ONLINE       0     0     0
            da18    ONLINE       0     0     0
            da19    ONLINE       0     0     0
            da20    ONLINE       0     0     0
            da21    ONLINE       0     0     0
            da22    ONLINE       0     0     0
            da23    ONLINE       0     0     0
```

### 24 盘 RAIDZ-2（RAID6）

```sh
zpool create storage raidz2 da0 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10 da11 da12 da13 da14 da15 da16 da17 da18 da19 da20 da21 da22 da23
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           434394  63 189521  39           1063626  75 516.9  12
Latency                       11369us    2130ms             80053us     153ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz2-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
            da10    ONLINE       0     0     0
            da11    ONLINE       0     0     0
            da12    ONLINE       0     0     0
            da13    ONLINE       0     0     0
            da14    ONLINE       0     0     0
            da15    ONLINE       0     0     0
            da16    ONLINE       0     0     0
            da17    ONLINE       0     0     0
            da18    ONLINE       0     0     0
            da19    ONLINE       0     0     0
            da20    ONLINE       0     0     0
            da21    ONLINE       0     0     0
            da22    ONLINE       0     0     0
            da23    ONLINE       0     0     0
```

### 24 盘 RAIDZ-3（RAID7）

```sh
zpool create storage raidz3 da0 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10 da11 da12 da13 da14 da15 da16 da17 da18 da19 da20 da21 da22 da23
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           405579  62 180470  39           1117221  70 592.4  13
Latency                         622ms    1959ms             94830us     187ms
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          raidz3-0  ONLINE       0     0     0
            da0     ONLINE       0     0     0
            da1     ONLINE       0     0     0
            da2     ONLINE       0     0     0
            da3     ONLINE       0     0     0
            da4     ONLINE       0     0     0
            da5     ONLINE       0     0     0
            da6     ONLINE       0     0     0
            da7     ONLINE       0     0     0
            da8     ONLINE       0     0     0
            da9     ONLINE       0     0     0
            da10    ONLINE       0     0     0
            da11    ONLINE       0     0     0
            da12    ONLINE       0     0     0
            da13    ONLINE       0     0     0
            da14    ONLINE       0     0     0
            da15    ONLINE       0     0     0
            da16    ONLINE       0     0     0
            da17    ONLINE       0     0     0
            da18    ONLINE       0     0     0
            da19    ONLINE       0     0     0
            da20    ONLINE       0     0     0
            da21    ONLINE       0     0     0
            da22    ONLINE       0     0     0
            da23    ONLINE       0     0     0
```

### 24 盘带区（RAID0）

```sh
zpool create storage da0 da1 da2 da3 da4 da5 da6 da7 da8 da9 da10 da11 da12 da13 da14 da15 da16 da17 da18 da19 da20 da21 da22 da23
bonnie++ -u root -r 1024 -s 16384 -d /storage -f -b -n 1 -c 4
```

```
Version  1.97       ------Sequential Output------ --Sequential Input- --Random-
Concurrency   4     -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
FreeBSDzfs      16G           692351  95 260259  50           1377547  75  1921  19
Latency                       12856us    1670ms             49017us   59388us
```

```sh
zpool status
```

```
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          da0       ONLINE       0     0     0
          da1       ONLINE       0     0     0
          da2       ONLINE       0     0     0
          da3       ONLINE       0     0     0
          da4       ONLINE       0     0     0
          da5       ONLINE       0     0     0
          da6       ONLINE       0     0     0
          da7       ONLINE       0     0     0
          da8       ONLINE       0     0     0
          da9       ONLINE       0     0     0
          da10      ONLINE       0     0     0
          da11      ONLINE       0     0     0
          da12      ONLINE       0     0     0
          da13      ONLINE       0     0     0
          da14      ONLINE       0     0     0
          da15      ONLINE       0     0     0
          da16      ONLINE       0     0     0
          da17      ONLINE       0     0     0
          da18      ONLINE       0     0     0
          da19      ONLINE       0     0     0
          da20      ONLINE       0     0     0
          da21      ONLINE       0     0     0
          da22      ONLINE       0     0     0
          da23      ONLINE       0     0     0
```

## 固态（纯 SSD）RAID

24 槽 RAID 机箱里安装的是 Samsung 840 256GB SSD (MZ-7PD256BW) 驱动器。通过安装在 PCIe 16x 插槽的 Avago LSI 9207-8i HBA 控制器连接。ZFS 在驱动层面实现了 COW，就无需使用 TRIM。

| 驱动器数（Nx 256GB） | 配置             | 容量   | 写（MB/s） | 改写（MB/s） | 读（MB/s） |
| -------------------- | ---------------- | ------ | ---------- | ------------ | ---------- |
| 1                    | 单盘             | 232 GB | 441        | 224          | 506        |
| 2                    | 镜像（RAID1）    | 232 GB | 430        | 300          | 990        |
| 2                    | 带区（RAID0）    | 464 GB | 933        | 457          | 1020       |
| 3                    | RAIDZ-1（RAID5） | 466 GB | 751        | 485          | 1427       |
| 4                    | RAIDZ-2（RAID6） | 462 GB | 565        | 442          | 1925       |
| 5                    | RAIDZ-1（RAID5） | 931 GB | 817        | 610          | 1881       |
| 5                    | RAIDZ-3（RAID7） | 464 GB | 424        | 316          | 1209       |
| 6                    | RAIDZ-2（RAID6） | 933 GB | 721        | 530          | 1754       |
| 7                    | RAIDZ-3（RAID7） | 934 GB | 591        | 436          | 1713       |
| 9                    | RAIDZ-1（RAID5） | 1.8 TB | 868        | 618          | 1978       |
| 10                   | RAIDZ-2（RAID6） | 1.8 TB | 806        | 511          | 1730       |
| 11                   | RAIDZ-3（RAID7） | 1.8 TB | 659        | 448          | 1681       |
| 17                   | RAIDZ-1（RAID5） | 3.7 TB | 874        | 574          | 1816       |
| 18                   | RAIDZ-2（RAID6） | 3.7 TB | 788        | 532          | 1589       |
| 19                   | RAIDZ-3（RAID7） | 3.7 TB | 699        | 400          | 1183       |
| 24                   | 带区（RAID0）    | 5.5 TB | 1620       | 796          | 2043       |

## 对结果的一些思考

RAID6 和 RAIDZ-2 是一种很多的对三个属性的折中方案。速度、一致性和容量都还并不错。能够丢失两块驱动器很适合于家用或办公室，您可以在短时间内就更换另一块磁盘。但对于数据中心来说，必须有热备才适合使用 RAIDZ-2（RAID6）。否则还是 RAIDZ-3（RAID7）更合适，因为管理员可能需要花一周时间才能到达现场更换磁盘。当然即便您将 RAIDZ-2（RAID6） 和热备一起使用，也可以考虑 RAIDZ-3（RAID7）。同样的容量，还可以丢失任意三块驱动器。

我们发现无论使用哪种机械硬盘驱动器，速度和延迟都大致相同。您无需像我们一样使用昂贵的 SAS 驱动器。只要制造商在其驱动器上标记了 RE 或者“启用 RAID”再或“针对 NAS 系统”字样即可。部分西部数据 3TB 红盘（WD30EFRX）的表现也不错。7200rpm 的驱动器大约要比 5400rpm 的快上 10%，但也会更热。

我们建议不要使用省电的或 ECO 模式的驱动器，比如西部数据绿盘（WD30EZRX）系列。这些驱动器因为降速、休眠模式和较长的 SATA 时序，在 RAID 配置中的表现很糟糕。如果您追求速度和最佳性能，请使用 SSD 并启用 ZFS 压缩。

池的可靠性与磁盘数量、硬件质量和重构时间呈线性关系。磁盘越多可用性越低，因为出现磁盘故障的可能性变高了。企业级 SAS 驱动器会比桌面级个人驱动器提供更高的可用性，因为两次故障的间隔时间会更长。RAID 重构得越慢，那么在此期间出现另一驱动器故障的概率就会越高。为了应对更多磁盘池失败的可能性，我们应增加奇偶校验磁盘的数量。较高的奇偶校验计数将减少平均数据丢失时间（Mean Time To Data Loss，MTTDL）。

一些随机观察结果：始终使用 LZ4 压缩，并向机器中塞尽可能多的内存条。当决定要使用多少个驱动器时，类型无关重要，SAS 和 SATA 都可以。池中的磁盘越多，用于奇偶校验的空间损失越少，空间效率越高。硬盘老化是分批的，尝试使用不同生产日期的、或不同厂家的驱动器。通常，对于随机 IOP，镜像更合适，而 RAIDZ-1、RAIDZ-2 和 RAIDZ-3 则看不出区别。

## 是否压缩？

是的，启用 LZ4 压缩。指令 `zfs set compression=lz4 tank` 会启用 tank 池的压缩。较长的答案也是肯定的，但您可以了解其中优劣，以便做出明智的决定。

通常来说机械硬盘确实很慢。仅等待数据往返硬盘，操作系统就会浪费大量时间。反对压缩的主要观点是说其在读写阵列时会占用 CPU 时间和增加延迟。但如果您使用计算机已有一段时间的话，您可能还记得 1990 年代的“Stacker”。理想情况下，我们希望系统能尽可能地减少与驱动器联系因为它们确实太慢了。

压缩可以减少驱动器需要访问的块数量从而用 CPU 时间换取更快的速度。您所得到的是更少的块读写，仅中位可压缩的文件都可以音效增加驱动器盘片的面密度。压缩让驱动器磁头只用在较小区域内移动便能访问相同数量的数据。

ZFS 的开发人员默认禁用了 LZ4 压缩。请注意 ZFS 和压缩使用多个 CPU 核。因此核越多越快，压缩就越快。启用 LZ4 压缩时，系统会比直接读写要多花费额外 20% 的 CPU 时间来压缩和解压缩文件。平均而言，LZ4 的最大压缩比在 2 到 3 倍之间。

我们的测试显示单驱动器读取 LZJB 压缩的 Bonnie++ 测试数据的速度都能够从 150MB/s 提升到 1174MB/s。即便是如 bzip2、zip、mp3、avi、mkv 或 webm 之类的压缩文件也不会浪费很多 CPU 时间，因为 LZ4 是一种一点都不昂贵的压缩方法。

在 FreeBSD 10 中，LZJB 和 LZ4 均可在 ZFS 中使用。LZJB 很快，但 LZ4 的 500MBps 压缩速度和 1.5GB/s 解压速度更快。LZ4 和 LZJB 对比压缩能快 50%、解压能快 80%。在 FreeBSD 9.1 中您可能需要使用 `zpool set feature@lz4_compress=enabled poolname` 启用池的 LZ4，因为 LZ4 无法与更早版本的 ZFS 兼容。

LZ4 在不可压缩数据上的性能表现同样令人印象深刻。LZ4 通过“提前中止”机制来实现更高的性能，该机制会在 LZ4 的实际压缩表现不足 12.5% 时触发。对那些希望给所有池都启用压缩的管理员们而言，LZ4 堪称完美。在全部卷上启用 LZ4 意味着只会在合适时压缩，并不会将 CPU 时间浪费在不可压缩的数据上，也不会因为透明解压缩而导致读取延迟。太不可思议了。

下表展示了许多 RAID 类型在不压缩、LZJB 压缩和 LZ4 压缩时的表现。Bonnie++ 测试所使用地是类数据库的文件，对压缩很友好。您会注意到随着我们添加更多的物理磁盘，压缩对性能的影响越小。事实上，我们只用了三块磁盘就达到了启用 LZ4 压缩的 RAIDZ-1（RAID5）的读写速度的最大化。我们的建议是对索引阵列都启用压缩，因为获得的吞吐量提升是超过 CPU 消耗的，而且压缩数据还可以节约空间。

| 压缩 | 驱动器    | 配置             | 容量   | 写（MB/s） | 改写（MB/s） | 读（MB/s） | SSD |
| ---- | --------- | ---------------- | ------ | ---------- | ------------ | ---------- | --- |
| 无   | 1x 2TB    | 单盘             | 1.8TB  | 131        | 66           | 150        |
| LZJB | 1x 2TB    | 单盘             | 1.8TB  | 445        | 344          | 1174       |
| LZ4  | 1x 2TB    | 单盘             | 1.8TB  | 471        | 351          | 1542       |
| 无   | 1x 256GB  | 单盘             | 232 GB | 441        | 224          | 506        | SSD |
| LZJB | 1x 256GB  | 单盘             | 232 GB | 510        | 425          | 1290       | SSD |
| 无   | 2x 2TB    | 镜像（RAID1）    | 1.8 TB | 126        | 79           | 216        |
| LZJB | 2x 2TB    | 镜像（RAID1）    | 1.8 TB | 461        | 386          | 1243       |
| LZ4  | 2x 2TB    | 镜像（RAID1）    | 1.8 TB | 398        | 354          | 1537       |
| 无   | 3x 2TB    | RAIDZ-1（RAID5） | 3.6 TB | 279        | 131          | 281        |
| LZJB | 3x 2TB    | RAIDZ-1（RAID5） | 3.6 TB | 479        | 366          | 1243       |
| LZ4  | 3x 2TB    | RAIDZ-1（RAID5） | 3.6 TB | 517        | 453          | 1587       |
| 无   | 5x 2TB    | RAIDZ-1（RAID5） | 7.1 TB | 469        | 173          | 406        |
| LZJB | 5x 2TB    | RAIDZ-1（RAID5） | 7.1 TB | 478        | 392          | 1156       |
| LZ4  | 5x 2TB    | RAIDZ-1（RAID5） | 7.1 TB | 516        | 437          | 1560       |
| 无   | 5x 256GB  | RAIDZ-1（RAID5） | 931 GB | 817        | 610          | 1881       | SSD |
| LZJB | 5x 256GB  | RAIDZ-1（RAID5） | 931 GB | 515        | 415          | 1223       | SSD |
| 无   | 7x 2TB    | RAIDZ-3（RAID7） | 7.1 TB | 393        | 169          | 423        |
| LZJB | 7x 2TB    | RAIDZ-3（RAID7） | 7.1 TB | 469        | 378          | 1127       |
| LZ4  | 7x 2TB    | RAIDZ-3（RAID7） | 7.1 TB | 507        | 436          | 1532       |
| 无   | 12x 2TB   | RAIDZ-1（RAID5） | 19 TB  | 521        | 272          | 738        |
| LZJB | 12x 2TB   | RAIDZ-1（RAID5） | 19 TB  | 487        | 391          | 1105       |
| LZ4  | 12x 2TB   | RAIDZ-1（RAID5） | 19 TB  | 517        | 441          | 1557       |
| 无   | 17x 2TB   | RAIDZ-1（RAID5） | 28 TB  | 468        | 267          | 874        |
| LZJB | 17x 2TB   | RAIDZ-1（RAID5） | 28 TB  | 478        | 380          | 1096       |
| LZ4  | 17x 2TB   | RAIDZ-1（RAID5） | 28 TB  | 502        | 430          | 1473       |
| 无   | 24x 2TB   | RAIDZ-1（RAID5） | 40 TB  | 528        | 291          | 929        |
| LZJB | 24x 2TB   | RAIDZ-1（RAID5） | 40 TB  | 478        | 382          | 1081       |
| LZ4  | 24x 2TB   | RAIDZ-1（RAID5） | 40 TB  | 504        | 431          | 1507       |
| 无   | 24x 256GB | 带区（RAID0）    | 5.5 TB | 1340       | 796          | 2037       | SSD |
| LZJB | 24x 256GB | 带区（RAID0）    | 5.5 TB | 1032       | 844          | 2597       | SSD |

## 并非所有的 SATA 控制器都一样

RAID 的性能高度依赖于硬件和操作系统的驱动程序。如果使用板载 SATA 连接器，性能就不如 SATA 扩展器或专用 RAID 卡。因为 SATA 端口说是遵循了 SATA 6 ，也使用了漂亮的线缆，但传输速度并不一定就真快。板载芯片组一般都是制造商可以找到的最廉价的方案。就和大多数板载网口无法达标的情况差不多。

在邮件列表和论坛中有帖子说 ZFS 奇慢无比。我们在上文中已经展示了硬件的局限在哪，和应该如何创建 RAID 文件系统才能获取惊人速度。我们怀疑许多 ZFS 的反对者都在使用很慢的或不达标的 I/O 子系统来搭建他们的 ZFS 系统。

我们在 FreeBSD 中分别使用三种不同的接口，先后测试了 Western Digital Black 2TB SATA6 (WD2002FAEX) 机械硬盘和 Samsung 840 PRO 256GB 固态硬盘的 SATA 吞吐量。包括一块华硕游戏主板的板载 SATA 6 Gbit/s 端口、一块超微服务器主板和一张 LSI MegaRAID 阵列卡。全部机器都是 6 核 CPU 和 16GB 内存。根据 Western Digital 官方的说法，黑盘系列 2TB（WD2002FAEX）理论应该能达到 150MB/s 的顺序读写。Samsung 840 Pro 的额定读取速度为 540MB/s，写入速度为 520MB/s。注意板载 SATA 端口（SATA 3 和 SATA 6）和专用卡的吞吐量差距。

| 控制器                             | 驱动器      | 写（MB/s） | 改写（MB/s） | 读（MB/s） |
| ---------------------------------- | ----------- | ---------- | ------------ | ---------- |
| Asus Sabertooth 990FX 板载 SATA 6  | WD2002FAEX  | 39         | 25           | 91         |
| SuperMicro X9SRE 板载 SATA 3       | WD2002FAEX  | 31         | 22           | 89         |
| LSI MegaRAID 9265-8i SATA 6 "JBOD" | WD2002FAEX  | 130        | 66           | 150        |
| Asus Sabertooth 990FX 板载 SATA 6  | MZ-7PD256BW | 242        | 158          | 533        |
| LSI MegaRAID 9265-8i SATA 6 "JBOD" | MZ-7PD256BW | 438        | 233          | 514        |

## 如何在 ZFS 池上 4K 对齐？

默认情况下，较旧的硬盘驱动器的扇区大小是 512 字节。当驱动器变得足够大时，扇区大小发生了变化，因为跟踪扇区也需要消耗很大一部分存储空间。许多现代驱动器被认定为高级格式驱动器，即 4096 字节的扇区大小。包括了所有的固态硬盘和大多数 2TB 以上的磁驱动器。您可以在制造商官网中查看驱动器的“扇区大小”或“扇区对齐”。

ZFS 会查询底层设备的扇区大小，并以此确定其动态宽度条带的大小。只要硬盘不撒谎，这都是种好办法。但可惜的是如今驱动器硬件普遍撒谎。驱动器说逻辑扇区大小是 512 字节（ashift=9 或 2^9=512），但实际物理扇区是 4KB（ashift=12 或 2^12=4096）。ZFS 会因为驱动器的谎言而错误地将条带对齐为 512 字节。这意味着条带几乎总是对不齐的，底层设备只能各自处理，降低了整体写入性能。在实践中，未对齐时的写入速度只比对齐后慢了不到 10%。但既然是优化，每个 B/s 都值得计较哇！

因此，我们需要忽略硬件告诉我们的事实，强制手动调整 ZFS 扇区对齐。这个过程并不难，但不太直观。让我们创建一个名为 tank 的单盘 ZFS 池。然后使用 `gnop` 来创建 .nop 设备，强迫 ZFS 将物理扇区对齐成 4K。

```sh
# 创建 4K 对齐的单盘 ZFS 池 tank

# 如果有未对齐的池，应该先销毁。
# 按需备份
zpool destroy tank

# 需要创建 4K gnop 设备来链到 ZFS 池
# 强制 ZFS 使用 4K 物理扇区
gnop create -S 4096 /dev/mfid0
zpool create tank /dev/mfid0.nop

# 接下来导出池
# 删掉 gnop 设备
zpool export tank
gnop destroy /dev/mfid0.nop

# 重新导入对齐的池
zpool import tank

# 完工。检查确保 ashift 为 12 就说明池是 4K 对齐的。
zdb -C tank | grep ashift
```

```
      ashift: 12
```

## 4K 对齐的池的性能

那么，正确地 4K 对齐了的 ZFS 池，性能上到底有什么区别？老实说，对一两个机械硬盘而言区别不大，通常都不到 10% 的速度变化。对齐后可以看到硬盘速度的轻微提升，是因为硬盘驱动器自身不再需要做扇区拆分和写调整（Read-Modify-Write）操作。

性能差异似乎还取决于使用的驱动器类型。与 RE（RAID Enabled）驱动器相比，即使 4K 对齐，绿盘或 ECO 驱动器的性能也很差。为了使事情变得更复杂，同一个驱动器制造商的某些驱动器就要比其它系列要慢。确保充分利用设备的唯一方法是将驱动器格式化为 512B 和 4K 后再使用 Bonnie++ 测试。如果有许多驱动器，就需要分别测试每个驱动器，然后将“快的”驱动器放在一个 RAID 中，而将“慢的”驱动器放入另一个中。将快速和慢速驱动器混合使用只会使整个装置都变慢。这很痛苦，但单独测试的方法再可靠不过。

对新的引导驱动器或 RAID，我们还是建议要对齐扇区。如果驱动器未对齐，并要擦除数据再重新对齐，请记住这些数字，测试您自己的设备并做出明智的决定。

| 驱动器      | 扇区大小 | 写（MB/s） | 改写（MB/s） | 读（MB/s） |
| ----------- | -------- | ---------- | ------------ | ---------- |
| WD2001FYYG  | 512B     | 130        | 65           | 146        |
| WD2001FYYG  | 4K       | 131        | 66           | 150        |
| MZ-7PD256BW | 512B     | 446        | 232          | 520        |
| MZ-7PD256BW | 4K       | 438        | 233          | 514        |

## 答疑

### 有可用的 ZFS 健康脚本吗？

可以看见我们的 [ZFS 健康状态检查脚本](https://calomel.org/zfs_health_check_script.html)。检查磁盘与卷的错误，以及池的运行状况，甚至在完成最后一次池的清理时。

### 为什么 ZFS 导入无法使用 GPT 标签？

导入卷时必须使用 `-d` 参数指明 GPT 标签所在目录。举例说明，我们的 GPT 标签以 `data1-XX` 打头，存在 Ubuntu 的 `/dev/disk/by-partlabel` 里。

```sh
zpool import -d /dev/disk/by-partlabel/ data1
zpool status
```

```
  pool: data1
 state: ONLINE
  scan: scrub repaired 0 in 0h0m with 0 errors on Mon Apr  8 14:36:17 2033
config:

        NAME          STATE     READ WRITE CKSUM
        data1         ONLINE       0     0     0
          raidz2-0    ONLINE       0     0     0
            data1-00  ONLINE       0     0     0
            data1-01  ONLINE       0     0     0
            data1-02  ONLINE       0     0     0
            data1-03  ONLINE       0     0     0
            data1-04  ONLINE       0     0     0
            data1-05  ONLINE       0     0     0
            data1-06  ONLINE       0     0     0
            data1-07  ONLINE       0     0     0
            data1-08  ONLINE       0     0     0
            data1-09  ONLINE       0     0     0
            data1-10  ONLINE       0     0     0
            data1-11  ONLINE       0     0     0
```

```sh
# 直接 zpool import 会使用原始设备名称
zpool import data1
zpool status
```

```
  pool: data1
 state: ONLINE
  scan: resilvered 0 in 0h0m with 0 errors on Mon Apr  8 14:33:58 2033
config:

        NAME                                              STATE     READ WRITE CKSUM
        data1                                             ONLINE       0     0     0
          raidz2-0                                        ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abe327b8d44f-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abe427c0c540-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abe427c9410f-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abe527d1dc47-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5c8456d9cb386-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abe627e2e502-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abe627eb60b9-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abe727fa267a-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abe82802c19f-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abe92811a6f3-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abea28205d1e-part1  ONLINE       0     0     0
            scsi-3600605b00512e8c018f5abeb282e7776-part1  ONLINE       0     0     0
```
