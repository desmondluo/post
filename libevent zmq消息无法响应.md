---
title: libevent zmq消息无法响应
tags: [libevent, zmq, 消息队列, 事件循环]
categories: [C++]
---

### 前言
服务器使用libevent做事件循环, 然后使用zmq做线程间通讯, libevent监听zmq的消息, 一旦zmq有消息过来, 就会回调到固定的函数。使用zmq需要注意的是, 一旦zmq消息回调, 必须读空整个队列的消息. 否则下次就不会回调。

### 问题症状
我们使用libevent监听了多个zmq, 同时监听了别的事件(退出事件等). 其中有一个zmq消息非常繁忙， 每秒会产生上万到十万不等的数据. 这种时候会发现libevent再也无法响应其他消息。

```
// 使用libevent监听退出事件
int fd = eventfd(0, 0);
set_nonblocking(fd);
event* pEvent = event_new(m_ebase, fd, EV_READ | EV_PERSIST, cb, pthis);
event_add(pEvent, NULL);
// 使用libevent监听zmq事件
event_new(m_ebase, m_cmdSubSocket.get_fd(), service_base::on_command, this);
event_add(pEvent, NULL);

// 当zmq有消息回调(伪代码)
while(zmq.hasmsg())
{
  msg = zmq.nextmsg();
  // 处理消息
}
```

### 问题分析
当zmq消息非常频繁的时候, 假设所有的消息都进入排队状态, 那么如果后面的消息进入队列肯定是在队列尾部, 这种情况下理论上会导致退出消息非常慢, 但是不会出现消息无法响应的情况。但是为了排除是这种情况，我们使用libevent的有限级。

```
// 首先给消息设置优先级
event_base_priority_init(m_base, EVENT_MAX_PRIORITIES  - 1);
// 给退出事件设置高优先级
event* pEvent = event_new(m_ebase, fd, EV_READ | EV_PERSIST, cb, pthis);
event_priority_set(pEvent, 0);
event_add(pEvent, NULL);
// 给zmq事件设置低优先级
event* cmd = event_new(m_ebase, m_cmdSubSocket.get_fd(), service_base::on_command, this);
event_priority_set(cmd, EVENT_MAX_PRIORITIES - 1);
event_add(pEvent, NULL);
```

结果证明没什么卵用, 问题依然存在

### 问题解决
在咨询同事和查阅相关资料后发现, zmq给出一个消息队列读的时候, 给出的是一个原始的消息队列，当你在队列头读取的时候, zmq线程依然在往队列尾写入, 如果我们读取的速度小于写入的速度, 那么zmq队列将一直有消息, 所以上面的代码while循环，将一直成立。解决方案就是让读取的速度大于写入的速度, 在读取的时候，就不要去处理消息了.
```
// 这个地方永远成立, 导致libevent无法响应其他消息
while(zmq.hasmsg())
{
  msg = zmq.nextmsg();
  // 处理消息
}

// 解决方案
std::vector<msgtype> vmsg;
while(zmq.hasmsg())
{
  vmsg.pushback(zms.nextmsg());
}
// 后面处理
dealmsg(vmsg)
```
