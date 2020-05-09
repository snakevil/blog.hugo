---
title: "PS4 网络环境优化"
slug: "PS4网络环境优化"
date: 2014-08-26T08:59:12Z
draft: true
category:
    - 家庭组网
tag:
    - OpenWRT
    - Linux
    - NAT
    - iptables
    - MiniUPnPd
---

我要被中国电信家用宽带的超大局域网结构快逼疯了！ PS4 联机动不动就连不上！尝试优化一下 NAT 类型，看会不会好些。

<!--more-->

## 0 案例环境

-   中国电信 100M 宽带接入，局域网段 `192.168.1/24`，光猫拥有管理员权限（后继需要）；
-   NetGear WNDR3700v2 跑 OpenWRT 作为家庭主路由器， WAN IP `192.168.1.234`，局域网段 `10.10.10/24`；
-   PS4 无线连家庭网络（主），IP `10.10.10.3`；有线连光猫（备），IP `192.168.1.3`。

## 1 NAT 类型

在 PS4（PS3） 中定义了三种类型的 NAT ：

1. 直连网络！
1. 通过路由中继但测试通过。
1. 同样通过路由中继，但无法完成 P2P 链接，导致联机、语音或其它联网功能故障…

前两者不影响实际使用。但第三种情况很要命！

### 1.1 家庭网络通道

#### 光猫

开启家庭路由器 `192.168.1.234` 为 DMZ 主机。

#### 路由

映射以下端口至 PS4 `10.10.10.3` ：

-   80
-   443
-   1935
-   3478-3480

在案例环境中，终端 SSH 到路由后执行：

```sh
iptables -t nat -A PREROUTING -p tcp -i eth1 --dport 80 -j DNAT --to-destination 10.10.10.3
iptables -t nat -A PREROUTING -p udp -i eth1 --dport 80 -j DNAT --to-destination 10.10.10.3
iptables -t nat -A PREROUTING -p tcp -i eth1 --dport 443 -j DNAT --to-destination 10.10.10.3
iptables -t nat -A PREROUTING -p udp -i eth1 --dport 443 -j DNAT --to-destination 10.10.10.3
iptables -t nat -A PREROUTING -p tcp -i eth1 --dport 1935 -j DNAT --to-destination 10.10.10.3
iptables -t nat -A PREROUTING -p udp -i eth1 --dport 1935 -j DNAT --to-destination 10.10.10.3
iptables -t nat -A PREROUTING -p tcp -i eth1 --dport 3478:3480 -j DNAT --to-destination 10.10.10.3
iptables -t nat -A PREROUTING -p udp -i eth1 --dport 3478:3480 -j DNAT --to-destination 10.10.10.3
```

### 1.2 光猫直连通道

在配置家庭网络通道之后，建议放弃光猫直连通道的配置。原因在于无法确定光猫对 DMZ 和 端口映射 机制的技术实现原理，可能会与之前的配置冲突。

## 2 ALG & UPnP

#### 光猫

启用所有的 ALG 模块，开启 UPnP 。

#### 路由

与光猫操作一致。

在案例环境中，终端 SSH 到路由后执行：

```sh
opkg update
opkg install miniupnpd luci-app-upnp
/etc/init.d/miniupnpd enable
/etc/init.d/miniupnpd start
```

## 3 MTU

使用另外一台同样在家庭网络内的设备探测有效 MTU 值（OS X 及 Linux 有效，Windows 请自行 [Google](https://www.google.com)）：

```sh
ping -s <SIZE> www.baidu.com
```

能够成功返回结果的最大数值，再额外增加 28 ，就是有效 MTU 。有效 MTU 对于拨号网络而言，一般都是 1492 。

#### 路由

设置所有网卡（包括 WAN 口和 LAN 口）的 MTU 为上述探测地有效 MTU 。

在案例环境中，终端 SSH 到路由后执行：

```sh
uci set network.wan.mtu=1492
uci set network.lan.mtu=1492
uci commit
/etc/init.d/network restart
```

#### PS4

在网络配置中，同样将 MTU 设置为上述探测地有效 MTU 。

## 4 DNS

#### PS4

在网络配置中，将 DNS 调整至：

-   168.126.63.2
-   168.126.63.1

## A 参考文献

1. [《NAT Type 2 Tutorial》](http://community.us.playstation.com/t5/PlayStation-General/NAT-Type-2-Tutorial/td-p/27538324)，作者 [CheckMatey](http://community.us.playstation.com/t5/user/viewprofilepage/user-id/304820)，最后修订于 2008 年 6 月 19 日。
2. [《Port Forwarding the Netgear WNDR3700 Router for PS4》](http://portforward.com/english/routers/port_forwarding/Netgear/WNDR3700/PS4.htm)
3. [《OpenWRT: UPnP》](http://advanxer.com/blog/2012/06/openwrt-upnp/)
4. [《PS4 加速下载 和番茄方法 解决登录 PSN 新增免费服务器》](http://bbs.duowan.com/thread-37923399-1-1.html)，作者 [op2009](http://www.psncn.com/?p=11)，最后修订于 2014 年 7 月 19 日。
