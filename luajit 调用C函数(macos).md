---
title: luajit 调用C函数
tags: [luajit, macos]
categories: [luajit]
---
### 前言
一个同事在写nginx的lua扩展, 遇到了性能瓶颈, 于是想用C函数解决, 求助与我, 无奈我也不会lua, 只好硬着头皮上. 网上大部分博文都说怎么调用C的库函数, 却没有明确的说怎么调用自定义函数, 并且部分博文以centos, 作为平台. 无奈macos编译方法不一样, 初学者很容易踩坑, 这里简单的演示一下luajit 怎么调用C的自定义函数.

### C自定义函数
```
void add(int x, int y)
{
    return x + y;
}
```
<!--more-->
### lua脚本
```
local ffi = require("ffi")
ffi.load('demo', true)
ffi.cdef[[
    int add(int x, int y);
]]

print(ffi.C.add(1, 2));
```

### 编译
这里用加-dynamic 然后生成.dylib文件
```
gcc -dynamic -o libdemo.dylib -shared demo.c
```

### 运行
```
luajit demo.lua
```
![](http://ww4.sinaimg.cn/large/005OdUDHgw1f2wpedpt57j30bd01xdfx.jpg)

参考 [地址](http://blog.csdn.net/alexwoo0501/article/details/50636785)
