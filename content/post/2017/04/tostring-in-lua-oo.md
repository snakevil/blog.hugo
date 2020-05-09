---
title: "Lua 面向对象式开发的 __tostring 问题"
slug: "Lua面向对象式开发的tostring问题"
date: 2017-04-07T03:31:04Z
category:
    - 开发技巧
tag:
    - Lua
---

至 [LuaJIT-2.0.4 为止](http://luajit.org)，`tostring()` 函数都只会检查并调用**元表**的 `__tostring` 函数。那么在面向对象式开发时，如何让基类定义地统一 `__tostring` 机制生效？

<!--more-->

```lua
print(setmetatable({
    __tostring = function ( self )
        return '3'
    end
}, setmetatable({
    __tostring = function ( self )
        return '2'
    end
}, {
    __tostring = function ( self )
        return '1'
    end
})))
```

在上面的案例中，输出内容是 `2`。如果将该函数定义删除，输出内容就变成了 `table: 0x01c773b8` 这样的值。

转以面向对象的思维来理解，`3` 是类的实例，`2` 是类，`1` 是父类。

因此为了能够使用本基类统一的 `__tostring` 方法，在定义每个派生类时，都需要显性地定义派生类中的方法，使其能逐层递归调用至基类。
