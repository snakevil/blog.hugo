---
title: "【译】Plex vs. Emby vs. Jellyfin——谁才是最优解？"
slug: "Plex-vs-Emby-vs-Jellyfin-谁才是最优解？"
date: 2020-05-16T18:33:09+08:00
category:
    - 家庭组网
tag:
    - Plex
    - Emby
    - Jellyfin
    - 自主翻译
---

[点击这里阅读原文。](https://medium.com/@EthR/plex-vs-emby-vs-jellyfin-which-is-the-best-option-6e308454c410)

---

家用媒体市场主要有三个竞争者。您很有可能听说过 [Plex][] 和 [Emby][]——而 [Jellyfin][] 则是一位日益流行的新秀。

[plex]: https://www.plex.tv
[emby]: https://emby.media
[jellyfin]: https://jellyfin.org

<!--more-->

在它们的官网上是这样自我定义的：

## [Plex][]

> [Plex][] 让您只用一处就可以找到和访问所有与您有关的媒体。从您自有服务器上的个人媒体，到播客、网络节目和新闻，甚至再到音乐流，您都可以在任何设备中通过一个应用程序便尽情享受。

## [Emby][]

> **您在任何设备上的个人媒体**

> 将您全部的家庭视频、音乐和相片汇集到一处变得前所未有的简单。您的个人 [Emby][] 服务器将自动实时转换和流式传输您的媒体内容，从而在任何设备中播放。

## [Jellyfin][]

> [Jellyfin][] 是能让您管理和流式传输您的媒体内容的免费软件媒体系统。没有附加条件，没有基于高级许可证的功能，也没有隐藏风险（agenda）。

仔细观察，会发现 [Jellyfin][] 的说明中包含了许多细节。[Jellyfin][] 直言“没有附加条件，没有基于高级许可证的功能，也没有隐藏风险”。这在后文中很重要。

三者的功能很相似，所以就不做详细比较了。

## 值得一提的差异

-   [Emby][] 和 [Jellyfin][] 有免费的家长控制功能
-   [Plex][] 是闭源的
-   [Emby][] 和 [Jellyfin][] 都提供免费的电视直播支持
-   [Jellyfin][] 不如它的竞争对手那样有功能繁多的移动端应用程序
-   [Emby][] 的大多数移动应用程序都需要付费

## 用户界面对比

![Plex UI](/2020/05/16/plex.ui.jpg)

![Emby UI](/2020/05/16/emby.ui.jpg)

![Jellyfin UI](/2020/05/16/jellyfin.ui.jpg)

老实说，[Plex][] 最好看。哪怕个人审美各不相同。[Jellyfin][] 和 [Emby][] 虽然同出一源，但区别还是很明显的。

## [Jellyfin][] 是啥？

[Jellyfin][] 由 [Emby][] 社区制作。在 [Jellyfin][] 之前，[Emby][] 是完全开源的。后来 [Emby][] 开始半闭源，破坏了社区的信用。而当其某些功能开始要求付费才能使用后，情况就变得更糟糕了。

于是社区转而创建了 [Jellyfin][]。一个基于 [Emby][] 最后的开源版本的免费开源重构项目。将 [Emby][] 的所有收费功能 100% 免费提供。并且承诺永远开源和免费。

尽管 [Jellyfin][] 看起来很不错，但它还很新。如果您坚持想使用 [Plex][] 和 [Emby][]，请看看它们的高级套餐。

## 功能与价格

高级套餐被称作 [Plex 套票](https://www.plex.tv/en-gb/plex-pass/) 和 [Emby 首映式](https://emby.media/premiere.html)。官网上的价格也是差不多：

| 产品     | 月费        | 年费       | 终身        |
| -------- | ----------- | ---------- | ----------- |
| [Plex][] | \$4.99USD   | \$39.99USD | \$119.99USD |
| [Emby][] | \$4.99/每月 | \$54/一年  | \$119/单次  |

[Emby][] 年付要贵 1 块钱，但其它选项价格就一样了。其功能如下：

![Emby 功能](/2020/05/16/emby.features.jpg)

[这里](https://support.emby.media/support/solutions/articles/44001173099-emby-premiere-feature-matrix)可以看到 [Emby][] 的完整“功能矩阵”。

[Plex][] 功能差不多：

![Plex 功能](/2020/05/16/plex.features.png)

[Plex][] 还开放了一个新的带广告的免费流服务。在[这里](https://www.plex.tv/en-gb/watch-free/)可以了解更多。

## 我的诚建

我认为虽然 [Plex][] 和 [Emby][] 都提供了出色的服务，但 [Jellyfin][] 才是卓越的。需要花钱才能在 [Plex][] 或 [Emby][] 中使用的功能，都可以在 [Jellyfin][] 中免费使用。

[Jellyfin][] 看起来也很棒。尽管开处于开发阶段，但已经有很多移动端应用程序了。如 FireTV、WebOS 和安卓等。而且 [Jellyfin][] 在 [Reddit](https://www.reddit.com) 中还有一个很不错的论坛社区，和可以实时聊天的 Matrix 频道。

如果您正在寻找免费选择，那么我真心推荐 [Jellyfin][]。如果您不差钱，那 [Plex][] 也是不错的选择。

[Emby][] 的开发团队正在改变他们的商业模式，每次都会对社区造成很大的冲击。因此很难预测 [Emby][] 将何去何从。

[Jellyfin][] 是我个人的选择，因为我相信他们真的会永久开源和免费。所以如果您真的想试试 [Plex][] 或 [Emby][] 之外的产品，请试试 [Jellyfin][]。

## 留言评论

> 我是一名相当长期的 Plex 用户，正在考虑迁徙到 Jellyfin 或 Emby，希望能找到一些比较意见。
>
> 您能否证实您的评价？特别是“Emby 对他们的社区并不友好，没人知道开发团队接下来会干些什么”
>
> 如果您最近一直在 Plex 社区中，您应该知道那里也有很多不开心的人。而且在很多人看来，Plex 变得越来越公司化，越来越无视社区意见。但有趣地是您指出了 Emby 的开发者社区的问题，却对 Plex 和 Jellyfin 的社区问题一字不提。
>
> 我虽然没有参与过 Emby 和 Jellyfin 的社区，但就目前所简单了解的 Emby 论坛情况，并没有如此时的 Plex 社区里那么多的冲突。您的文章写于几个月之前，事实上 Plex 圈子自 Tidal 功能之后问题就愈发严重了，他们强制将免费流服务塞给所有人（但并没有人请求过这个功能）。您认为这个功能是积极的，但普遍观念并非如此。
>
> 事实上，在我看来，您的文章似乎有点不大对劲。老实说，有点像 Plex 的广告软文。
