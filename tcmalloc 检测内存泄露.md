---
title: tcmalloc 内存泄露
categories: [工具, tcmalloc]
tags: [google, tcmalloc]
---

### 介绍
简单的说一下怎么用 tcmalloc 检测内存泄露. [tcmalloc](https://google-perftools.googlecode.com/svn/trunk/doc/heap_checker.html)  

### tcmalloc
tcmalloc[http://goog-perftools.sourceforge.net/doc/tcmalloc.html] 是google的C++开源项目之一, 解决了 glibc malloc造成的内存增长. 同时提供了一套工具让我们分析内存泄露, 内存增长分析, CPU占用分析.

### 简单使用
```
env HEAPCHECK=normal ./server
```
