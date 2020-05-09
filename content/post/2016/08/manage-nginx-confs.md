---
title: "如何更好地管理 Nginx 多主机配置"
slug: "如何更好地管理Nginx多主机配置"
date: 2016-08-26T05:45:17Z
lastmod: 2016-08-26T07:50:46Z
toc: true
category:
    - 开发技巧
tag:
    - Linux
    - Nginx
---

[Nginx][] 一直以来都有一个很尴尬地问题——就是为了效率，抛弃了 [htaccess](http://baike.baidu.com/view/91163.htm) 目录配置文件。当一个项目打算使用 [Nginx][] 来提供 HTTP 服务时，就不得不在配置文件中大量地**硬编码**目录信息，可移植性和可维护性很差。那么，能否找到一种相对变通的方法，来提高可移植性和可维护性？

[nginx]: http://nginx.org/

<!--more-->

## 〇 从网络节点的主机名称出发

传统方法中，当一套网络系统需要部署在多个节点配合工作时，节点相互之间的通信必须通过赋予地主机名称完成，而非 IP 。IP 和主机名称的对应关系，由 `/etc/hosts` 文件负责维护。

其目的，就是遵循[解耦](http://baike.baidu.com/view/471757.htm)地思路，减少程序代码中的**硬编码**内容（网络节点 IP）。

借鉴这种方法，将项目配置文件都使用[符号链接][]“假装”存放在 [Nginx][] 系统配置目录 `/etc/nginx` 中，是否可行？

[符号链接]: http://baike.baidu.com/view/1955541.htm

## 一 [Nginx][] 配置

### 1.1 公用配置

系统配置目录中有一个 `conf.d` 的子目录，其中后缀名为 `.conf` 的文件，会在主配置文件的最后（[`http` 块][nginx#http]）被加载。

[nginx#http]: http://nginx.org/en/docs/http/ngx_http_core_module.html#http

因为 [Nginx][] 不支持动态配置调整，所以一些声明定义在整个配置中地先后顺序，并不影响其功能使用。

那么，一些跨域的公用性配置可以都放在这个目录下。比如 `/etc/nginx/conf.d/upstream-php.conf`：

```nginx
upstream php {
    server unix:/var/run/php5-fpm.sock;
}
```

理论上，不仅仅是 [`upstream`][nginx#upstream] ，其余所有允许使用在 [`http` 块][nginx#http]的通用性指令都可以放在这个目录中。又比如这样一份自定义的 `/etc/nginx/conf.d/types.conf`（这样可以保护 `mime.types` 文件不被修改）：

[nginx#upstream]: http://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream

```nginx
types {
    text/html do;
}
```

### 1.2 域配置

同样是在系统配置目录，还有两个 `sites-available` 和 `sites-enabled` 子目录。而后一目录中的所有文件，都会被加载。

模仿 [Nginx][] 默认 [VHOST](http://baike.baidu.com/view/7383.htm) 的管理方式，我们可以将域配置文件 `dummy.local` 存放为 `sites-available/dummy.local`，然后[符号链接][]至 `sites-enabled/dummy.local` 使其生效。

但在实际生产过程中，比较常见地情况是 `dummy.local` 会被实际拆分为诸如主域 `dummy.local`、静态资源子域 `s.dummy.local`（static）、业务接口子域 `api.dummy.local` 和数据资源子域 `a.dummy.local`（asset）。而且，在不同的网络节点中，会选择性地部署其中的一个或多个，以均衡负载或容灾。

因此，最可靠的方式是将每个子域都独立配置（成独立的文件）。虽然全部都会存放在 `sites-available` 子目录中，但是否需要[符号链接][]至 `site-enabled` 使其生效，视节点规划而定。

### 1.3 域共用配置

一些情况下，部分配置可能会同时出现在一个主域的不同子域中。如前文中比较个性的文件后缀，又如 [CORS](http://baike.baidu.com/item/CORS/16411212) ，再比如特殊的 [PHP-FPM](http://baike.baidu.com/view/4168033.htm) 配置，等等。

以最粗浅的道理来分析，拆分这些配置有助于提高复用，保障一致性和可靠性。那么它们放在什么地方更合适？

#### 1.3.1 “被调用”的 [`upstream`][nginx#upstream]

[nginx#location]: http://nginx.org/en/docs/http/ngx_http_core_module.html#location

[`upstream`][nginx#upstream] 被 [Nginx][] 定义为**服务器组**（server group），只用于被 [`proxy_pass`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)、[`fastcgi_pass`](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass)、[`uwsgi_pass`](http://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_pass)、[`scgi_pass`](http://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_pass) 和 [`memcached_pass`](http://nginx.org/en/docs/http/ngx_http_memcached_module.html#memcached_pass) 指令所调用。保存在文件 `/etc/nginx/conf.d/upstream-php-dummy.local.conf` 并不会对全局配置造成干扰或污染。

```nginx
upstream php-dummy {
    server unix:/var/run/php5-fpm.sock;
}
```

建议文件以 `<directive>-<purpose>-<root domain>.conf` 地形式三段化命名，便于识别和管理。

#### 1.3.2 “被引用”的其它配置

除上述地 [`upstream`][nginx#upstream] 和 [`location @name`][nginx#location] 之外，其它配置都是需要在域配置的相应位置被**重复引用加载**。所以存放至 `/etc/nginx/sites-available/dummy.local.d/cors.conf` 是个不错地选择，引用也很方便：

```nginx
include sites-available/dummy.local.d/cors.conf;
```

建议目录和文件以 `sites-available/<root domain>.d/<purpose>.conf` 地形式两段命名，便于识别和管理。

### 1.4 子项目配置

在另一些情况下，同一个域的体验表现，其实是由多个项目组合而成。比如：主体是 CMS，而 `/bbs/` 则挂上了 BBS 。后者的配置文件应该放在什么地方以便管理？

我个人的做法，是将其存放为 `/etc/nginx/sites-available/s.dummy.local.d/50-bbs.sub`。这样在域配置文件中所有 [`location`][nginx#location] 之前批量引用即可：

```nginx
include sites-available/s.dummy.local.d/*.sub;
```

建议目录和文件以 `sites-available/<sub-domain>.d/<priority>-<root-uri>.sub` 地形式三段命名。

加上很尴尬地优先级编号地原因，在于 `/bbs/lib/` 子项目的配置一定要在 `/bbs/` 子项目配置之前被引用才能正常工作，但批量引用时 `bbs-lib.sub` 却会晚于 `bbs.sub` 。而有优先级编号之后，我们就可以确保 `05-bbs-lib.sub` 是在 `50-bbs.sub` 之前被引用的。

### 1.5 总结、反向查找与后继维护

此时，[Nginx][] 系统配置目录应该是这样的：

```
/etc/nginx
├── conf.d
│   ├── upstream-php.conf               # 公共配置文件
│   └── upstream-php-dummy.local.conf   # 域 upstream 配置文件
├── sites-available
│   ├── dummy.local                     # 主域配置文件
│   ├── dummy.local.d
│   │   ├── 05-bbs-lib.sub              # 主域子项目配置文件
│   │   ├── 50-bbs.sub                  # 主域子项目配置文件
│   │   └── cors.conf                   # 域共用配置文件
│   ├── s.dummy.local                   # 静态资源子域配置文件
│   └── s.dummy.local.d
│       ├── 05-bbs-lib.sub              # 静态资源子域子项目配置文件
│       └── 50-bbs.sub                  # 静态资源子域子项目配置文件
└── sites-enabled
    ├── dummy.local                     # -> ../sites-available/dummy.local
    └── s.dummy.local                   # -> ../sites-available/s.dummy.local
```

## 二 域代码部署

### 2.0 仿 Linux 目录结构

近几年的时间，我都会参考 Linux 的目录结构，来划分项目代码的目录结构，因为这样最好识别：

```
PROJECT
├── bin                         # 可执行程序
├── etc                         # 配置
│   ├── cron.d                  # 计划任务配置
│   └── nginx                   # 项目 Nginx 配置
│       ├── conf.d              # 被调用的域共用 Nginx 配置
│       └── sites-available     # 域 Nginx 配置
│           ├── ROOT-DOMAIN.d   # 被引用的共用 Nginx 配置和主域子项目 Nginx 配置
│           └── SUB-DOMAIN.d    # 子域子项目 Nginx 配置
├── include                     # 第三方代码库
├── lib                         # 后端程序代码
│   ├── Controller              # 控制器层程序代码
│   ├── Model                   # 模型层程序代码
│   │   └── DAO                 # 数据访问对象程序代码
│   ├── Utility                 # 工具层程序代码
│   └── View                    # 视图层程序代码
│       └── Helper              # 视图组件程序代码
├── libexec                     # XGI 入口程序
├── share                       # 其它程序代码
│   ├── cron                    # 计划任务脚本
│   ├── public                  # 主域静态文件
│   │   ├── .cache              # -> ../../var/cache
│   │   └── .cgi-bin            # -> ../../libexec
│   ├── scss                    # 主域 SCSS 样式代码
│   └── static                  # 静态资源子域文件
└── var                         # 数据
    ├── cache                   # 静态缓存文件
    ├── db                      # 数据库数据
    └── log                     # 日志
```

### 2.1 用好 `/var/www`

在生产节点上部署好之后，就应该是：

```
/var/www
└── dummy.local                         # 域目录
    ├── _bbs.git                        # 子项目代码检出版本库
    │   ├── etc
    │   │   └── nginx
    │   │       └── sites-available
    │   │           ├── dummy.local.d
    │   │           │   └── 50-bbs.sub  # 子项目主域配置文件
    │   │           └── s.dummy.local.d
    │   │               └── 50-bbs.sub  # 子项目静态资源子域配置文件
    │   └── share
    │       ├── public                  # 主域静态文件
    │       └── static                  # 静态资源子域文件
    ├── _main.git                       # 主项目代码检出版本库
    │   ├── etc
    │   │   └── nginx
    │   │       └── sites-available
    │   │           ├── dummy.local     # 主项目主域配置文件
    │   │           └── s.dummy.local   # 主项目静态资源子域配置文件
    │   └── share
    │       ├── public                  # 主项目主域静态文件
    │       │   └── bbs                 # -> ../../../_bbs.git/share/public
    │       └── static                  # 主项目静态资源子域文件
    │           └── bbs                 # -> ../../../_bbs.git/share/static
    ├── @                               # -> _main.git/share/public
    ├── s                               # -> _main.git/share/static
    ├── www                             # -> @
    └── ~log                            # Nginx 日志目录
```

在域目录中，

-   所有 `_` 符号起始的目录都是代码包；
-   `~log` 目录用于记录该域的所有 HTTP 日志，以满足数据挖掘工作所需；
-   其它目录均为子域目录，`@` 遵循 DNS 规约习惯作主域（根域）使用，内部文件和结构与实际体验保持一致。

但是，如果没有主项目代码检出版本库，该怎么办？照着同样的目录结构，建一个名为 `_` 的目录替代就可以了。看着也还算一目了然～

此时对应的 [Nginx][] 系统配置目录文件结构为：

```
/etc/nginx
├── sites-available
│   ├── dummy.local                     # -> /var/www/dummy.local/_main.git/etc/nginx/sites-available/dummy.local
│   ├── dummy.local.d
│   │   └── 50-bbs.sub                  # -> /var/www/dummy.local/_bbs.git/etc/nginx/sites-available/dummy.local.d/50-bbs.sub
│   ├── s.dummy.local                   # -> /var/www/dummy.local/_main.git/etc/nginx/sites-available/s.dummy.local
│   └── s.dummy.local.d
│       └── 50-bbs.sub                  # -> /var/www/dummy.local/_bbs.git/etc/nginx/sites-available/s.dummy.local.d/50-bbs.sub
└── sites-enabled
    ├── dummy.local                     # -> ../sites-available/dummy.local
    └── s.dummy.local                   # -> ../sites-available/s.dummy.local
```
