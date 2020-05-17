---
title: "【译】Plex vs. Emby vs. Jellyfin vs. Kodi——2020 年深度比较"
slug: "Plex-vs-Emby-vs-Jellyfin-vs-Kodi-2020年深度比较"
date: 2020-05-17T11:59:47+08:00
toc: true
category:
    - 家庭组网
tag:
    - Plex
    - Emby
    - Jellyfin
    - Kodi
    - 自主翻译
---

[点击这里阅读原文。](https://www.smarthomebeginner.com/plex-vs-emby-kodi-jellyfin-2020/)

---

Plex 和 Emby，应该选哪个？或许 Kodi 更好？新人 Jellyfin 怎么样？本篇完善后的指南介绍了 Plex 和 Emby 之间的差异，并与 Kodi 做比较，以帮助您做出选择。

[plex]: https://www.plex.tv
[emby]: https://emby.media
[kodi]: https://kodi.tv
[jellyfin]: https://jellyfin.org

<!--more-->

[媒体服务器软件的选择](https://www.smarthomebeginner.com/media-server-software-options-2017/)很多。Plex 是 Emby 是最常见的选择。

如果您像我一样，那您已经花费了大量的时间来思考和权衡 [Plex]() 与 [Emby]()（以前叫 Media Browser），希望能决定您的媒体服务器究竟用啥。

## Plex 和 Emby 是什么？

> Plex 和 Emby 是两种最常见的媒体服务器方案，即将媒体在内网或互联网上流式传输到不同设备。此外，Plex 还提供基于广告的免费电影、新闻和其它内容。可以将其想象成您独有的随时随地可用的 Netflix，而且还可以共享给家人和朋友。但两者都需要付费才能使用某些功能。

您可能已经问过自己，Emby 是否就比 Plex 更好？

要从太多的信息中迅速列出 Emby 和 Plex 之间全部的注意事项的不同是一件很困难的事情。而且从 2019 年以后 Plex 的战略还发生了许多改变，而这也是必须要考量的点。

因此，本文会尝试总结 Plex 和 Emby 之间的异同。向您介绍一些值得考量的关键要素。并努力对它们评分！

## Plex 和 Emby 如何工作？

Kodi 是一个独立应用程序。这意味着它可以装在任何系统中，也深受系统影响瓶颈的影响。客户端必须足够强力才能解码播放本地的和远端的视频。因此远端服务器只通过目录共享协议（Samba 或 NFS）提供文件本身。

> **从 Kodi 开始入门？可以先嘟嘟这些新手指南！**
>
> 1. Kodi 新手指南系列：第一篇[《Kodi 是什么？》](https://www.smarthomebeginner.com/kodi-guide-what-is-kodi/)，第二篇[《Kodi 的用途》](https://www.smarthomebeginner.com/kodi-beginners-guide-how-to-use-kodi/)，第三篇[《添加媒体》](https://www.smarthomebeginner.com/add-media-sources-to-kodi/)，第四篇[《改变外观》](https://www.smarthomebeginner.com/change-kodi-skin-guide/)和第五篇[《文件夹结构》](https://www.smarthomebeginner.com/kodi-folder-location-and-structure/)。
> 2. 《理解 Kodi 设置》：[音频](https://www.smarthomebeginner.com/kodi-audio-settings/)和[视频](https://www.smarthomebeginner.com/kodi-video-settings/)。
> 3. 新手蓝图：[《Kodi 完整安装指南》](https://www.smarthomebeginner.com/blueprint-kodi-setup-guide/)。
> 4. 新手蓝图：[《亚马逊 FireTV + Kodi 完整指南》](https://www.smarthomebeginner.com/blueprint-amazon-fire-tv-kodi-guide/)。

另外一边的 Plex、Emby 和 Jellyfin 都遵循着“服务端—客户端”模型。服务端版本的应用程序运行在服务器上，对全部媒体进行分类，并分享给您自己或您分享过的其他人。客户端应用程序则连接服务端并播放媒体。尽管客户端功能强大会更好，但强大的服务端也能弥补较弱的客户端。中央式的库管理决定了所有客户端都需要与服务端保持同步。（参考阅读：[《多设备串流所需的 10 种媒体服务器软件方案》](https://www.smarthomebeginner.com/media-server-software-options-2017/)）

## Plex 与 Emby 对比表（也额外包含 Kodi）

我们整理了一个 Plex 与 Emby 比较表，其中详细列出了差异和相似之处，各自的优缺点。 我们采用了一些最受欢迎的考虑因素，并对不同的媒体中心分别进行了评论和评价。

这是基于我们之前完成的[Kodi 与 Plex 的比较](https://www.smarthomebeginner.com/plex-vs-kodi-comparison-guide/)的补充，我们还比较了 Plex、Emby 和 Kodi！ 关于此表，请注意几点：

1. 对于 Kodi，我们使用的是独立应用程序版本（因此无法享受其作为 Plex 插件或 Emby 插件被各服务端集成时的好处）
2. 某些行没有计分是因为那些行为毫无意义
3. 我们试图从初学者的视角来对每个应用程序打分
4. 无论如何评分都会有些主观

我们很乐意在下方的评论中聆听您的想法（或您的评分），这些评论对其他正在考虑该使用何种媒体中心的用户也会提供帮助。

| 功能                                                  | Kodi（独立）                                                                                                              | Plex 媒体服务器                                                                                                                                                                                              | Emby 服务器                                                                                                                                             | Jellyfin                                                                                |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| [支持的服务端平台](#1-支持的服务端平台)               | 不适用                                                                                                                    | ![5](/rating-10.png) 多操作系统、家用游戏机、NAS、路由器和 Docker                                                                                                                                            | ![4](/rating-8.png) 多操作系统、家用游戏机、NAS、路由器和 Docker                                                                                        | ![3](/rating-6.png) 多操作系统、部分 NAS 和 Docker                                      |
| [支持的客户端平台](#2-支持的客户端平台)               | ![4.5](/rating-9.png) 很多                                                                                                | ![5](/rating-10.png) [比很多还多](https://www.smarthomebeginner.com/best-plex-client-devices-2018/)（部分平台需要付费，[详见下文](#6-Emby首映式和Plex套票)）                                                 | ![4.5](/rating-9.png) [很多](https://www.smarthomebeginner.com/best-emby-client-devices-2018/)（部分平台需要付费，[详见下文](#6-Emby首映式和Plex套票)） | ![3](/rating-6.png) 寥寥                                                                |
| [安装与使用的简易度](#3-安装和使用的简易度)           | ![3.5](/rating-7.png) 中度复杂                                                                                            | ![5](/rating-10.png) 非常简单                                                                                                                                                                                | ![4](/rating-8.png) 简单                                                                                                                                | ![4](/rating-8.png) 简单                                                                |
| [可定制性](#4-可定制性)                               | ![5](/rating-10.png) 非常灵活                                                                                             | ![2](/rating-4.png) 灵活性受限                                                                                                                                                                               | ![3](/rating-6.png) 灵活的选择                                                                                                                          | ![4](/rating-8.png) 灵活的选择且开源                                                    |
| [本地和远端串流](#5-本地和远端串流)                   | ![3](/rating-6.png) 本地串流没有限制，远端串流很困难                                                                      | ![4](/rating-8.png) 本地和远端串流都容易（但可能收费）                                                                                                                                                       | ![4](/rating-8.png) 本地和远端串流都容易（但可能收费）                                                                                                  | ![5](/rating-10.png) 本地和远端串流容易且免费                                           |
| [Plex 套票和 Emby 首映式](#6-plex-套票和-emby-首映式) | 不适用                                                                                                                    | ![3](/rating-6.png) 没有 Plex 套票也能活                                                                                                                                                                     | ![4](/rating-8.png) Emby 首映式是增值服务                                                                                                               | 不适用                                                                                  |
| [TV、IPTV 和 DVR 直播](#7-电视直播和-dvr-支持)        | ![5](/rating-10.png) 通过插件完美实现                                                                                     | ![3](/rating-6.png) 对 OTA 调谐器和 DVR（需套票）支持良好。不支持 IPTV                                                                                                                                       | ![3.5](/rating-7.png) 支持 OTA 和 DVR（需首映式，限 HDHomeRun 和 Haupauge）。对 IPTV 支持很好                                                           | ![4](/rating-8.png) 支持 OTA 和 DVR（限 HDHomeRun 和 Haupauge）。对 IPTV 支持很好       |
| [元信息管理](#8-数据库管理元信息管理器)               | ![2](/rating-4.png) 客户端界面                                                                                            | ![4](/rating-8.png) 网页界面                                                                                                                                                                                 | ![5](/rating-10.png) 网页界面                                                                                                                           | ![4](/rating-8.png) 网页界面                                                            |
| 多客户端的中央数据库                                  | ![2](/rating-4.png) [使用 MySQL 的高级安装](https://www.smarthomebeginner.com/kodi-mysql-setup-to-share-library/)可能支持 | ![4.5](/rating-9.png) 是                                                                                                                                                                                     | ![5](/rating-10.png) 是                                                                                                                                 | ![4.5](/rating-9.png) 是                                                                |
| [免费内容](#9-免费内容)                               | ![3](/rating-6.png) 插件里有很多                                                                                          | ![4](/rating-8.png) 带广告的电影、电视、新闻和广播                                                                                                                                                           | ![1](/rating-2.png) 几乎没有                                                                                                                            | ![1](/rating-2.png) 几乎没有                                                            |
| [插件/频道](#10-插件亦称频道支持)                     | ![5](/rating-10.png) [很多](https://www.smarthomebeginner.com/best-addons-for-kodi-18-leia-2019/)                         | ![0.5](/rating-1.png) 无                                                                                                                                                                                     | ![3](/rating-6.png) 寥寥                                                                                                                                | ![2](/rating-4.png) 寥寥                                                                |
| [离线浏览（无互联网）](#11-离线浏览同步)              | 不适用                                                                                                                    | ![5](/rating-10.png) 良好支持                                                                                                                                                                                | ![4.5](/rating-9.png) 良好支持                                                                                                                          | ![1.5](/rating-3.png) 只能通过网页应用程序下载                                          |
| [用户管理](#12-用户管理)                              | ![5](/rating-10.png) 本地化且很多可控                                                                                     | ![3](/rating-6.png) 只能管理线上账号                                                                                                                                                                         | ![5](/rating-10.png) 本地、线上（可选）且很多可控                                                                                                       | ![5](/rating-10.png) 本地且很多可控                                                     |
| 库共享                                                | ![1](/rating-2.png) 无法便捷共享                                                                                          | ![4](/rating-8.png) 可以共享给他人                                                                                                                                                                           | ![5](/rating-10.png) 可以免费共享给他人                                                                                                                 | ![5](/rating-10.png) 可以免费共享给他人                                                 |
| [Kodi 集成](#13-kodi-皮肤可用性和集成)                | 不适用                                                                                                                    | ![4](/rating-8.png) 通过 [Plex for Kodi](https://www.smarthomebeginner.com/plex-for-kodi-becomes-free/) 或 [PlexKodiConnect](https://www.smarthomebeginner.com/combine-plex-and-kodi-using-plexkodiconnect/) | ![5](/rating-10.png) 通过 [EmbyCon](http://kodi.emby.media/) 或 [Emby for Kodi](https://emby.media/emby-for-kodi.html)                                  | ![4](/rating-8.png) 通过 [Jellyfin for Kodi](https://github.com/jellyfin/jellyfin-kodi) |
| [虚拟现实（VR）](#14-虚拟现实)                        | ![0.5](/rating-1.png) 不支持                                                                                              | ![5](/rating-10.png) Plex VR 可用                                                                                                                                                                            | ![0.5](/rating-1.png) 不支持                                                                                                                            | ![0.5](/rating-1.png) 不支持                                                            |
| [关注用户/社区](#15-关注用户社区)                     | ![4](/rating-8.png)                                                                                                       | ![2](/rating-4.png)                                                                                                                                                                                          | ![3.5](/rating-7.png)                                                                                                                                   | ![4.5](/rating-9.png)                                                                   |
| [长期策略](#16-长期策略)                              | ![4](/rating-8.png) 预期免费和开源                                                                                        | ![2](/rating-4.png) 高概率被收购                                                                                                                                                                             | ![3](/rating-6.png) 小概率被收购                                                                                                                        | ![4](/rating-8.png) 预期免费和开源                                                      |
| [开发模型](#17-开发模型)                              | ![5](/rating-10.png) 开源                                                                                                 | ![2](/rating-4.png) 闭源（所有权）                                                                                                                                                                           | ![3](/rating-6.png) 几乎闭源，部分开源                                                                                                                  | ![5](/rating-10.png) 开源                                                               |
| 支持/维护                                             | ![4](/rating-8.png) 活跃社区                                                                                              | ![2](/rating-4.png) 开发者响应很慢。但论坛活跃                                                                                                                                                               | ![4](/rating-8.png) 开发者积极响应。活跃社区                                                                                                            | ![4.5](/rating-9.png) 开发者和用户都积极响应                                            |
| 媒体中心平台成熟度                                    | ![5](/rating-10.png) 成熟                                                                                                 | ![4.5](/rating-9.png) 已成型                                                                                                                                                                                 | ![4](/rating-8.png) 正在成型                                                                                                                            | ![3.5](/rating-7.png) 崭新                                                              |

如您所见四个应用程序的差异其实非常接近！在许多情况下，一个应用程序的强行征是另一应用程序的弱项。

### 有 Plex 和 Emby 的替代品吗？

> Jellyfin 是一个新兴的媒体中心，正处于非常活跃的开发阶段。是 Emby 的免费且开源的衍生品。因此并不涉及任何付费服务。虽然当前客户端应用程序的支持非常有限，但 Android 客户端还是很棒。

在评估选择安装哪个应用程序才能更合适时，您确实应该考量您的个人需求。假如您希望通过移动端应用程序访问自己的库，Plex 或 Emby 可能相较独立的 Kodi 更适合您。

## Emby 或 Plex ——细节比较

好了，让我们更深入低研究上表中提到的 Plex 和 Emby 的一些差异。通常来说，两者之间绝大多数的差异都源于是否开源，或程序的成熟度（即他们投入在研发中打磨产品的时间）。

### 1. 支持的服务端平台

Emby、Plex 和 Jellyfin 都需要配套的服务端。Plex 和 Emby 支持多个平台，可以满足绝大部分常用用户：

-   操作系统：Windows、Linux、Mac 和 FreeBSD
-   家用游戏机和媒体播放器 - Nvidia Shield TV
-   NAS
    -   两者都支持：Asustor、FreeNAS、Netgear ReadyNAS、Open Media Vault、QNAP、Synology、TerraMaster、Thecus 和 Western Digital
    -   限 Plex：Drobo 和 Seagate
-   路由器：Netgear Nighthawk（限 Plex）
-   Docker

除了 Drobo NAS、Seagate NAS 和路由器外，Plex 和 Emby 的服务端都可以很好地运行于任何平台。（参考阅读：[《最好的 Plex 媒体服务器：7 套出色的整机、NAS 和 DIY 配置》](https://www.smarthomebeginner.com/best-media-server-for-plex-2020/)）

Jellyfin 由于是新产品，服务端支持有限（常见操作系统的服务端）。也不支持 NAS、家用游戏机或路由器。

但 Plex、Emby 和 Jellyfin 都可以跑在 Docker 里。因此任何支持 Docker 的平台都可以运行这些媒体服务器。

> **推荐 HTPC/ 家庭服务器配置：**
>
> -   [《2017 能解决所有问题的最好家庭影院电脑配置（Plex、Kodi、NAS 和游戏）》](https://www.smarthomebeginner.com/best-home-theater-pc-build-2017-to-do-it-all-plex-kodi-nas-gaming/)
> -   [《2018 最好的 Emby 服务器配置——整机或组装配置单》](https://www.smarthomebeginner.com/best-emby-server-builds-2018/)
> -   [《2017 搭建 Kodi、Plex 和游戏的中度预算 4K HTPC 配置》](https://www.smarthomebeginner.com/medium-budget-4k-htpc-build-2017/)
> -   [《2017 搭建 Kodi、OpenELEC 和 LibreELEC 的廉价 4K HTPC 配置》](https://www.smarthomebeginner.com/4k-htpc-build-for-kodi-2017/)
> -   [《2017 网络文件和媒体存储的低能耗家庭服务器配置》](https://www.smarthomebeginner.com/low-power-home-server-build-2017/)
> -   [《2017 搭建 4K Kodi 的 \$400 中度预算的最好 HTPC》](https://www.smarthomebeginner.com/best-htpc-for-kodi-with-4k/)
> -   [《2016 高能效预算的 HTTP-NAS 组合配置》](https://www.smarthomebeginner.com/htpc-nas-combo-build-2016/)

### 2. 支持的客户端平台

Plex 超出 Emby 的最大优势之一就在于其更早更成熟。这让其能够支持相当广泛的客户端设备，甚至包括串流设备：

-   Roku
-   Amazon Fire TV
-   Chromecast
-   Apple TV
-   家用游戏机（XBox 和 PlayStation）
-   移动设备（Android 和 iOS）
-   智能电视（Samsung、Vizio、LG、Opera TV 等）
-   等等

看起来 Plex 的公司性质也让其与很多客户端设备公司的目标保持一致。（参考阅读：[《10 个最好的 Plex 客户端设备：整机和 DIY 配置单》](https://www.smarthomebeginner.com/best-plex-client-devices-2018/)）

Emby 也很快迎头赶上并支持了上述的所有平台。先忙的截图展示了 Emby 所支持的全部客户端平台。

![Emby 支持的客户端平台](/2020/05/17/emby-supported-platforms.png)

iOS 上的应用程序似乎有点落后。此外，与 Plex 相比客户端应用程序看起来还不够完善。（参考阅读：[《20 种最佳 Emby 客户端设备——通过服务端串流》](https://www.smarthomebeginner.com/best-emby-client-devices-2018/)）

[Plex](https://www.plex.tv/apps-devices/) 和 [Emby](https://emby.media/download.html) 的网站上有其各自支持的服务端和客户端平台的完整列表。

> **最佳 Plex 客户端设备：**
>
> 1. [NVIDIA SHIELD TV Pro 家庭媒体服务器](https://www.amazon.com/dp/B07YP9FBMM/?tag=shbeg-20) - \$199.99 _编辑推荐_
> 2. [Amazon Fire TV 串流媒体播放器](https://www.amazon.com/dp/B00U3FPN4U/?tag=htpcbeg-20) - \$89.99
> 3. [Roku Premiere+ 4K UHD](https://www.amazon.com/dp/B01LXUZPQU/?tag=htpcbeg-20) - \$83.99
> 4. [CanaKit Raspberry Pi 3 完整入门包](https://www.amazon.com/dp/B01C6Q2GSY/?tag=htpcbeg-20) - \$69.99
> 5. [Xbox One 500GB](https://www.amazon.com/dp/B00KAI3KW2/?tag=htpcbeg-20) - \$264.99

Jellyfin 此时只支持很少的客户端（网页应用程序和 Android），但 iOS 和 Roku 应用程序正在开发中。

> **更新（2020 年 3 月 26 日）：**好像可以使用 Emby 应用程序连接 Jellyfin 服务端。我在 Roku 中通过指定主机和端口号成功地用 Emby 应用程序连接到 Jellyfin 了。我猜在其它平台上这样也行得通。

Jellyfin 通过 DLNA 协议也支持家用游戏机。

### 3. 安装和使用的简易度

Emby 和 Plex 的安装过程很相似。但安装之后的配置就大相径庭了。

虽然两者都易于安装和使用，但 Plex 的简单性和易用性是显而易见的，更适合于新手用户。

![Plex 配置向导中添加库](/2020/05/17/plexs-step-by-step-walkthrough-to-add-libraries.png)

Emby 安装完成后，您会发现界面过于复杂，也没有配置向导，对新手用户不够友好。但经验丰富的用户还是可以通过各自的方式很快地搞定。

![Emby 添加库的方式](/2020/05/17/emby-media-server-library-addition.png)

Emby 具备更多功能也更可定制化——如强大的库控制、用户支持、家长控制和全面的元信息管理等等。

Emby 的开发者似乎更愿意接受用户的建议。因此自定义选项可能会随着时间的推移继续增加。（参考阅读：[《如何在 Ubuntu 服务器上轻松安装 Emby》](https://www.smarthomebeginner.com/install-emby-on-ubuntu/)）

最后总结，这其实是个人喜好的问题——更精致（Plex）还是更可定制（Emby）？Jellyfin 与 Emby 类似。

### 4. 可定制性

可定制性是我个人认为的 Plex 和 Emby 间最大的差异之一。这块明显 Emby 更强。但这其实取决于您是否是定制爱好者。

Plex 就保守得多，即插即用。Emby 虽然也可以即插即用，但其可自身和通过插件实现很多深度定制。下面是部分 Emby 提供但 Plex 未原生提供得定制：

-   对库中图片和缩略图创建机制的更多控制
-   Kodi 快速同步所需的 Kodi 伴侣插件
-   导入 IPTV 播放列表
-   备份服务端配置
-   对智能家居中枢的支持——SmartThings、Vera 和 Philips Hue
-   通知——邮件、Join、Prowl、Pushbullet、Slack 等等
-   更多字幕源——Addic7ed、Podnapisi、SubDB 等等，还有 OpenSubtitles

再次重申，这取决于您更喜欢 Plex 的简洁还是 Emby（或 Jellyfin）的可定制性。[Jellyfin 甚至允许您调整底层 CSS](https://jellyfin.org/docs/general/clients/css-customization.html) 以满足您对外观的偏好。

### 5. 本地和远端串流

Emby 和 Plex 的串流功能几乎一模一样。关于这个话题有许多困惑：

> **Plex / Emby 的远端串流功能免费吗？**
>
> 是的。两者的远端串流（和本地串流）都是免费的。但部分串流客户端可能需要购买 Plex 套票或 Emby 首映式，才能解锁播放限制。

请查看下面 Plex 套票和 Emby 首映式的功能对比以确定哪些客户端免费而哪些收费。在付费前，部分客户端只允许有限时长（比如 1 分钟）的播放。

Jellyfin 没有任何限制。

> **最佳串流应用程序和插件：**
>
> -   [《20 个最佳 Kodi 插件：升级后的可用列表》](https://www.smarthomebeginner.com/best-kodi-addons-2016/)
> -   [《20 个 Nvidia Shield TV 最佳串流应用程序：电影、音乐和其它》](https://www.smarthomebeginner.com/best-streaming-apps-for-nvidia-shield-tv-2017/)
> -   [《10 个 Amazon Fire TV 最佳媒体串流应用程序》](https://www.smarthomebeginner.com/best-amazon-fire-tv-apps/)
> -   [《10 个 Kodi 媒体中心可用的最佳合法串流插件》](https://www.smarthomebeginner.com/kodi-legal-streaming-addons/)

### 6. Plex 套票和 Emby 首映式

Plex 引入 Plex 套票来提供一些高级功能。然后 Emby 也有了 Emby 首映式。两边的价格差不多。碰上节假日您还可能遇到不错的终身会员的折扣。

| 付费方式 | Plex     | Emby   | Jellyfin |
| -------- | -------- | ------ | -------- |
| 月付     | \$4.99   | \$4.99 | 免费     |
| 年付     | \$39.99  | \$54   | 免费     |
| 终身     | \$119.99 | \$119  | 免费     |

Plex 套票和 Emby 首映式付费后增加的功能，或者说必须付费才能用的功能，是不同的。但差别并不明显，所以很容易搞混。下表提供了两者的对比摘要。请注意下表只列出了一些关键功能或主要区别。

| 功能                   | Plex                                                                                                             | Plex 套票                               | Emby                                    | Emby 首映式                                                                                                       | Jellyfin                                                                                   |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------- | --------------------------------------- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 摄像头上传             |                                                                                                                  | 可                                      |                                         | 可                                                                                                                | 否                                                                                         |
| 远端串流               | 可（参考完整播放）                                                                                               |                                         | 可（参考完整播放）                      |                                                                                                                   |
| 本地串流               | 可（参考完整播放）                                                                                               |                                         | 可（参考完整播放）                      |                                                                                                                   |
| 完整播放（本地和远端） | 网页应用程序、非移动端 Android（Fire TV 和 Android TV）、Apple TV、Chromecast、Roku、智能电视、TiVO 和家用游戏机 | 移动端 Android 和 iOS 应用程序—需要解锁 | 网页应用程序、Roku、Apple TV 和智能电视 | Android（包括 Fire TV 和 Android TV）、iOS、Emby 剧场、家用游戏机—需要解锁                                        | Android（包括 Fire TV）、iOS、tvOS、网页应用程序、家用游戏机（通过 DLNA）、Roku 和其它平台 |
| 媒体优化               | 可                                                                                                               |                                         |                                         | 可                                                                                                                | 否                                                                                         |
| 硬件转码               |                                                                                                                  | 可                                      |                                         | 可                                                                                                                | 可                                                                                         |
| 电视直播               |                                                                                                                  | 可                                      |                                         | 可                                                                                                                | 可                                                                                         |
| DVR                    |                                                                                                                  | 可                                      |                                         | 可                                                                                                                | 可                                                                                         |
| 移动端/文件夹同步      |                                                                                                                  | 可                                      |                                         | 可                                                                                                                | 可                                                                                         |
| 多用户                 |                                                                                                                  | 可（需要 Plex 线上账号）                | 可（本地账号，与 Emby 关联是可选操作）  |                                                                                                                   | 可（本地账号）                                                                             |
| 家长控制               |                                                                                                                  | 可                                      | 可                                      |                                                                                                                   | 可                                                                                         |
| 相片专辑               |                                                                                                                  | 可                                      | 否                                      |                                                                                                                   | 否                                                                                         |
| 歌词                   |                                                                                                                  | 可                                      | 否                                      |                                                                                                                   | 否                                                                                         |
| 库共享                 | 可                                                                                                               | 更多控制                                | 可                                      |                                                                                                                   | 可                                                                                         |
| 预告片和花絮           |                                                                                                                  | 可                                      |                                         | 可                                                                                                                | 否                                                                                         |
| 云同步                 | 否（[Plex 云停止服务](https://forums.plex.tv/t/plex-cloud-to-be-discontinued-nov-30-2018/306986)）               |                                         |                                         | [Google Drive](https://support.emby.media/support/solutions/articles/44001162194-google-drive) 或 DropBox（插件） | 否                                                                                         |
| 智能家居               | 非官方（[FlexTV](https://github.com/d8ahazard/FlexTV)）                                                          |                                         |                                         | Alexa 和 Google 助手                                                                                              | 无                                                                                         |
| 其它内容               | 电影、电视、在线秀、广播和视频                                                                                   |                                         |                                         | 广播                                                                                                              | 基于插件                                                                                   |
| 备份和恢复             | [手动](https://support.plex.tv/articles/201539237-backing-up-plex-media-server-data/)                            |                                         |                                         | 可（使用插件）                                                                                                    | 否                                                                                         |

功能的最新信息请通过这里的链接进行查看：[Plex 套票](https://www.plex.tv/plex-pass/#modal-features) 和 [Emby 首映式](https://support.emby.media/support/solutions/articles/44001173099-emby-premiere-feature-matrix)。

> **2020 年 4 月 27 日**：Plex 为会员发布了[两款全新的应用程序](https://www.plex.tv/blog/two-delicious-new-apps-from-plex-labs/)：通过移动电话管理服务器的 Plex Dash 和全新的音乐串流体验 Plexamp。两个应用程序都很受好评。

两者对比的关键亮点在于：

-   Plex 的串流功能在绝大部分客户端上都是免费的（Roku、Fire TV、家用游戏机等）。而 Emby 则需要付费才能解锁很多平台。对于移动平台（Android 和 iOS）两家都要求付费。
-   Plex 只免费提供基础库共享，需要付费才能高级控制。Emby 则没有付费要求。
-   Plex 添加多个用户也需要付费，Emby 则提供了完全免费的、私密的、高级的用户管理功能。
-   Plex 提供免费内容（电影、电视、在线秀、广播和新闻），Emby 只有付费用户才能享受广播。
-   Plex 放弃了云同步。而 Emby 的付费用户还可以使用 Google Drive（或使用 DropBox 插件） 同步媒体。
-   如果 Jellyfin 能满足您的需求，可别忘了它是免费的。

因此是否付费取决于您使用什么客户端和什么功能。和 Kodi 类似，Jellyfin 也有一些频道提供免费内容（比如 [LazyMan](https://github.com/dkanada/awesome-jellyfin)）。但千万要注意使用 [VPN](https://www.smarthomebeginner.com/go/ipvanish) 来保护自己的隐私。

### 7. 电视直播和 DVR 支持

Plex 和 Emby 都提供了电视直播和 DVR，但其中还是有些异同。

-   都需要付费才能使用电视直播和 DVR 功能。
-   Plex 支持很多调谐器（HDHomeRun、DVBLogic、AVerMedia 和 Hauppauge）而 Emby 只支持 HDHomeRun 和 Hauppauge（限 Windows）。
-   Plex 一次只能使用一个物理调谐器。Emby 则可以一次使用多个。
-   Plex 频道上限为 480。Emby 没有限制。

如果您在美国在支持的调谐器上使用 OTA 天线，两者都工作正常。在 Plex 中要用满 480 个频道是很稀罕的事情。但如果您是订阅了 IPTV（其频道数量超过 480 个），Plex 就无法正常处理了。

![Plex 使用 HDHomeRun 调谐器时的电视直播](/2020/05/17/plex-live-tv-guide.png)

您可以[使用诸如 xTeVe 或 TellyTV 截断列表后再传输给 Plex](https://www.smarthomebeginner.com/plex-iptv-guide/)。以前 Plex 是允许通过查看从 IPTV 播放列表中观看视频流的。但现在您只能通过扩展源中继的方式来截断您的频道列表。

![Jellyfin 和 Emby 的电视直播](/2020/05/17/jellyfin-live-tv.png)

Emby（包括 Jellyfin）对此并无限制。两者甚至可以导入 m3u 播放列表。更多的调谐器硬件可以通过插件来支持。就我个人而言，更喜欢 Emby 提供的电视直播和 DVR 功能。因为看起来更完善。（参考阅读：[《指南：如何安装 Emby IPTV 插件》](https://www.smarthomebeginner.com/guide-install-emby-iptv-plugin/)）

### 8. 数据库管理——元信息管理器

Plex 和 Emby 都提供了可以通过网页管理工具访问的中央管理数据库。对大多数用户来说两者都很容易掌握，但我发现 Plex 更偏向于非专业用户，提供了更简单、更直观的库操作方式。

![Plex 元信息管理器](/2020/05/17/plex-metadata-manager.png)

另外一边，Emby 数据库管理器提供了更多的功能。最好的管理数据库的方法之一就是通过内置的“元信息管理器”将全部电影都罗列出来，并使用图标对列表中的每一项标记缺失的元信息（图片、预告片、字幕等）。下图展示了这个功能的具体界面。

![Emby 元信息管理器](/2020/05/17/emby-metadata-manager.png)

如果您认为数据库管理就得查找方便，自己也还是新手，那么 Plex 更适合您。如果您更需要得是更多的管理元信息的功能和方法，那么 Emby 就是您想要的。

Jellyfin 目前并没有专业的元信息管理器，只能通过网页界面进行编辑。

### 9. 免费内容

Plex 和 Emby 原本都意在观看和分享个人媒体。但 Plex 额外提供了免费内容：

-   电影和电视——[添加于 2019 年 12 月](https://www.plex.tv/blog/boom-we-just-dinosized-your-movie-collection-for-free/)
-   广播——添加于 2018 年 6 月
-   新闻——添加于 2017 年 9 月

![Plex 的带广告的免费电影和电视](/2020/05/17/plex-free-movies-and-tv.jpg)

此外，Plex 还支持如 TIDAL（高保真音乐串流服务） 之类的第三方服务。最近的一些举动（如禁用插件以打击盗版）表明可能会有更多这样的举动。

相反，Emby 只提供了广播内容，还只针对付费用户。Jellyfin 则完全没有提供免费内容。

因此如果你觉得免费内容很重要，Plex 是不二选择。

### 10. 插件（亦称频道）支持

Plex 和 Emby 以前都可以使用插件（Plex 称之为频道）。插件通过添加新的功能（如[字幕自动下载器](https://www.smarthomebeginner.com/install-sub-zero-plugin-for-plex/)）或串流方案（如[串流频道](https://www.smarthomebeginner.com/best-plex-unofficial-channels-2017/)）来扩展媒体服务器的能力。Plex 的非官方应用商店 [PiexWebTools](https://www.smarthomebeginner.com/install-plex-web-tools-2-0/) 也提供了一些插件。

但 Plex 已经[声明停止插件支持](https://www.plex.tv/blog/subtitles-and-sunsets-big-improvements-little-housekeeping/)。您只能使用 Plex 自身提供的功能。无法再添加功能，或从互联网上获取非官方的串流内容。

![Emby 头部插件](/2020/05/17/emby-plugins.png)

另一边，Emby 和 Jellyfin 都还支持插件。本文撰写时 Emby 约有 75 个插件，Jellyfin 则将近一半。所有的插件都可以通过界面安装。

因此插件支持方面，Emby 完胜。次之是 Jellyfin，再次 Plex。

### 11. 离线浏览（同步）

离线浏览功能需要付费。然后您就可以将内容下载到客户端设备中，无论有没有网络都可以观看。

![Plex Android 客户端的离线观看/下载功能](/2020/05/17/offline-viewing-on-plex-android.png)

尽管两者都支持这一功能，Emby 的表现要差于 Plex，特别是在 iOS 设备上。

Jellyfin 尽管也可以通过网页应用程序下载内容，但客户端应用程序中并没有实现这一功能（至少 Android 中如此）。

### 12. 用户管理

Plex 和 Emby 的用户管理完全不同。Plex 首先要先付费才能添加多个用户。Emby 的这个功能则是免费的。而且更多控制项、更私密。您可以为家庭成员创建“托管用户”（本地账户）。

所有的 Plex 用户都必须有线上账号才能访问您的服务器，幸而[您无需线上账号就可以本地串流](https://support.plex.tv/articles/207538527-do-i-need-a-plex-account-to-stream-locally/)。但默认管理账户还是必需线上账号。

![Plex 添加托管用户](/2020/05/17/plex-add-managed-users.png)

必需 Plex 线上账户才能管理和分享给其他人，对某些注重隐私的人来说可能是个大难题。在 Emby 里所有的账户都是本地的（即便是付费用户）。

![Emby 添加用户](/2020/05/17/emby-add-new-user.png)

而且 Emby 还免费提供更多用户级别的控制项。Jellyfin 和 Emby 一样提供了很多控制项，也一样不需要线上账户。这让 Emby 和 Jellyfin 更具优势。

### 13. Kodi 皮肤可用性和集成

Plex 和 Emby 的关键差异之一还在于对 Kodi 的集成支持。Emby 做的很不错，Plex 也通过[官方 Plex Kodi 插件](https://www.smarthomebeginner.com/plex-for-kodi-becomes-free/)和 PlexKodiConnect 插件迎头赶上。不过许多人还是更亲睐 Emby 一些。（参考阅读：[《组合 Plex 和 Kodi——使用 PlexKodiConnect 一起使用两者功能》](https://www.smarthomebeginner.com/combine-plex-and-kodi-using-plexkodiconnect/)）

![使用 Emby for Kodi 插件在 Kodi 中看到的 Emby 库](/2020/05/17/emby-for-kodi-addon.jpg)

Kodi 的集成方法有两种：

1. **作为单独的视频插件集成：**Plex 和 Emby 都可以将 Kodi 作为视频单独的视频插件集成——就像在智能手机或 Roku 里安装应用程序。这种集成方式中，Kodi 和 Plex/Emby 的功能是隔离的。在 Emby 中可以使用 [EmbyCon Kodi 插件](http://kodi.emby.media)，在 Plex 中可以用 [Plex for Kodi 插件](https://kodi.tv/addon/scripts-video-add-ons/plex)。

2. **合并 Kodi：**另一种方法是将 Plex/Emby 库和 Kodi 合并。安装方法较上一种会复杂一些，但可以使用 Kodi 的原生界面实现很多惊艳的功能。Plex/Emby 和 Kodi 定期同步以确保库的一致性。您甚至可以选择要同步哪个库。在 Emby 中可以使用 [Emby for Kodi 插件](https://emby.media/emby-for-kodi.html)，而在 Plex 中使用 [PlexKodiConnect 插件](https://forums.plex.tv/t/plexkodiconnect-kodi-plex-integration-done-right/137555)（Emby for Kodi 的分支）。

![PlexKodiConnect 界面](/2020/05/17/plexkodiconnect-screenshot.jpg)

Emby for Kodi 和 PlexKodiConnect 都可以使用任意 Kodi 的皮肤。

时至今日，两者对 Kodi 的支持几乎没有区别。

### 14. 虚拟现实

我们并未将此对比放入值得拥有的功能列表中。如果 VR 对您很重要，那 Plex 还是 Emby 会更适合？这很简单。

与 Emby 不同，[Plex 提供了 Plex VR](https://www.plex.tv/your-media/virtual-reality/)，您可以假装身处剧场环境来观看库中内容。

![Plex VR](/2020/05/17/plex-virtual-reality.jpg)

此外，您还可以和您远方的朋友或家人一起观看您的内容，甚至还可以在观看时互动。

### 15. 关注用户/社区

Plex 已成长为一家商业公司。努力赚钱和在日常问题上与用户和社区建立联系，这两件事情要并行不悖就日益困难。Plex 有一个官方论坛，除此之外您还可以从 r/Plex 中获得帮助。但要直接与开发人员交流是几乎不可能的。您的反馈意见可能被采纳，也可能不被。

Emby 同样也有[论坛](http://emby.media/community)和 [r/emby](https://www.reddit.com/r/emby/)。您甚至能在某些时候看到开发者潜水或是回答问题。此外，您还可以将问题或反馈提交到 Emby 的 [GitHub 页面](https://github.com/MediaBrowser/Emby)（尽管在写文章时发现已经有段时间不活跃了）。

另一边的 Jellyfin 目前就关注用户多了，我亲眼所见很长一段时间内都是如此。以下是您可以为 Jellyfin 社区做出贡献或交流的途径：

-   论坛 - https://forum.jellyfin.org/
-   Reddit - [r/jellyfin](https://www.reddit.com/r/jellyfin/)
-   功能申请 - https://features.jellyfin.org/
-   GitHub - https://github.com/jellyfin/jellyfin
-   Matrix Chat/IRC - https://matrix.to/#/#jellyfin:matrix.org
-   示范站点 - https://demo.jellyfin.org/

Emby 的支持也很棒（唯一缺失的就是聊天）。因此在和社区和用户保持联系这一方面，我的评价是 Jellyfin 优于 Emby 再优于 Plex。

### 16. 长期策略

有明确迹象表明 Plex 正在改变其商业模式。Plex 正在与媒体公司合作，将带广告的免费内容和付费订阅归为一类。但这与以下内容有直接冲突：

1.  插件 - 插件又被称作频道。有许多[免费和非法的流媒体频道](https://www.smarthomebeginner.com/best-plex-unofficial-channels-2017/)（类似于 [Kodi 插件](https://www.smarthomebeginner.com/best-addons-for-kodi-18-leia-2019/)）。因此，Plex 在 2018 年底[取消了对插件的支持](https://www.plex.tv/blog/subtitles-and-sunsets-big-improvements-little-housekeeping/)。

2.  库共享 - Plex 用户将他们的库共享给家人和朋友。一部分用户可能还运行着私人（合法性有待商榷）“Netflix”，甚至像生意一样收费共享。因此版权团队一直声称 Plex 在阻止盗版方面作得不足。

因此我们不知道 Plex 和 Emby 的长期策略将会走向何方。如果某些大的媒体公司想要收购它们，我会认为 Plex 才会是当下的大目标。

Plex 目前的一些动向意味不明。因此短期内，我会更愿意用 Emby（或者 Jellyfin）来构建我的媒体服务器。

### 17. 开发模型

最后但并非最不主要的一点，是代码的性质。免费和开源对很多人而言都是非常重要的。如果您是其中之一，那么 Plex 可能就不是您的选择。

即便其同样源自 XMBC（Kodi 的前称），Plex 是完全闭源和专有的。

Emby 已部分闭源。服务端代码是开源的，并在 GitHub 上可以找到。但绝大多数客户端应用程序已经闭源了。

Jellyfin 是完全免费和开源的，包括服务端和客户端应用程序的代码。但请注意，目前 Jellyfin 支持的客户端很有限。

## 示例场景

决定究竟基于哪个平台来构建您的媒体服务器生态系统是一件很困难的事情。因此我们提供了一些示范场景以帮助您从 Plex、Emby 和 Jellyfin 中选择。

### Kodi 最适合，当

-   您无需要远端串流
-   您不希望花钱
-   您不关心多设备中库的同步问题，也不介意通过 [MySQL 共享库](https://www.smarthomebeginner.com/kodi-mysql-setup-to-share-library/)或使用别的[库同步方法](https://www.smarthomebeginner.com/backup-kodi-watched-status/)。

### Plex 最适合，当

-   您需要合法的免费内容以扩充您的个人库
-   您相较可控性更亲睐安装的便捷和精致的外观
-   您并不介意一些高级功能付费（电视直播、DVR 等）

### Emby 最适合，当

-   您更亲睐可控性和可定制性
-   您更在意用户隐私
-   您并不介意为一些高级功能付费（智能家居功能、电视直播、DVR 等等）
-   您更喜欢 Kodi 的界面。您可以运行 Emny 服务端作为后端，通过 Emby for Kodi 插件来使用 Kodi 的界面。虽然也可以通过 PlexKodiConnect 来和 Plex 协同工作，但很多用户似乎更喜欢 Emby 的 Kodi 集成。

### Jellyfin 最合适，当

-   您更亲睐可控性和可定制性
-   您更亲睐免费和开源软件
-   您使用服务器运行 Jellyfin（或能运行 Docker 的 NAS），同时您的客户端都是 Android

## 谁更棒？Plex 还是 Emby？

答案显而易见，这取决于您的个人偏好。我喜欢 Emby/Jellyfin 提供的可操控性，而我的家人更喜欢 Plex 精致的 UI。

在对比 Plex 和 Emby 时，其相似之处远过于差异——都以少量的费用便提供了一种简单方法，使您能从多设备中集中媒体数据库，还包括对 Kodi 的集成。

Emby 的优势在于其有一种可以带来一定收益的商业模式（Emby 首映式）。这也有助于吸引一些敬业的开发人员，在不断改进的同时还愿意聆听社区的意见。

Jellyfin 是这块的新手，目前看来还略显粗糙。但它也绝对没有任何开销。另一方面，它的免费性质也意味着开发人员只能是有时间时就做出贡献的爱好者。但似乎有一些热情的开发者正在积极地工作。我坚信 Jellyfin 有成长和挑战 Plex 和 Emby 的潜力。

就我个人而言，我同时运行着全部三套系统（Plex、Emby 和 Jellyfin）。有了 Docker 要启动和运行起来是很容易的事情。家里使用 Plex，我个人使用 Emby（AirSonic for Music），Jellyfin 则是我的备份。三者都是对我的智能家居设置的补充。

最后的最后，关于 Plex 还是 Emby 的选择还是取决于您的需求。希望本文能帮您确定最适合您的产品。
