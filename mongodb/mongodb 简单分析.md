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
