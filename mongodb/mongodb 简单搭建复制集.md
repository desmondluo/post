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

### 开始搭建
#### 准备
```
mkdir mongodbtest
cd mongodbtest
mkdir data   // 数据存放
mkdir data/28001
mkdir data/28002
mkdir data/28003
mkdir conf   // 配置存放
mkdir log    // 日志存放
```

#### 实例1
```
cd conf
vim 28001.conf
port=28001                                                     # 端口
bind_ip=192.168.0.235                                          # 绑定的IP
logpath=/home/deploy/dev/mongotest/log/28001.log               # 日志文件
dbpath=/home/deploy/dev/mongotest/data/28001/                  # 数据存放目录
logappend=true                                                 # 日志追加模式
pidfilepath=/home/deploy/dev/mongotest/data/28001/28001.pid    # pid存放的文件
fork=true                                                      # 后台进程
oplogSize=1024                                                 # oplog的大小(大一点的好,这个地方测试)
replSet=IMOOC                                                  # 集群名称(同步是整个实例的数据, 并不是中间一个imooc的db, 这个是集群名称, 不是db名称)
auth=true                                                      # 是否开始验证(初步使用的时候, 设置为false, 然后再创建用户)
keyFile=/home/deploy/dev/mongotest/conf/.keyfile               # 集群之间验证的key(默认开启auth=true)
```

#### 实例2
```
cd conf
vim 28002.conf
port=28002                                                     # 端口
bind_ip=192.168.0.235                                          # 绑定的IP
logpath=/home/deploy/dev/mongotest/log/28002.log               # 日志文件
dbpath=/home/deploy/dev/mongotest/data/28002/                  # 数据存放目录
logappend=true                                                 # 日志追加模式
pidfilepath=/home/deploy/dev/mongotest/data/28002/28002.pid    # pid存放的文件
fork=true                                                      # 后台进程
oplogSize=1024                                                 # oplog的大小(大一点的好,这个地方测试)
replSet=IMOOC                                                  # 集群名称(同步是整个实例的数据, 并不是中间一个imooc的db, 这个是集群名称, 不是db名称)
auth=true                                                      # 是否开始验证(初步使用的时候, 设置为false, 然后再创建用户)
keyFile=/home/deploy/dev/mongotest/conf/.keyfile               # 集群之间验证的key(默认开启auth=true)
```

#### 实例3
```
cd conf
vim 28003.conf
port=28003                                                     # 端口
bind_ip=192.168.0.235                                          # 绑定的IP
logpath=/home/deploy/dev/mongotest/log/28003.log               # 日志文件
dbpath=/home/deploy/dev/mongotest/data/28003/                  # 数据存放目录
logappend=true                                                 # 日志追加模式
pidfilepath=/home/deploy/dev/mongotest/data/28003/28003.pid    # pid存放的文件
fork=true                                                      # 后台进程
oplogSize=1024                                                 # oplog的大小(大一点的好,这个地方测试)
replSet=IMOOC                                                  # 集群名称(同步是整个实例的数据, 并不是中间一个imooc的db, 这个是集群名称, 不是db名称)
auth=true                                                      # 是否开始验证(初步使用的时候, 设置为false, 然后再创建用户)
keyFile=/home/deploy/dev/mongotest/conf/.keyfile               # 集群之间验证的key(默认开启auth=true)
```

#### 启动集群
```
// 测试的时候, 如果没有权限, 注释掉auth和keyFile
mongod -f conf/28001.conf
mongod -f conf/28002.conf
mongod -f conf/28003.conf
```

#### 初始化
```
mongo 192.168.0.235:28001
config = {
	"_id" : "IMOOC",
	"members" : [
		{
			"_id" : 0,
			"host" : "192.168.0.235:28001"
		},
		{
			"_id" : 1,
			"host" : "192.168.0.235:28002"
		},
		{
			"_id" : 2,
			"host" : "192.168.0.235:28003"
		}
	]
}
rs.initiate(config)   // 初始化这个集群
// 测试将一个改为投票结点
config.members[2].arbiterOnly = true
rs.remove("192.168.0.235:28003")
rs.reconfig(config)
```

这样就搭建完成一个简单的复制集了
