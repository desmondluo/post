---
title: tornado pyzmq使用
categories: [tornado, pyzmq]
tags: [tornado, pyzmq, ioloop]
---

### 共用事件流
[共用事件流](http://learning-0mq-with-pyzmq.readthedocs.io/en/latest/pyzmq/multisocket/tornadoeventloop.html) ，这篇文档讲了我们应该怎样使用同一个事件流.

### 为什么要用tornado + zmq
#### tornado 多进程异步处理
众所周知tornado可以用来做webserver， 那么我们肯定不希望我们用户等待太久, 如果我们再tornado与用户打交道的进程里面处理太多的业务, 势必会影响到用户体验, 所以一般我们都会将耗时的业务放到另一个进程, 然后讲结果投递回来, 我们再发送给用户, 这样我们既能保证用户体验, 同时又能保证较大的计算量.

#### pyzmq 基于消息流的rpc通信框架
[zmq](http://zguide.zeromq.org/page:all) 是一个高效的异步rpc通信框架, pyzmq提供了一套异步接口, 完善的异步接口, 并且兼容tornado(共用ioloop), 也就是说我们再异步处理zmq的时候，同时可以异步处理tornado事件. (zmq是一个非常强悍的rpc通信组件, 在python下被阉割了很多功能, 但是还是非常好用, 这里我们仅仅用来做进程间的通信组件)

tornado+多进程作为高效的业务处理框架, zmq作为框架内进程间的通信方式.

### 代码演示
- tornado 使用三个进程
- zmq 使用pub + sub 模式

#### zmqconfig.py
```
#-*- coding: utf-8 -*-
import zmq

context = zmq.Context()
one_zmq_addr = "ipc://one_zmq_addr"
one_to_two_subject = "one_to_two_subject"
one_to_three_subject = "one_to_three_subject"

two_zmq_addr = "ipc://two_zmq_addr"
two_to_one_subject = "two_to_one_subject"
two_to_three_subject = "two_to_three_subject"

three_zmq_addr = "ipc://three_zmq_addr"
three_to_one_subject = "three_to_one_subject"
three_to_two_subject = "three_to_two_subject"
```
这里我们定义了三个进程zmq的通信地址, 每个地址与另外两个地址发布订阅的主题

#### oneservice.py
```
# coding: UTF-8
import time
import tornado
import tornado.ioloop
import tornado.iostream
import tornado.web
import json
import zmq
from urls import urls
from config import zmqconfig
from zmq.eventloop import ioloop, zmqstream
from zmq.eventloop.ioloop import IOLoop
from zmq.eventloop.ioloop import ZMQIOLoop
#ioloop.install()
#application = tornado.web.Application(urls)
class service:

    def __init__(self):
        self.ioloop = ZMQIOLoop()
        self.ioloop.install()
        return

    def process_message_two(self, msg):
        print "get thread two message"
        print "processing .....", msg
        return

    def process_message_three(self, msg):
        print "get thread three message"
        print "processing......", msg
        return

    def timeout(self):
        print "thread one timeout"
        data = {}
        data['thread'] = 'one'
        self.socket_to_others.send(zmqconfig.one_to_two_subject, zmq.SNDMORE)
        self.socket_to_others.send(json.dumps(data))
        self.socket_to_others.send(zmqconfig.one_to_three_subject, zmq.SNDMORE)
        self.socket_to_others.send(json.dumps(data))
        self.ioloop.add_timeout(time.time() + 3, self.timeout)
        return

    def run(self):
        self.socket_to_others = zmqconfig.context.socket(zmq.PUB)
        self.socket_to_others.bind(zmqconfig.one_zmq_addr)
        self.socket_from_two = zmqconfig.context.socket(zmq.SUB)
        self.socket_from_two.connect(zmqconfig.two_zmq_addr)
        self.socket_from_two.setsockopt(zmq.SUBSCRIBE, zmqconfig.two_to_one_subject)
        self.stream_from_two_sub = zmqstream.ZMQStream(self.socket_from_two)
        self.stream_from_two_sub.on_recv(self.process_message_two)
        self.socket_from_three = zmqconfig.context.socket(zmq.SUB)
        self.socket_from_three.connect(zmqconfig.three_zmq_addr)
        self.socket_from_three.setsockopt(zmq.SUBSCRIBE, zmqconfig.three_to_one_subject)
        self.socket_from_three_sub = zmqstream.ZMQStream(self.socket_from_three)
        self.socket_from_three_sub.on_recv(self.process_message_three)
        self.ioloop.add_timeout(time.time(), self.timeout)
        application = tornado.web.Application(urls)
        application.listen(8887)
        self.ioloop.start()
        return
```
为了更加贴合需求, 这里用了webserver, 表示这条线程是直接与用户打交道的.

twoservice.py
```
# coding: UTF-8
import time
import tornado
import tornado.ioloop
import tornado.iostream
import json
import zmq
from config import zmqconfig
from zmq.eventloop import ioloop, zmqstream
context = zmq.Context()
ioloop.install()

class service:

    def __init__(self):
        self.ioloop = ioloop.IOLoop().instance()
        return

    def process_message_one(self, msg):
        print "get thread one message"
        print "processing .....", msg
        return

    def process_message_three(self, msg):
        print "get thread three message"
        print "processing......", msg
        return


    def timeout(self):
        print "thread two timeout"
        data = {}
        data['thread'] = 'two'
        self.socket_to_others.send(zmqconfig.two_to_one_subject, zmq.SNDMORE)
        self.socket_to_others.send(json.dumps(data))
        self.socket_to_others.send(zmqconfig.two_to_three_subject, zmq.SNDMORE)
        self.socket_to_others.send(json.dumps(data))
        self.ioloop.add_timeout(time.time() + 3, self.timeout)
        return

    def run(self):
        self.socket_to_others = zmqconfig.context.socket(zmq.PUB)
        self.socket_to_others.bind(zmqconfig.two_zmq_addr)
        self.socket_from_one = zmqconfig.context.socket(zmq.SUB)
        self.socket_from_one.connect(zmqconfig.one_zmq_addr)
        self.socket_from_one.setsockopt(zmq.SUBSCRIBE, zmqconfig.one_to_two_subject)
        self.stream_from_one_sub = zmqstream.ZMQStream(self.socket_from_one)
        self.stream_from_one_sub.on_recv(self.process_message_one)
        self.socket_from_three = zmqconfig.context.socket(zmq.SUB)
        self.socket_from_three.connect(zmqconfig.three_zmq_addr)
        self.socket_from_three.setsockopt(zmq.SUBSCRIBE, zmqconfig.three_to_two_subject)
        self.socket_from_three_sub = zmqstream.ZMQStream(self.socket_from_three)
        self.socket_from_three_sub.on_recv(self.process_message_three)
        self.ioloop.add_timeout(time.time(), self.timeout)
        self.ioloop.start()
        return
```
#### threeservice.py
```
# coding: UTF-8
import time
import tornado
import tornado.ioloop
import tornado.iostream
import json
import zmq
from config import zmqconfig
from zmq.eventloop import ioloop, zmqstream
context = zmq.Context()
ioloop.install()

class service:

    def __init__(self):
        self.ioloop = ioloop.IOLoop().current(True)
        return


    def process_message_one(self, msg):
        print "get thread one message"
        print "processing .....", msg
        return

    def process_message_two(self, msg):
        print "get thread two message"
        print "processing .....", msg
        return

    def timeout(self):
        print "thread three timeout"
        print "thread two timeout"
        data = {}
        data['thread'] = 'three'
        self.socket_to_others.send(zmqconfig.three_to_one_subject, zmq.SNDMORE)
        self.socket_to_others.send(json.dumps(data))
        self.socket_to_others.send(zmqconfig.three_to_two_subject, zmq.SNDMORE)
        self.socket_to_others.send(json.dumps(data))
        self.ioloop.add_timeout(time.time() + 3, self.timeout)
        return

    def run(self):
        self.socket_to_others = zmqconfig.context.socket(zmq.PUB)
        self.socket_to_others.bind(zmqconfig.three_zmq_addr)
        self.socket_from_two = zmqconfig.context.socket(zmq.SUB)
        self.socket_from_two.connect(zmqconfig.two_zmq_addr)
        self.socket_from_two.setsockopt(zmq.SUBSCRIBE, zmqconfig.two_to_three_subject)
        self.stream_from_two_sub = zmqstream.ZMQStream(self.socket_from_two)
        self.stream_from_two_sub.on_recv(self.process_message_two)
        self.socket_from_one = zmqconfig.context.socket(zmq.SUB)
        self.socket_from_one.connect(zmqconfig.one_zmq_addr)
        self.socket_from_one.setsockopt(zmq.SUBSCRIBE, zmqconfig.one_to_three_subject)
        self.stream_from_one_sub = zmqstream.ZMQStream(self.socket_from_one)
        self.stream_from_one_sub.on_recv(self.process_message_one)
        self.ioloop.add_timeout(time.time(), self.timeout)
        self.ioloop.start()
        return
```

#### server.py
```
import tornado
import tornado.ioloop
import tornado.web
import time
import multiprocessing
from threadone import service as oneservice
from threadtwo import service as twoservice
from threadthree import service as threeservice
import zmq
context = zmq.Context()

def timeout():
    print "timeout"
    return

def one():
    one = oneservice.service()
    one.run()
    return

def two():
    two = twoservice.service()
    two.run()
    return

def three():
    three = threeservice.service()
    three.run()
    return


if __name__ == "__main__":
    serviceone = multiprocessing.Process(target=one, args=())
    servicetwo = multiprocessing.Process(target=two, args=())
    servicethree = multiprocessing.Process(target=three, args=())
    servicethree.start()
    servicetwo.start()
    serviceone.start()

    for p in multiprocessing.active_children():
        print("child   p.name:" + p.name + "\tp.id" + str(p.pid))
```

### 详解
每个进程我们都有3个socket, 一个对外的pub, 两个监听外部输入的socket
```
self.socket_to_others = zmqconfig.context.socket(zmq.PUB)
self.socket_to_others.bind(zmqconfig.three_zmq_addr)
self.socket_from_two = zmqconfig.context.socket(zmq.SUB)
self.socket_from_two.connect(zmqconfig.two_zmq_addr)
self.socket_from_two.setsockopt(zmq.SUBSCRIBE, zmqconfig.two_to_three_subject)
self.stream_from_two_sub = zmqstream.ZMQStream(self.socket_from_two)
self.stream_from_two_sub.on_recv(self.process_message_two)
self.socket_from_one = zmqconfig.context.socket(zmq.SUB)
self.socket_from_one.connect(zmqconfig.one_zmq_addr)
self.socket_from_one.setsockopt(zmq.SUBSCRIBE, zmqconfig.one_to_three_subject)
self.stream_from_one_sub = zmqstream.ZMQStream(self.socket_from_one)
self.stream_from_one_sub.on_recv(self.process_message_one)
self.ioloop.add_timeout(time.time(), self.timeout)
```
外部两个输入的时候, 都会回调到不同的地址, 这样实现异步

### 结果
![](http://ww4.sinaimg.cn/large/005OdUDHgw1f6lnbj5mddj30mz0hswlo.jpg)
虽然所有进程的消息都达到一起了, 但是大致可以看出来每个进程都正常工作!

### 总结
tornado + multiprocessing + pyzmq 性能非常好, 效率也非常高, 用得好基本上能秒杀一般的python框架了.

最后放上代码[github地址](https://github.com/desmondluo/tornado_zmq_example) Have Fun!
