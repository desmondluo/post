---
title: zmq 踩坑.md
categories: [C++, zmq]
tags: [C++, zmq]
---
zmq 作为一个基础的消息组件, 基本上是我们做通信的唯一选择, 事实上也是如此. 这里略记一下刚踩到的一个坑.
<!--more-->
### pub sub 消息订阅
订阅一个主题的时候我们可能会有如此写法
```
const char* const subject = "subject";
zmq_setsockopt (socket, ZMQ_SUBSCRIBE, subject, strlen(subject) + 1);
```

上面代码是错误的, 正确应该这样写
```
const char* const subject = "subject";
zmq_setsockopt (socket, ZMQ_SUBSCRIBE, subject, strlen(subject));
```

差别只有一个+1

一个正常的send写法
```
rc = zmq_send (socket, &part1, ZMQ_SNDMORE);
rc = zmq_send (socket, &part2, ZMQ_SNDMORE);
....
rc = zmq_send (socket, &partn, 0);
```

python的写法

```
socket.send("subject message");
```

我们会发现subject主题与message只用了一个空格表示, 这样就能正确的找到路由, 经验发现通过第一种方法, 我们是无法接受到python发送的数据的, 我们猜测主题订阅的时候, 是通过二进制匹配的, 实际上我们订阅的主题应该是subject\0. 所以第一种写法是错误的.

### 有趣的事情
如果订阅和发布两边都是用C++写的, 我们采用第一种方法却不会出问题.