---
title: tornado 使用多线程
categories: [tornado, threading]
tags: [tornado, ioloop, threading]
---

### 前言
tornado 线程支持非常不好, 我们更应该使用多进程, 而不是多线程. tornado 里面ioloop是一个单例, 也就是说全局只能有一个ioloop。
<!--more-->

### 传统的多线程
为了让线程能并发处理更多事情, 我们会让每个线程用有独立的ioloop, 如果我们用libevent， 我们会为每一个线程创建一个event_base。

![](http://ww2.sinaimg.cn/large/005OdUDHgw1f6lj75nn9zj30fb0a0q3b.jpg)

### tornado 如果我们要用多线程
我们应该在主线程里面处理异步io事件, 然后发送给各个独立线程, 最后获取处理结果.

![](http://ww1.sinaimg.cn/large/005OdUDHgw1f6lreturnjefvaw4j30ft09paae.jpg)

### 上一段错误的代码
尝试使用多线程
```
import tornado
import tornado.ioloop
import tornado.web
import threading
import time
import os
import ctypes

class ThreadService:
    def timeout(self):
        print 'Thread ' + str(ctypes.CDLL('libc.so.6').syscall(186)) + ' TimeOut'
        self.ioloop.add_timeout(time.time() + 3, self.timeout)

    def run(self):
        print 'Thread ' + str(ctypes.CDLL('libc.so.6').syscall(186))
        tornado.ioloop.IOLoop.instance().make_current()
        self.ioloop = tornado.ioloop.IOLoop.current()
        self.ioloop.add_timeout(time.time(), self.timeout)
        #self.ioloop.start()

def onethread():
    one = ThreadService()
    one.run()
    return

def twothread():
    two = ThreadService()
    two.run()
    return

if __name__ == "__main__":
    print 'Main Thread ' + str(os.getpid())
    threadone = threading.Thread(target=onethread, args=())
    threadtwo = threading.Thread(target=twothread, args=())
    threadone.start()
    threadtwo.start()
    tornado.ioloop.IOLoop.instance().start()
    threadone.join()
    threadtwo.join()
    # ioloop.IOLoop().instance().start()
    # threadone.join()
    # threadtwo.join()
    # threadthree.join()
```
打印结果

![](http://ww2.sinaimg.cn/large/005OdUDHgw1f6ljlqwvotj30de06nq4p.jpg)

我们会发现, 最后同过ioloop回调回来的都是一条线程回调回来的, 事实上这个地方有三个线程, 只用一个线程拥有了ioloop的控制权, 最后回调回来的时候, 都是通过那条线程回来的。

### 我们更应该使用多进程
下一篇博客, tornado pyzmq, 使用多进程异步方式, 让程序更高效.
