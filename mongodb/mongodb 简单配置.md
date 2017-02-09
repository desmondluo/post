---
title: mongodb 简单配置
tags: [mongodb]
categories: [mongodb]
---

### 初始化配置
```
mkdir mongodbtest   // 工作目录
cd mongodbtest
mkdir data          // 数据目录
mkdir conf          // 配置目录
mkdir log           // log目录
```

### 配置文件
```
cd conf
vim mongod.conf

port = 9810               // 启动端口
dbpath = data             // 数据保存路径, 相对路径或是绝对路径
logpath = log/mongod.log  // 日志文件, 必须用文件名称, 不单单是路径
fork = true               // 是否是后台进程
```

### 启动
```
mongod -f conf/mongod.conf
```

### 连接
```
mongo 127.0.0.1:9810/test

# 关闭
use admin              // 切换权限
db.shotdownServer()    // 关闭数据库
```
