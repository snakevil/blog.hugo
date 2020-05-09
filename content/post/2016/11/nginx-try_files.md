---
title: "Nginx try_files 的美感"
slug: "Nginx-try_files的美感"
date: 2016-11-15T02:04:05Z
lastmod: 2016-11-21T07:49:05Z
toc: true
category:
    - 开发技巧
tag:
    - Nginx
---

[Nginx][nginx] `try_files` 指令自实现之日启，风靡至今。单纯说功能实现地话，确实能够快速地适配各种需要场景，省掉了大段的 `if` 和 `set`。但相应的 URL 拼写格式，就真地不忍直视了。

[nginx]: http://nginx.org

<!--more-->

## 丢失的 _301_ 跳转

比如这样一段最基本的配置，

```nginx
location / {
    try_files $uri $uri/index.html;
}
```

就会导致 `GET /foo` 请求也能直接访问到 `foo/index.html` 文件。尽管似乎省掉了一次传统的 _301_ 跳转，让响应变得更快。但给我的感觉，却像是一桌川菜里突然混进了一盘糖醋里脊，尴尬得不行。

正因为 HTTP 标准教育给我的美学中，不带 `/` 访问一个目录，就 **应该** 先做 _301_ 跳转，再去尝试访问默认索引文件。

当然上面的那个例子，其实纯粹是为了展现问题。不写这句 `try_files`，也能正常工作。

## 退路（fallback）机制

但我们实际使用 `try_files` 上，为了避免让用户看到 [Nginx][nginx] 默认的出错页面，一定会加上一个退路机制。比如：

```nginx
location / {
    try_files $uri $uri/index.html @fallback;
}
```

其预期地访问策略为：

1. 目录中存在地文件；
2. 目录中存在地子目录的 `index.html`；
3. 其它退路机制。

如果不使用 `try_files`，那么能用地指令就剩下 `error_page` 了。对应的配置应该是：

```nginx
location / {
    error_page 403 404 = @fallback;
}
```

## 多目录如何遍历

到了实际的项目中，我们会发现需要被访问地文件其实是分布在多个目录中的。比如静态文件目录 `share/public` 和静态缓存文件目录 `share/public/.cache` （指向 `var/cache`），如果业务更复杂，或许还需要更多的目录。

比较常见的访问策略应该如下：

1. 某目录中存在地文件；
2. 某目录中存在且拥有 `index.html` 地目录；
3. 另一目录中存在地文件；
4. 另一目录中存在且拥有 `index.html` 地目录；
5. ...
6. 其它退路（fallback）机制。

简单实践了一番，找到了一种似乎行之有效地配置方法：

```nginx
location ~^ /.another/ {
    internal;
    error_page 403 404 = @fallback;
}

location ~^ /.cache/ {
    internal;
    recursive_error_pages on;
    error_page 403 404 = /.another$request_uri;
}

location / {
    recursive_error_pages on;
    error_page 403 404 = /.cache$request_uri;
}
```

**注意**，使用这种配置方法时需非常注意 `internal` 和 `recursive_error_pages` 的限制。

## `internal` 的次数限制

[Nginx][nginx] 在[文档](http://nginx.org/en/docs/http/ngx_http_core_module.html#internal)中特别指出：

> 每个请求都有天然的 _10_ 次内部（`internal`）跳转地限制，以免在配置有误时导致请求处理环路（request processing cycle）。当达到上限时，会以 _500_ 错误中止。此时，可以在错误日志中看到“rewrite or internal redirection cycle”字样的信息。

所以，使用上一节中的配置方法，最多可以配置 _10_ 个分布目录。超出则需要在 `location @name` 上做文章了。

不过就我个人经验而言，_10_ 个目录已经是绰绰有余了。还要再多…这种变态需求还是再想想有没有其它更有效地方法吧。

## `recursive_error_pages` 的次数限制

[Nginx][nginx] 在[文档](http://nginx.org/en/docs/http/ngx_http_core_module.html#recursive_error_pages)中描述得很简略，看不出如何使用，也似乎没有任何限制。

但我通过 [DockerServer](https://www.docker.com)/1.12.1 [Debian](https://www.debian.org)/8 [Nginx][nginx]/1.6.2 的反复测试，找到了三点规律：

1. 从 `error_page` 声明处开始生效；

    以上上节中的案例加以说明，请求首先命中地是 `location /`。因此要在其中声明 `recursive_error_pages`，后继 `location ~^ /.cache/` 的 `error_page` 才能生效。

2. 只支持 _2_ 次跳转；

    仍然以上上节中的案例加以说明，`location /` 的 `recursive_error_pages`，能够确保请求处理正确地内部跳转至 `location ~^ /.another/`。

    但 `location ~^ /.another/` 中的 `error_page` 不会生效。即 `location @fallback` 不工作。

3. 每个 `location` 路径中都需要 `recursive_error_pages`。

    跳过 `location ~^ /.cache/`，直接在 `location ~^ /.another/` 中声明 `recursive_error_pages`，`location @fallback` 不工作。

    在 `location ~^/.cache/` 中声明，`location @fallback` 工作。

## `REQUEST_METHOD` 被改变地问题

在实践过程中发现，经过 `error_page` 指令 `internal` 跳转至 `location @fallback` 时，`REQUEST_METHOD` 数据被强制从 `POST` 改为了 `GET`。

不过解决这一问题并不困难，对上述配置方案进行简单地调整即可：

```nginx
location ~^ /.cgi-bin/fcgi.php/ {
    internal;
    include fastcgi_params;
    fastcgi_param REQUEST_METHOD $original_method;
    # ...
}

location ~^ /.another/ {
    internal;
    error_page 403 404 = /.cgi-bin/fcgi.php$request_uri;
}

location ~^ /.cache/ {
    internal;
    recursive_error_pages on;
    error_page 403 404 = /.another$request_uri;
}

location / {
    set $original_method $request_method;
    recursive_error_pages on;
    error_page 403 404 = /.cache$request_uri;
}
```
