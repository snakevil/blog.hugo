---
title: "GNUMake 使用静态模式规则"
slug: "GNUMake使用静态模式规则"
date: 2017-04-07T04:43:00Z
category:
    - Linux
    - 开发技巧
tag:
    - GNUMake
    - 系统构建
    - MoonScript
    - Lua
---

最近在写 [Luadex2][] 时，顺手就用 [GNUMake][] 来做构建工具了——这样还能热热手把好久好久没用地 [GNUMake][] 再捡回来。中间碰到一个问题，我需要动态地扫描出全部的某类型的文件，然后按照同样的规则进行处理。

[luadex2]: https://github.com/snakevil-archive/luadex2
[gnumake]: http://www.gnu.org/software/make/

<!--more-->

## 1 扫描文件

如果同类型的文件分布在同一目录中，那照着 [GNUMake][] 手册中的范例写就可以搞定。

```makefile
%.o: %.c
	# ...
```

但实际上当我需要手写 [GNUMake][] 时，项目的目录结构通常都比较复杂了。所以才需要使用变量来处理：

```makefile
LUA := $(patsubst %.moon, \
	$(DESTDIR)/%.lua, \
	nginx.moon $(foreach dir, \
		lib lib/model lib/view, \
		$(wildcard $(dir)/*.moon) \
	) \
)
```

上面的代码是从 [Luadex2][] [Makefile][gnumake] 中复制出来的，用于扫描 `lib`、`lib/model` 和 `lib/view` 目录中的所有 [MoonScript][] 文件，**并将路径转化为目标文件**。

[moonscript]: http://moonscript.org

## 2 静态模式规则

将所有扫描结果转化为目标文件，是因为 [GNUMake][] 的基本工作机制是 *目标（Target）*驱动的。我们需要先知道究竟应该产出哪些文件。

有了这个目标列表之后，就可以使用[静态模式规则](http://www.gnu.org/software/make/manual/html_node/Static-Usage.html#Static-Usage)这一大杀器了。

```makefile
$(LUA): $(DESTDIR)/%.lua: %.moon
	mkdir -p $(dir $@)
	moonc -p $^ | luajit -b - - > $@
```

这个例子同样是 [Luadex2][] 中的代码。

[GNUMake][] 在处理时，会遍历 `$(LUA)`，生成符合 `$(DESTDIR)/%.lua: %.moon` 要求的规则，比如：`$(DESTDIR)/nginx.lua: nginx.moon`。问题一下子就非常简单了！

按照实际需要编写 _“配方”（Recipe）_ 即可。
