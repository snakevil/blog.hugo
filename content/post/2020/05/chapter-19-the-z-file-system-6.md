---
title: "【译】第 19 章：Z 文件系统㈥"
slug: "第19章：Z文件系统六"
date: 2020-05-14T16:39:58+08:00
category:
    - Linux
    - 博闻广记
tag:
    - ZFS
    - FreeBSD
    - Linux
    - 自主翻译
---

[点击这里阅读原文。](https://www.freebsd.org/doc/handbook/zfs-advanced.html)

---

## 19.6 进阶主题

<!--more-->

[arc]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-arc
[sysctl(8)]: https://www.freebsd.org/cgi/man.cgi?query=sysctl&sektion=8&manpath=freebsd-release-ports
[镜像]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-vdev-mirror
[raid-z]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-vdev-raidz
[l2arc]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-l2arc
[事务组]: https://www.freebsd.org/doc/handbook/zfs-term.html#zfs-term-txg

## 19.6.1 调优

有许多可调参数可以调整，以使 ZFS 在不同的工况下表现最佳。

-   `vfs.zfs.arc_max` - [ARC][] 的最大值。默认取 1GB 或内存总大小的 5/8 中较大的值。但如果系统要运行其它需要内存的进程或守护进程，就应该取较低的值。在运行时可以使用 [sysctl(8)][] 调整这个值，或在 `/boot/loader.conf` 或 `/etc/sysctl.conf` 中设置。

-   `vfs.zfs.arc_meta_limit` - 限制 ARC 可用于存储元数据的部分。默认值是 `vfs.zfs.arc_max` 的 1/4。如果工况涉及对大量文件和目录的操作或频繁的元数据操作，提高此值能提升性能，损失则是 ARC 中能存的文件数据量减少。在运行时可以使用 [sysctl(8)][] 调整这个值，或在 `/boot/loader.conf` 或 `/etc/sysctl.conf` 中设置。

-   `vfs.zfs.arc_min` - [ARC][] 的最小值。默认值是 `vfs.zfs.arc_meta_limit` 的一半。调整这个值可以避免其它应用程序压垮整个 ARC。在运行时可以使用 [sysctl(8)][] 调整这个值，或在 `/boot/loader.conf` 或 `/etc/sysctl.conf` 中设置。

-   `vfs.zfs.vdev.cache.size` - 池中每个设备的内存缓存的预分配数量。实际占用的内存总量为该值和设备数量的乘积。该值只能在引导阶段调整，或在 `/boot/loader.conf` 中设置。

-   `vfs.zfs.min_auto_ashift` - 创建池时自动调用的扇区大小（ashift）最小值。值必须是 2 的幂值。默认值 9 指 2^9 = 512，即 512 字节的扇区大小。为避免 _写放大_ 并获得最佳性能，请将此值设置为池中设备使用的最大扇区大小。

    许多设备的扇区大小是 4KB。在这些设备上使用 ashift 默认值 9 就会造成写放大。4KB 数据会分 512 字节写入八次。ZFS 在创建池时会读取设备的扇区大小，但很多 4KB 扇区的设备因为兼容性而报告成 512 字节。在创建池之前设置 `vfs.zfs.min_auto_ashift` 为 12 （2^12 = 4096）可以强制 ZFS 使用 4KB 块以获取最好的性能。

    强制使用 4KB 块也有助于在将来升级池中的磁盘。因为以后的磁盘更多使用 4KB 扇区，但一旦池被创建，其 ashift 值就无法再被修改。

    在某些特殊情况下，较小的 512 字节块大小反而是更适合的。特别是数据库和虚拟机存储器所使用的 512 字节扇区的磁盘，小数据量随机读操作只需传输更少量的数据即可。这可以提供更好的性能，尤其是在使用较小的 ZFS 记录大小时。

*   `vfs.zfs.prefetch_disable` - 禁用预取。其值 0 为启用 1 为禁用。除非系统内存不足 4GB，否则默认值取 0。预取会读取比实际向 ARC 请求的要大的数据块，以期望能很快用上。如果工况有大量的随机读操作，禁用预取会减少不必要的读行为，从而提高效率。可随时使用 [sysctl(8)][] 调整这个值。

*   `vfs.zfs.vdev.trim_on_init` - 控制添加到池中的新设备是否运行 TRIM 指令。这确保 SSD 的最佳性能和寿命，但会花费额外的实践。如果设备被安全擦除过，禁用此设置会加快新设备的添加速度。可随时使用 [sysctl(8)][] 调整这个值。

*   `vfs.zfs.vdev.max_pending` - 限制每个设备的 I/O 请求等待量。较高的值可以让设备的命令队列更满符合工作，因此吞吐量更高。较低的值会减少等待实践。可随时使用 [sysctl(8)][] 调整这个值。

*   `vfs.zfs.top_maxinflight` - 么个顶层 vdev 的未完成 I/O 的最大数量。通过限制命令队列的长度来防止高延迟。这个限制针对单个顶层虚拟设备，所以也包括[镜像][]、[RAID-Z][] 和其它虚拟设备。可随时使用 [sysctl(8)][] 调整这个值。

*   `vfs.zfs.l2arc_write_max` - 限制每秒写入 [L2ARC][] 的数据量。此调优项旨在通过限制写入设备的数据量来延长 SSD 的使用寿命。可随时使用 [sysctl(8)][] 调整这个值。

*   `vfs.zfs.l2arc_write_boost` - 此调优项的值会加入 `vfs.zfs.l2arc_write_max` 中以提高 SSD 的写入速度直至碰到不在 [L2ARC][] 中的块。此“涡轮预热阶段”旨在减少重新启动后 [L2ARC][] 为空导致的性能损失。可随时使用 [sysctl(8)][] 调整这个值。

*   `vfs.zfs.scrub_delay` - 清理时每个 I/O 等待的刻数。为了确保清理操作不会干扰池的正常运行，一旦出现其它 I/O，清理将会在每个指令间延迟。这个值控制清理的 IOPS（每秒读写，I/Os Per Second）总量。设置的粒度由 kern.hz （默认每秒 1000 刻）的值决定。通过变更设置可以调整不同的 IOPS 限制。默认值为 4，因此限制为 1000/4 = 250 IOPS。值为 20 时限制则为 1000/20 = 50 IOPS。仅当池中有活动时才会根据 `vs.zfs.scan_idle` 限制清理速度。可随时使用 [sysctl(8)][] 调整这个值。

*   `vfs.zfs.resilver_delay` - 重构时每个 I/O 等待的毫秒数。为了确保重构操作不会干扰池的正常运行，一旦出现其它 I/O，重构将会在每个指令间延迟。这个值控制重构的每秒 IOPS 总量。设置的粒度由 kern.hz 的值决定。通过变更设置可以调整不同的 IOPS 限制。默认值为 2，因此限制为 1000/2 = 500 IOPS。如果另一个设备发生故障可能会使池发生故障，从而导致数据丢失，则使池返回联机状态可能更为重要。 值为 0 时重构操作与其他操作相同的优先级，从而加快恢复过程。仅当池中有活动时才会根据 `vs.zfs.scan_idle` 限制重构速度。可随时使用 [sysctl(8)][] 调整这个值。

*   `vfs.zfs.scan_idle` - 判断池在最后一次操作之后多久算发呆的毫秒数。发呆的池不对清理和重构限速。可随时使用 [sysctl(8)][] 调整这个值。

*   `vfs.zfs.txg.timeout` - [事务组][]间的最大秒数。每隔这一时长就将当前事务组写入池中并重新开启新的事务组。但当事务组的数据太多时会立即写入并新开。默认值是 5 秒。较大的值通过延迟异步写入来提高读取性能，但在写入事务组时可能会导致性能不均衡。可随时使用 [sysctl(8)][] 调整这个值。

## 19.6.2 i386 架构中的 ZFS

ZFS 的部分功能特性需要大量内存，因此在系统内存受限时需要调优以获取最大效率。

### 19.6.2.1 内存

总系统内存不能小于 1GB。推荐内存大小视池的大小和用到哪些 ZFS 功能特性而定。常用经验是每 1TB 的存储空间需要 1TGB 的内存。而去重功能则每 1TB 的存储空间需要 5GB 的内存才满足要求。尽管有用户使用较少的内存就成功运行了 ZFS，但一旦内存耗尽系统负荷会迅速加重。对内存低于推荐值得系统，可能还需进一步调优。

### 19.6.2.2 内核配置

因为 i386 平台的寻址限制，必须将下面的选项保存至自定义内核配置文件、重新构建内核、再重启，才能使用 ZFS：

```
options        KVA_PAGES=512
```

这扩展了内核寻址空间，允许 `vm.kvm_size` 可调项能突破 1GB 的当前强行限制，和 2GB 的 PAE 限制。将寻址空间以 MB 为单位的大小值除去 4 便可找到最合适的值。比如对 2GB 而言就是 512。

### 19.6.2.3 加载器可调项

所有 FreeBSD 架构都可以增加 _kmem_ 寻址空间。在只有 1GB 物理内存的测试系统中，将下面的选项添加到 `/boot/loader.conf` 并重启，可以正常工作：

```ini
vm.kmem_size="330M"
vm.kmem_size_max="330M"
vfs.zfs.arc_max="40M"
vfs.zfs.vdev.cache.size="5M"
```

关于 ZFS 相关的调优建议的细节列表，详见 [https://wiki.freebsd.org/ZFSTuningGuide](https://wiki.freebsd.org/ZFSTuningGuide)。
