---
title: mongodb 安全
tags: [mongodb]
categories: [mongodb]
---

### auth 与 keyfile
auth 开启
```
vim conf/mongo.conf
auth = true
```
创建用户
```
db.create({user: "test", "pwd": "test", roles:[{role:"userAdmin", db:"admin"}]})
// 登录
mongo 127.0.0.1:9810 -u test -p test
```
权限
```
1: 数据库角色(read, readWrite, dbAdmin, dbOwner, userAdmin)
2: 集群角色(clusterAdmin, clusterManager..)
3: 备份角色(backup, restore..)
4: 其他(DBAdminAnyDatabase..)
```
