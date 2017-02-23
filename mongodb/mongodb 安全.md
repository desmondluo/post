---
title: mongodb 安全
tags: [mongodb]
categories: [mongodb]
---

### auth
auth 开启(单点验证)
```
vim conf/mongo.conf
auth = true
```
创建用户
```
use admin;
db.createUser(
  {
    user: "admin",
    "pwd": "admin",
    "roles":[
      {role:"root", db:"admin"}
     ]
  }
  )
// 登录
mongo 127.0.0.1:9810 -u admin -p admin
//或者
mongo 127.0.0.1:9810
use admin;
db.auth('amdin', 'admin')
```
权限
```
1: 数据库角色(read, readWrite, dbAdmin, dbOwner, userAdmin)
2: 集群角色(clusterAdmin, clusterManager..)
3: 备份角色(backup, restore..)
4: 其他(DBAdminAnyDatabase..)
```
### keyfile
- 内容 base64编码集
- 长度 1000bytes
- 权限 chmod 600 keyFile

keyfile 开启
vim conf/mongo.conf
keyFile=/home/des/dev/conf/.keyfile
