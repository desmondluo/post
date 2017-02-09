---
title: mongodb 简单搭建复制集
tags: [mongodb, replset]
categories: [mongodb]
---

### 复制集配置文件参数
|字段|类型|取值|解释|
|:--- |:--- |:--- |:--- |
|id|   整数| 从0开始|编号|
|host| 字符串| 192.168.0.235:9990|IP:Port|
|arbiterOnly|布尔值|trre/false|是否投票结点|
|priority|整数|0-1000|权重, 数字越大, 越能成为主节点, 0表示永远不会成为主节点|
|hidden|布尔值|true/false, 0/1|是否是隐藏结点, priority=0|
|votes|整数|0/1|是否能够投票|
|slaveDelay|整数|x秒(3600)|延迟结点(用来保证数据安全,如果主库删了,从库都会同步删)|
|buildIndexes|布尔值|true/false|是否同步建立索引,priority=0|

### 结点属性
- 主节点 priority 至少为1
- 从节点 priority 可以为0
- 延迟结点 priority 为0 并且hidden = true, slaveDelay = XX
- 隐藏结点 priority 为0 并且hidden = true(isMaster的时候, 不会发现)
- 无索引 priority 为0 并且buildIndexes = false

### help
当重新配置文件的时候, 应该查看一下版本号, 保证生效
