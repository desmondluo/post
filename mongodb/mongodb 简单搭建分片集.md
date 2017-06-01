---
title: mongodb 简单搭建分片集
tags: [mongodb, shared]
categories: [mongodb]
---

### 分片目的
- 改善单机机器数据数据的存储及数据吞吐新能
- 提高在大量数据下随机访问性能

### 成员结点
- Shard节点: 存储数据的节点(单个mongod或是副本集)
- Config Server: 存储元数据, 为mongos服务, 将数据路由到Shard
- Mongos: 接入前端请求, 进行路由

### 启动服务
- shard启动: mongod --shardsvr -f conf/shard.conf
- config启动: mongod --configsvr -f conf/config.conf
- mongod启动: mongos -f conf/config.conf

### 添加分片
- 连接到mongos
- Add Shards
- Enable Sharding
- 对一个集合进行分片

### Add Shards
