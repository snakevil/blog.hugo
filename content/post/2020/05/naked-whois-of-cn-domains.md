---
title: "毫无隐私可言的 .cn 域名 WHOIS 信息"
slug: "毫无隐私可言的cn域名WHOIS信息"
date: 2020-05-09T21:57:31+08:00
category:
    - 杂谈日记
tag:
    - WHOIS
    - ICANN
    - CNNIC
---

这两天提起精神给我最近从[阿里云][]买的一个 `.cn` 域名备案。正巧前两天看了一个财经类自媒体的视频节目，提到了如何通过域名的 [WHOIS](https://baike.so.com/doc/5507304-5743050.html) 信息（吐槽为啥中文泛百科只有 baidu 和 360 两家？我没一个瞧得上的）。心血来潮就说自己也试着玩玩，结果把自己吓一大跳！赤裸裸毫无隐私可言！！真&trade;荒诞！！！

[阿里云]: https://www.aliyun.com

<!--more-->

随手打开搜索引擎，通过关键词“_WHOIS 查询_ ”找到了 [站长之家](http://www.chinaz.com)。心想这么老牌的站点工具了，其提供的 WHOIS 信息应该比较靠谱：

> [http://whois.chinaz.com/abc.cn](http://whois.chinaz.com/abc.cn "abc.cn的Whois信息 - 站长工具")
>
> ![站长之家 WHOIS](/2020/05/09/chinaz.png)

果不其然！我的名字（注册人）和邮箱（注册邮箱）就直接可以看到。虽然邮箱打了码，但可以直接 [whois 反查](http://whois.chinaz.com/reverse?host=**cn.jean@163.com&ddlSearchMode=1&domain=abc.cn)啊！

**这就意味着这第三方工具是知道我的真实姓名和完整邮箱的！是否会被滥用，仅仅在于第三方工具自己的商业道德！** 问题是，这种玩意儿真的存在于书本之外吗？！

但我依稀记得在[阿里云][]挑选买域名的时候，确定是看不到注册人信息的。于是我又回头试了试，确实如此。

> [https://whois.aliyun.com/whois/domain/abc.cn](https://whois.aliyun.com/whois/domain/abc.cn "关于域名 abc.cn 的whois信息及网站信息-阿里云")
>
> ![阿里云 WHOIS](/2020/05/09/aliyun.png)

[腾讯云](https://cloud.tencent.com)也是这样。

> [https://whois.cloud.tencent.com/domain?domain=abc.cn](https://whois.cloud.tencent.com/domain?domain=abc.cn "域名信息查询 - 腾讯云")
>
> ![腾讯云 WHOIS](/2020/05/09/tencent.png)

这就让我搞不懂了——_域名服务商一直都有提供隐私保护功能，第三方工具是如何知道我的名字和邮箱的？_

然后阿里云的客服告诉我：

> ![阿里云客服聊天记录](/2020/05/09/messages.jpg)
>
> “[ICANN（互联网名称与数字地址分配机构）][icann]于 2018 年 5 月 17 日公布《通用顶级域名注册数据临时规范（Temporary Specification for gTLD Registration Data）》，要求注册局和注册商对 WHOIS 查询服务的公开显示信息进行必要调整。此次调整是 [ICANN][] 为应对 2018 年 5 月 25 日生效的欧盟《通用数据保护条例（GDPR）》所做出的调整。 为落实 [ICANN][] 临时规范要求，自 2018 年 5 月 25 日起阿里云 WHOIS 查询公开信息中将不再显示域名注册人、管理联系人和技术联系人的个人数据，包括姓名、邮箱、电话、街道地址等。”
>
> “通过阿里云注册的域名，除.COM .NET 等 Verisign 域名的 WHOIS 信息由阿里云直接按调整后的规则提供外，其他阿里云域名的 WHOIS 信息均由相应的注册局提供，并由各注册局自行决定在其 WHOIS 平台的显示信息内容。由于各注册局、注册商对于 GDPR 和 WHOIS 显示信息调整的落实方案与进度暂不统一，因此该等在第三方 WHOIS 平台如何显示，取决于对应的注册局和注册商政策。”
>
> “这个需要到对应三方平台那边咨询反馈”
>
> “这个是三方平台和注册局那边的政策”
>
> “三方平台显示信息非阿里云提供，.cn 域名 注册局是 [cnnic（中国互联网络信息中心）][cnnic] 目前 [cnnic][] 暂时并未对域名做隐私保护 阿里云已经按照要求保护隐私做的不展示，如第三方平台展示客户信息不排除是调用注册局接口获取 注册局 whois [whois.cnnic.net.cn](https://whois.cnnic.cn/WelcomeServlet) 您可以自行查看”
>
> “阿里云只保证在阿里云注册的 com 和 net 后缀 在所有 whois 平台查不到域名所有者 其他后缀的域名信息不受阿里云控制 第三方 whois 可以自行通过调用注册局的接口获取域名真实信息”

[icann]: https://www.icann.org
[cnnic]: https://whois.cnnic.cn

我仔细再读三读客服贴给我的帮助解释说明，简直荒诞得令人不敢置信！大概意思就是 [ICANN][] 在 2018 年 5 月发了一个临时规范，然后 [CNNIC][] 跟进把域名服务商的隐私保护服务停了。但到现在 2020 年 5 月都整整两年了，后继还没跟上。**所有 `.cn` 域名的注册人信息还裸着，就这么裸了两年，还会继续这么裸下去…** 懒得匪夷所思！

在搜索引擎通过“_某某云域名隐私保护_ ”的关键词组合，确实找到了[《阿里云域名隐私保护服务暂停通知》](https://help.aliyun.com/knowledge_detail/84444.html)、[《腾讯云域名隐私保护相关——下线说明》](https://cloud.tencent.com/document/product/242/30404#.E4.B8.8B.E7.BA.BF.E8.AF.B4.E6.98.8E)和[《华为云为什么下线域名隐私保护功能？》](https://support.huaweicloud.com/domain_faq/domain_faq_0002.html)。

再跑到[中国互联网络信息中心][cnnic]官网一看，实锤！

> [https://whois.cnnic.cn/WhoisServlet?queryType=Domain&domain=abc.cn](https://whois.cnnic.cn/WhoisServlet?queryType=Domain&domain=abc.cn "whois")
>
> ![CNNIC WHOIS](/2020/05/09/cnnic.png)

狂怒！一股酸臭的官僚气息！唉，不说了。
