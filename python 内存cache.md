---
title: python 内存cache
categories: [python]
tags: [python, cache, 缓存]
---

### 我们说的cache
为了缓解数据库的压力, 我们一般都会把热数据放到缓存, 常用的缓存只要有redis, memcache等. 但是像redis, memcache这样的缓存我们真的一定需要需要么. 非也！非也！redis, memcache这样的缓存, 更多的是给无状态的程序使用, 当一个请求过来, 我们启动一个python进程, 处理完成之后, 进程释放, 我们无法保存各种数据的状态, 于是我们引入的redis, memcache等第三方有状态服务, 帮助我们解决这个问题. 可是如果我们程序本身就是有状态的, 这个时候还是用redis做缓存就会显得有些鸡肋了(这个地方不是绝对的, 每个程序有自己需求).
<!--more-->

### 一个简单的内存缓存
```
#coding=utf-8
from time import time
class Cache:
    '''简单的缓存系统'''
    def __init__(self):
        '''初始化'''
        self.mem = {}
        self.time = {}

    def set(self, key, data, age=-1):
        '''保存键为key的值，时间位age'''
        self.mem[key] = data
        if age == -1:
            self.time[key] = -1
        else:
            self.time[key] = time() + age
        return True

    def get(self,key):
        '''获取键key对应的值'''
        if key in self.mem.keys():
            if self.time[key] == -1 or self.time[key] > time():
                return self.mem[key]
            else:
                self.delete(key)
                return None
        else:
            return None

    def delete(self,key):
        '''删除键为key的条目'''
        del self.mem[key]
        del self.time[key]
        return True

    def clear(self):
        '''清空所有缓存'''
        self.mem.clear()
        self.time.clear()
# 代码来自：https://github.com/ma6174/pycache/blob/master/cache.py
```
### 内存cache
说白了就是一个变量, 我们把这个变量保存在内存里, 同时给它一个过期时间, 过期则失效.

### cache失效策略
懒惰失效, 去取数据的时候才失效. redis事实上也是用这种失效模式, 用空间换取性能.
