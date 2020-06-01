---
title: "OpenWRT 安装配置手记"
slug: "OpenWRT安装配置手记"
date: 2020-05-30T20:11:27+08:00
category:
    - 家庭组网
tag:
    - OpenWRT
    - N3150
---

OpenWRT 现在的版本是支持 x86 的。这样我的 N3150 还能勉强再战几年，用到彻底报废为止。毕竟 8G 内存和 1T 硬盘，做个软路由真是干啥不能干呀！

<!--more-->

## 一 安装

一般在路由器中刷 OpenWRT，都是在浏览器中管理界面直接上传 ROM 镜像。本质上是用硬盘镜像直接还原覆写硬盘。耳熟能详地类似的例子，是曾经的 Ghost。

但这种习以为常的方式搁在 x86 平台上，就有点对不上了。我在网上翻了翻，居然还有人直接把硬盘拆下来塞进硬盘盒，来做覆写操作！这简直了！

还好[也有理智人，找到了正确的命令行工具](https://weisunit.com/2019/08/28/%E8%AE%B0lede%E8%BD%AF%E8%B7%AF%E7%94%B1x86-64%E5%AE%89%E8%A3%85/)。

照做，就完成安装了。

## 二 通网

配置 LAN 和 WAN，其实很容易就可以搞定。

但很诡异地是，N3150 的 LAN 配置明明是 `10.0.0.10/24`。DHCP 分配出去的地址信息里网关和 DNS 服务器居然会是 `10.0.0.101`。而这个地址，其实是一台 [TL-LA1000W](/2020/05/家庭组网升级规划/)。

虽然在 TL-LA1000W 的系统日志里可以看到来源于 `DHCPD` 的分配日志，但把管理界面翻烂了都没找到哪儿有配置项。这就很糙蛋了！只能解释成：**TL-LA1000W 很智能地会在网络中没有 DHCP 服务时自己顶上。**

在 OpenWRT 里找 DHCP 设置网关的选项，没有。重启 N3150，没用。唔，开始有点上头了！

通过 SSH [直接 uci 配置](https://openwrt.org/docs/guide-user/base-system/dhcp_configuration#dhcp_options)，还是不生效！

```sh
uci add_list dhcp.lan.dhcp_option="3,10.0.0.10" # gateway
uci add_list dhcp.lan.dhcp_option="6,10.0.0.10" # dns
uci commit dhcp
/etc/init.d/dnsmasq restart
```

怒了！之前自己 Debian 也没碰上这种破事啊！通过查进程指令翻临时 dnsmasq.conf 看到底缺了啥。发现没有强制性配置，改！这次终于生效了。

```sh
sed -i 's/ dhcp_option / dhcp_option_force /g' /etc/config/dhcp
uci set dhcp.lan.force="1"
uci commit dhcp
/etc/init.d/dnsmasq restart
```

## 三 改源

使用[中科大](http://mirrors.ustc.edu.cn)的镜像源，要不然那网速，真是让人欲仙欲死！

```sh
sed -i 's/downloads.openwrt.org/mirrors.ustc.edu.cn\/lede/g' /etc/opkg/distfeeds.conf
```

### 四 搭桥

#### a. 准备环境

回 X3421 中用 Alpine 做一个 [OpenWRT-SDK](https://openwrt.org/docs/guide-developer/using_the_sdk) 环境。

```sh
wget https://downloads.openwrt.org/releases/19.07.3/targets/x86/64/openwrt-sdk-19.07.3-x86-64_gcc-7.5.0_musl.Linux-x86_64.tar.xz
tar -xf openwrt-sdk-19.07.3-x86-64_gcc-7.5.0_musl.Linux-x86_64.tar.xz
docker run --name openwrt-sdk -v ~:/mnt/shared -w /root -it alpine sh
```

创建普通用户以满足编译要求：

```sh
# docker container
mv /mnt/shared/openwrt-sdk-19.07.3-x86-64_gcc-7.5.0_musl.Linux-x86_64 /sdk
adduser -h /sdk -G users -D snakevil
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
apk update
apk add bash sudo
adduser snakevil wheel
visudo
exit
```

切换成普通用户，

```sh
docker start openwrt-sdk
docker exec -it -u snakevil -w /sdk openwrt-sdk bash
```

并按照[官方指南](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)安装必要包以完成编译环境。

```sh
sudo apk add asciidoc bash bc binutils bzip2 cdrkit coreutils diffutils findutils flex g++ gawk gcc gettext git grep intltool libxslt linux-headers make ncurses-dev patch perl python2-dev python3-dev tar unzip  util-linux wget zlib-dev
make prereq
./scripts/feeds update -a
./scripts/feeds install -a
```

#### b. shadowsocks-libev

先解决无法使用搜索引擎的问题。果断先从 `luci-app-shadowsocks-libev` 开始入手。安装之后刷新一下，就多出了“Services”大类。

在配置 `ss-redir` 实例时，OpenWRT 会引导我们安装 `shadowsocks-libev-ss-redir` 和 `shadowsocks-libev-ss-rules`。

做好配置，结果无法成功启动，系统日志里也看不到任何有用信息。仔细检查，发现没有指定端口。但是 GUI 也并没有说这项必填。先随便填了一个试试，结果就好了。低级错误。

再配置 `ss-tunnel` 实例，GUI 会引导我们安装 `shadowsocks-libev-ss-tunnel`。所有的实例都不能同名，否则会出现各种稀奇古怪的问题。

使用同样的服务器，但本地地址 `127.0.0.1`，模式也只限 UDP。运行起来先测试跑通没有：

```sh
dig www.google.com @127.0.0.1 -p 5300
```

#### c. 智能 DNS

找了一圈觉得还是 [pexcn/openwrt-chinadns-ng](https://github.com/pexcn/openwrt-chinadns-ng) 最合适。根据 gfwlist 或 chnlist 分路查询国内外地址。国内求快，国外求准。

```sh
git clone https://github.com/pexcn/openwrt- chinadns-ng.@chinadns-ng[0].git package/chinadns-ng
make menuconfig
make package/chinadns-ng/{clean,compile} V=s
cp bin/packages/x86_64/base/chinadns-ng_1.0-beta.22-4_x86_64.ipk /mnt/shared/
exit
```

这个步骤各种报错，各种龟速下载不知道些啥。结果许久之后我再回来看，居然还成了。安装配置：

```sh
scp chinadns-ng_1.0-beta.22-4_x86_64.ipk root@gateway:
ssh root@gateway
```

```sh
opkg install chinadns-ng_1.0-beta.22-4_x86_64.ipk
uci set chinadns-ng.@chinadns-ng[0].enable="1"
uci set chinadns-ng.@chinadns-ng[0].bind_addr="127.0.0.1"
uci set chinadns-ng.@chinadns-ng[0].repeat_times="1"
uci commit chinadns-ng
```

调整 DHCP 配置，将所有 DNS 查询全部转发到 `127.0.0.1#5353`，并关掉 `resolve.conf`。大功告成。

```sh
dig www.google.com
```

#### d. 智能线路

同样找了一圈，同样没有找到现成的 luci-app，但 [cokebar/gfwlist2dnsmasq](https://github.com/cokebar/gfwlist2dnsmasq) 足以解决问题了。

首先是将默认的 dnsmasq 更换成 dnsmasq-full 来支持 ipset。

```sh
opkg remove dnsmasq
opkg install dnsmasq-full
```

然后调整 dnsmasq 配置以支持自定义配置文件——没办法在 luci 的 DHCP 模块中操作，很郁闷。

```sh
echo 'conf-dir=/etc/dnsmasq.d' >> /etc/dnsmasq.conf
mkdir /etc/dnsmasq.d
```

然后生成配置并手动重启 dnsmasq 即可。

```sh
wget -O gfwlist2dnsmasq.sh https://raw.githubusercontent.com/cokebar/gfwlist2dnsmasq/master/gfwlist2dnsmasq.sh
opkg install coreutils-mktemp coreutils-base64
sh gfwlist2dnsmasq.sh -s ss_rules_dst_forward -o /etc/dnsmasq.d/gfwlist.conf
/etc/init.d/dnsmasq restart
```

## 五 扩容

暂时没啥需求想法。懒得折腾了，后面再说。
