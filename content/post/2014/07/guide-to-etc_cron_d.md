---
title: "/etc/cron.d 攻略"
slug: "etc-cron_d攻略"
date: 2014-07-17T10:01:15Z
lastmod: 2014-08-12T16:12:15Z
toc: true
category:
    - Linux
    - 开发技巧
tag:
    - Linux
    - crontab
---

[crontab][] 是日常使用最为频繁地定时器工具。它将每个用户的所有定时任务统一记录、统一管理。

## 为什么不用 [crontab][] ？

[crontab]: http://linux.die.net/man/1/crontab

<!--more-->

[/etc/cron.d]: http://linux.die.net/man/8/crond

但对于项目系统中的定时任务而言，不同的定时任务可能需要交由不同的系统帐号执行。但如此，管理又很容易出现疏漏，比较尴尬。特别是当项目地不同时期由不同人员负责时，出现疏漏地概率会变得更高。

这种情况下，将所有定时任务以文件方式统一管理地 [/etc/cron.d][] 似乎就更被我们所亲睐。

## [/etc/cron.d][] 怎么玩？

[crontab][] 的定时任务格式是：

```
# MIN HOUR DAY MON WEEK CMD
*/15 * * * * /usr/sbin/ntpdate time.nist.gov > /dev/null 2>&1
```

[/etc/cron.d][] 的任务文件格式与之基本相似，唯独多了 `USER` 一项：

```
# MIN HOUR DAY MON WEEK USER CMD
*/15 * * * * root /usr/sbin/ntpdate time.nist.gov > /dev/null 2>&1
```

将上述内容保存至 [/etc/cron.d][] 目录中的 `ntpdate.job` 文件，就搞定了。

我们可以这样的等价指令来理解 [/etc/cron.d][] 的定时任务是如何被执行地：

```sh
'sudo' -u <USER> -H <CMD>
```

## 检查是否被执行

查看 `/var/log/cron` 日志，如找到下面地记录，就说明执行成功了。

> Jul 17 17:01:01 centos crond[21350]: (root) CMD (/usr/sbin/ntpdate time.nist.gov > /dev/null 2>&1)

## 统一管理项目定时任务

> ```
> $ sudo -s
> # cd /etc/cron.d
> # ls -1 project.*.job
> project.update-foo.job@
> project.remove-bar.job@
> project.sync-blah.job@
> ```

## WRONG FILE OWNER

但有时候，我们会从 `/var/log/cron` 日志中找到这样地记录：

> Jul 17 16:51:01 centos crond[1885]: (\*system\*) WRONG FILE OWNER (/etc/cron.d/project.update-foo.job)

这是因为 `project.update-foo.job` 这个任务文件的权限不是 `root:root`。这也是为什么上文中会使用 `sudo` 作为等价指令地缘由。

## (Oops)

还有时候（今天），我们会无法从日志中看到新配置地任务记录。追查 `man cron` 之后才发现——

除了权限问题之外， `/etc/cron.d` 文件夹中的任务文件命名有特殊要求，只能使用 `[\w\-]` 字符，不能有 `.` ！

## [crontab][] + [/etc/cron.d][]

回到最初的问题，我们需要思路上理清什么时候该用 [crontab][] ？什么时候又该用 [/etc/cron.d][] 呢？

#### 最原始、最粗糙地识别方法——基于执行帐号判断

如果执行帐号是系统帐号，那么就 [/etc/cron.d][] ；如果执行帐号是非系统帐号，那么就 [crontab][] 。

这种方法在大多数情况下，都是行之有效地。简单、粗暴、实用。
