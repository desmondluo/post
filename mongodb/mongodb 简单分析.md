---
title: mongodb 简单分析
tags: [mongodb]
categories: [mongodb]
---

### mongostat
```
mongostat -h 127.0.0.1:9810
# ids miss 查询没有走索引的情况
```

### profile 集合
慢操作日志, 合理使用profile
```
use test;
db.getProfilingLevel();
db.getProfilingStatus();

db.setProfilingLevel(2);  // 所有操作集合

db.system.profile.find().sort({$natural:-1}).limit(10);
```

### mongod 日志
```
cd conf
vim mongod.conf

verbose = vvvvv // 保存日志的级别, 1-5个v, 级别越高, 日志越详细
```

### explain
```
db.test.find({x:3}).explain()  //详细查询数据
```

### 状态查看
```
rs.status()                    // 复制集状态查询
rs.printReplicationInfo()      // 查看oplog状态
rs.printSlaveReplacationInfo() // 查看复制延迟
db.serverStatus()              // 查看状态详情
```

### rs.status()
- self 只会出现在rs.status()命令的成员里面
- uptime 从本节点网络可达到当前所经历的时间
- lastHeartbeat 当前服务器最后一次收到其心跳的时间
- optime & optimeDate 命令发出是oplog所记录的操作的时间戳
- pingMs 网络延迟
- syncingTo 复制源

sateStr 服务状态
- 可提供服务状态： PRIMARY, SECONDARY, ARBITER
- 即将提供服务状态: STARTUP, STARTUP2, RECOVERING
- 不可提供服务状态: DOWN, UNKNOW, REMOVED, ROLLBACK, FATAL

### rs.printReplicationInfo()
- log length start to end oplog 的时间窗口
- oplog first event time 开始时间
- oplog end event time 结束时间
- now 现在的时间

### rs.printSlaveReplacationInfo
- syncedTo 复制进度
- X secs (XX) behind the primary 落后进度

### 监控项目
- QPS: 每秒查询数量
- I/O: 读写性能
- Memory: 内存使用
- Connections: 连接数
- Page Faults: 缺页中断(要查的东西, 不在内存, 要跑硬盘)
- Index hit: 索引命中率
- Background flush: 后台刷新
- Queue: 队列

### mongostat
重点字段

- getmore 大量排序操作正在进程
- faults 需要的数据不在内存中
- locked db 锁比率最好的库
- inedx miss 索引未命中
- qr|qw 读写产生队列, 供求失衡
