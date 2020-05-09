---
title: "OpenWrt BarrierBreaker 14.07-rc3 手记"
slug: "OpenWrt-BarrierBreaker_14_07_rc3手记"
date: 2014-09-18T05:59:59Z
lastmod: 2014-09-23T04:11:59Z
toc: true
category:
    - 家庭组网
tag:
    - OpenWRT
    - Linux
    - MiniUPnPd
    - Dnsmasq
    - ipset
    - LigHTTPd
    - MiniDLNA
    - Aria2
---

[OpenWRT][] 新的大版本发布了第三个候选包，据说 Bug 收敛得很不错，几乎不影响正常使用了。正好我也期盼已久，果断给路由升级！

<!--more-->

## 1 外置存储 - ExtRoot

### 1.1 USB HDD 分区

##### 安装依赖和工具包：

```sh
opkg install kmod-usb-storage kmod-fs-ext4 libext2fs block-mount fdisk e2fsprogs

reboot
```

##### 挂载 USB HDD ，使用 `fdisk /dev/sda` 分成两个数据区：

1. 分区 /dev/sda1 大小 1G ，用于保存系统数据；
2. 分区 /dev/sda2 为剩余大小，用于保存用户数据。

##### 使用 ext4 文件系统格式化两个分区：

（USB HDD 大小超过 21G 时，对 dev/sda2 格式化操作才需要添加 `-m 1` 操作。）

```sh
mkfs.ext4 -L overlay /dev/sda1
mkfs.ext4 -b 4096 -L data -m 1 /dev/sda2
```

重置路由至初始（出厂）状态。

### 1.2 挂载 USB HDD

##### 安装依赖包并启动 fstab ：

```sh
opkg install kmod-usb-storage kmod-fs-ext4 libext2fs block-mount

/etc/init.d/fstab enable
```

检查分区 /dev/sda1 是否存在，若不存在，`reboot` 一次。

##### 挂载系统分区 /dev/sda1 ：

```sh
uci add fstab mount
uci set fstab.@mount[-1].enabled=1
uci set fstab.@mount[-1].target=/overlay
uci set fstab.@mount[-1].device=/dev/sda1
uci set fstab.@mount[-1].fstype=ext4
uci commit

mount -t ext4 /dev/sda1 /mnt
cp -ar /overlay/* /mnt/
```

`reboot` 之后，`df -h` 检查是否挂载成功。成功时命令输出结果应包含下行内容（忽略容量和使用占比）：

> ```
> /dev/sda1               975.9M    140.6M    768.1M  15% /overlay
> ```

##### 挂载 swap 虚拟内存：

（一般为内存大小两倍。）

```sh
dd if=/dev/zero of=/swap bs=1M count=128
mkswap /swap

uci add fstab swap
uci set fstab.@swap[-1].enabled=1
uci set fstab.@swap[-1].device=/swap
uci commit
```

##### 挂载数据分区 /dev/sda2 ：

```sh
mkdir /home

uci add fstab mount
uci set fstab.@mount[-1].enabled=1
uci set fstab.@mount[-1].target=/home
uci set fstab.@mount[-1].device=/dev/sda2
uci set fstab.@mount[-1].fstype=ext4
uci commit
```

##### 确认挂载成功：

`reboot` 之后，`free` 检查 swap 是否挂载成功。成功时命令输出结果应包含下行内容（忽略使用大小）：

> ```
> Swap:       131068            0       131068
> ```

`df -h` 检查数据分区是否挂载成功。成功时命令输出结果应包含下行内容（忽略容量和使用占比）：

> ```
> /dev/sda2               824.7G     72.1M    816.3G   0% /home
> ```

##### 迁移 root 家目录：

```sh
cd /
chmod 0700 root
mv root home/
ln -s home/root
```

## 2 局域网优化 - DMZ + UPnP

### 2.1 LAN

##### 调整 MTU ：

```sh
uci set network.lan.mtu=1492
uci commit
```

##### 调整网段（略）

### 2.2 WAN

##### 调整为静态地址（192.168.1.234）：

```sh
uci set network.wan.proto=static
uci set network.wan.ipaddr=192.168.1.234
uci set network.wan.netmask=255.255.255.0
uci set network.wan.gateway=192.168.1.1
uci set network.wan.dns=114.114.115.115
uci commit
```

##### 调整 MTU ：

```sh
uci set network.wan.mtu=1492
uci commit
```

##### 生效：

```sh
/etc/init.d/network restart
```

### 2.3 配置 DMZ （需光猫高级管理员权限）：

在光猫中配置 DMZ 主机至 192.168.1.234 。

### 2.4 配置 UPnP ：

```sh
opkg install luci-app-upnp

/etc/init.d/miniupnpd enable
/etc/init.d/miniupnpd start
```

## 3 科学上网 - dnsmasq + ipset + shadowsocks

### 3.1 配置 shadowsocks-libev

想办法从 [shadowsocks 官网](http://shadowsocks.org/en/download/clients.html) 下载对应芯片的 [OpenWRT][] 包。

##### 安装：

```sh
opkg install shadowsocks-libev-polarssl_1.4.6_ar71xx.ipk
rm -f shadowsocks-libev-polarssl_1.4.6_ar71xx.ipk

sed -e 's/ss-local/ss-redir/' /etc/init.d/shadowsocks > shadowsocks
mv -f shadowsocks /etc/init.d/shadowsocks

/etc/init.d/shadowsocks enable
```

##### `vi /etc/shadowsocks.json` 配置后启动（本地端口 1080）：

```sh
/etc/init.d/shadowsocks start
```

### 3.2 配置 dnsmasq-full

##### 安装：

```sh
opkg install dnsmasq-full

mkdir /etc/dnsmasq.d
echo conf-dir=/etc/dnsmasq.d > /etc/dnsmasq.conf

cat > /etc/dnsmasq.d/ipset.shadowsocks.org <<END
server=/.shadowsocks.org/208.67.220.220#443
ipset=/.shadowsocks.org/shadow
END
```

### 3.3 配置 ipset

##### 安装（其中 1080 为 shadowsocks 本地端口）：

```sh
opkg install iptables-mod-nat-extra ipset

cat > /etc/rc.local <<END
/usr/sbin/ipset -N shadow iphash

exit 0
END
cat > /etc/firewall.user <<END
/usr/sbin/iptables -t nat -A prerouting_rule -p tcp -m set --match-set shadow dst -j REDIRECT --to-port 1080
END

sh /etc/rc.local
sh /etc/firewall.user
```

### 3.4 google.com （示例）

##### 配置：

```sh
cat > /etc/dnsmasq.d/ipset.google.com <<END
server=/.google.com/208.67.220.220#443
server=/.google.com.hk/208.67.220.220#443
server=/.google.co.jp/208.67.220.220#443
server=/.gstatic.com/208.67.220.220#443
server=/.ggpht.com/208.67.220.220#443
server=/.googleusercontent.com/208.67.220.220#443
server=/.googlecode.com/208.67.220.220#443
server=/.googleapis.com/208.67.220.220#443
server=/.gmail.com/208.67.220.220#443
server=/.google-analytics.com/208.67.220.220#443
server=/.googlevideo.com/208.67.220.220#443

ipset=/.google.com/shadow
ipset=/.google.com.hk/shadow
ipset=/.google.co.jp/shadow
ipset=/.gstatic.com/shadow
ipset=/.ggpht.com/shadow
ipset=/.googleusercontent.com/shadow
ipset=/.googlecode.com/shadow
ipset=/.googleapis.com/shadow
ipset=/.gmail.com/shadow
ipset=/.google-analytics.com/shadow
ipset=/.googlevideo.com/shadow

server=/.youtube.com/208.67.220.220#443
server=/.youtube-nocookie.com/208.67.220.220#443
server=/.ytimg.com/208.67.220.220#443

ipset=/.youtube.com/shadow
ipset=/.youtube-nocookie.com/shadow
ipset=/.ytimg.com/shadow

server=/.appspot.com/208.67.220.220#443
server=/.blogspot.com/208.67.220.220#443
server=/.blogger.com/208.67.220.220#443

ipset=/.appspot.com/shadow
ipset=/.blogspot.com/shadow
ipset=/.blogger.com/shadow
END
```

##### 生效：

```sh
/etc/init.d/dnsmasq restart
```

## 4 网络存储 - lighttpd （WebDAV 暂无法实现）

### 4.1 替代 uhttpd

```sh
/etc/init.d/uhttpd disable
/etc/init.d/uhttpd stop

opkg install lighttpd-mod-cgi

mv /etc/lighttpd/lighttpd.conf /etc/lighttpd/lighttpd.conf.opkg
cat > /etc/lighttpd/lighttpd.conf <<END
index-file.names = ( "index.html" )
mimetype.assign = (
  ".html"  => "text/html",
  ".htm"   => "text/html",
  ".css"   => "text/css",
  ".js"    => "text/javascript",
  ".json"  => "application/json",
  ".xml"   => "text/xml",
  ".dtd"   => "text/xml",
  ".txt"   => "text/plain",
  ".bmp"   => "image/x-ms-bmp",
  ".gif"   => "image/gif",
  ".ico"   => "image/x-icon",
  ".jpeg"  => "image/jpeg",
  ".jpg"   => "image/jpeg",
  ".png"   => "image/png",
  ".svg"   => "image/svg+xml",
  ".svgz"  => "image/svg+xml",
  ".tif"   => "image/tiff",
  ".tiff"  => "image/tiff",
  ".3gp"   => "video/3gpp",
  ".3gpp"  => "video/3gpp",
  ".asf"   => "video/x-ms-asf",
  ".asx"   => "video/x-ms-asf",
  ".avi"   => "video/x-msvideo",
  ".flv"   => "video/x-flv",
  ".m4v"   => "video/mp4",
  ".mng"   => "video/x-mng",
  ".mkv"   => "video/x-matroska",
  ".mov"   => "video/quicktime",
  ".mp4"   => "video/mp4",
  ".mpe"   => "video/mpeg",
  ".mpeg"  => "video/mpeg",
  ".mpg"   => "video/mpeg",
  ".ogv"   => "video/ogg",
  ".webm"  => "video/webm",
  ".wmv"   => "video/x-ms-wmv",
  ".kar"   => "audio/midi",
  ".mid"   => "audio/midi",
  ".midi"  => "audio/midi",
  ".m4a"   => "audio/mpeg",
  ".mp2"   => "audio/mpeg",
  ".mp3"   => "audio/mpeg",
  ".mpga"  => "audio/mpeg",
  ".mpega" => "audio/mpeg",
  ".oga"   => "audio/ogg",
  ".ogg"   => "audio/ogg",
  ".ra"    => "audio/x-realaudio",
  ".spx"   => "audio/ogg",
  ".wav"   => "audio/x-wav",
  ".weba"  => "audio/webm",
  ".ai"    => "application/postscript",
  ".doc"   => "application/msword",
  ".docx"  => "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
  ".eps"   => "application/postscript",
  ".ogx"   => "application/ogg",
  ".pac"   => "application/x-ns-proxy-autoconfig",
  ".pdf"   => "application/pdf",
  ".ppt"   => "application/vnd.ms-powerpoint",
  ".pptx"  => "application/vnd.openxmlformats-officedocument.presentationml.presentation",
  ".ps"    => "application/postscript",
  ".psd"   => "application/photoshop",
  ".swf"   => "application/x-shockwave-flash",
  ".xls"   => "application/vnd.ms-excel",
  ".xlsx"  => "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
  ".7z"    => "application/x-7z-compressed",
  ".bz2"   => "application/bzip2",
  ".dmg"   => "application/x-apple-diskimage",
  ".gz"    => "application/gzip",
  ".tar"   => "application/tar",
  ".xz"    => "application/x-xz",
  ".zip"   => "application/zip",
  ""       => "application/octet-stream"
)
server.document-root = "/www"
server.errorlog-use-syslog = "enable"
server.max-connections = 16
server.modules = ()
server.network-backend = "write"
server.pid-file = "/var/run/lighttpd.pid"
server.upload-dirs = ( "/tmp" )
static-file.exclude-extensions = ()
cgi.execute-x-only = "enable"
cgi.assign = ()
include "luci.conf"
END
cat > /etc/lighttpd/luci.conf <<END
server.modules += ( "mod_cgi" )

\$HTTP["url"] =~ "^/cgi-bin/luci($|/)" {
  cgi.assign += (
    "cgi-bin/luci" => ""
  )
}
END

/etc/init.d/lighttpd enable
/etc/init.d/lighttpd start
```

### 4.2 支持 WebDAV

（Lighttpd 对 WebDAV 最新规范地实现度也不足 100% ，以后再找找有没有更好的办法…）

## 5 家庭多媒体共享 - minidlna + pure-ftpd

### 5.1 安装 minidlna

```sh
opkg install minidlna

mkdir -p /home/dlna/data
mkdir /home/dlna/db

uci set minidlna.config.db_dir=/home/dlna/db
uci set minidlna.config.log_dir=/home/dlna
uci delete minidlna.config.media_dir
uci add_list minidlna.config.media_dir=/home/dlna/data
uci commit

/etc/init.d/minidlna enable
/etc/init.d/minidlna start
```

### 5.2 安装 pure-ftpd

```sh
opkg install pure-ftpd

uci set pure-ftpd.@pure-ftpd[-1].chrooteveryone=0
uci set pure-ftpd.@pure-ftpd[-1].enabled=1
uci commit

/etc/init.d/pure-ftpd enable
/etc/init.d/pure-ftpd start
```

## 6 脱机下载 - aria2

### 6.1 安装 aria2

官方源中的版本阉割掉了 BitTorrent 下载功能 :-( 所以得从 [南浦月](http://blog.nanpuyue.com/) 的帖子 [《luci-app-aria2、yaaw、aria2-1.18.3 下载》](http://www.right.com.cn/forum/thread-135833-1-1.html) 下载相应芯片的包。

（BT 下载速度非常差，原因有待深入研究…）

```sh
opkg install aria2_1.18.3-1_ar71xx.ipk

mkdir -p /home/aria/data
touch /home/aria/session
touch /home/aria/dht

cat > /etc/aria2.conf <<END
log=/home/aria/info.log
dir=/home/aria/data
file-allocation=trunc
continue=true
input-file=/home/aria/session
deferred-input=true
max-concurrent-downloads=10
quiet=false
log-level=error
remote-time=true
enable-rpc=true
rpc-listen-all=true
rpc-allow-origin-all=true
rpc-save-upload-metadata=true
disable-ipv6=true
save-session=/home/aria/session
max-connection-per-server=5
min-split-size=5M
max-download-result=100
retry-wait=15
disk-cache=64M
save-session-interval=60
user-agent=uTorrent/341(109279400)(30888)
max-overall-upload-limit=16K
listen-port=6801-6899
follow-torrent=true
seed-time=1440
seed-ratio=0
peer-id-prefix=-UT341-
enable-peer-exchange=true
enable-dht=true
dht-listen-port=6801-6899
bt-seed-unverified=true
bt-max-peers=96
bt-save-metadata=true
bt-enable-lpd=true
END
cat > bt-tracker <<END
bt-tracker=
http://anisaishuu.de:2710/announce,
http://bigfoot1942.sektori.org:6969/announce,
http://bt.careland.com.cn:6969/announce,
http://exodus.desync.com/announce,
http://exodus.desync.com:6969/announce,
http://hdreactor.org:2710/announce,
http://i.bandito.org/announce,
http://shadowshq.yi.org:6969/announce.php,
http://tracker.ex.ua/announce,
http://tracker.nwps.ws:6969/announce,
http://tracker.trackerfix.com:80/announce,
http://tracker2.torrentino.com/announce,
http://tracker3.torrentino.com/announce
END
awk 'BEGIN{ORS=""}1;END{print"\n"}' bt-tracker >> /etc/aria2.conf
rm -f bt-tracker
cat > /etc/init.d/aria2 <<END
#!/bin/sh /etc/rc.common

START=95

SERVICE_USE_PID=1
SERVICE_WRITE_PID=1
SERVICE_DAEMONIZE=1

start() {
    service_start /usr/bin/aria2c --conf-path=/etc/aria2.conf
}

stop() {
    service_stop /usr/bin/aria2c
}
END

/etc/init.d/aria2 enable
/etc/init.d/aria2 start

uci add firewall rule
uci set firewall.@rule[-1].name=Allow-Aria2-RPC
uci set firewall.@rule[-1].src=wan
uci set firewall.@rule[-1].proto=tcp
uci set firewall.@rule[-1].dest_port=6800
uci set firewall.@rule[-1].target=ACCEPT
uci add firewall rule
uci set firewall.@rule[-1].name=Allow-Aria2-BT
uci set firewall.@rule[-1].src=wan
uci set firewall.@rule[-1].proto="tcp udp"
uci set firewall.@rule[-1].dest_port=6801-6899
uci set firewall.@rule[-1].target=ACCEPT
uci commit

/etc/init.d/firewall restart
```

### 6.2 安装 WebUI

从 [yaaw 项目官网](https://github.com/binux/yaaw) 下载 zip 包并上传至路由器。

```sh
opkg install unzip

cd /home/aria
mv /root/yaaw-master.zip .
unzip yaaw-master.zip
rm -f yaaw-master.zip
mv yaaw-master webui
cd /www
ln -s /home/aria/webui aria
```

## 9 远程控制 - DDNS

### 9.1 3322

##### 配置计划任务（USER、PASSWORD、DOMAIN 自填）：

```sh
echo "* * * * * /usr/bin/wget -qO - 'http://<USER>:<PASSWORD>@members.3322.net/dyndns/update?hostname=<DOMAIN>' > /dev/null 2>&1" > /etc/crontabs/root
```

### 9.2 外部 SSH 访问

```sh
uci add firewall rule
uci set firewall.@rule[-1].name=Allow-SSH
uci set firewall.@rule[-1].proto=tcp
uci set firewall.@rule[-1].src=wan
uci set firewall.@rule[-1].dest_port=22
uci set firewall.@rule[-1].target=ACCEPT
uci commit
```

### 9.3 外部 HTTP 访问

（光猫会占很不智能地占用 80 端口…所以采用 86 端口。）

```sh
echo '/usr/sbin/iptables -t nat -A prerouting_rule -p tcp -d 192.168.1.234 --dport 86 -j REDIRECT --to-port 80' >> /etc/firewall.user

uci add firewall rule
uci set firewall.@rule[-1].name=Allow-HTTP
uci set firewall.@rule[-1].src=wan
uci set firewall.@rule[-1].proto=tcp
uci set firewall.@rule[-1].dest_port=86
uci set firewall.@rule[-1].target=ACCEPT
uci commit

/etc/init.d/firewall restart
```
