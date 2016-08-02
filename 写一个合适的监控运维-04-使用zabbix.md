---
title: 写一个合适的监控运维系统-04-使用zabbix
categories: [监控, 运维, 监控系统]
tags: [工具, 监控, 运维, zabbix, zabbixapi]
---

### 什么是zabbix
[zabbix](https://www.zabbix.org/wiki/Main_Page) 是一款异常强大的分布式系统监控工具, 几乎可以监控到系统的所有东西, 并提供了报警绘图等等一系类我们需要的工具, 几乎是应有尽有. 同时它提供了一套完整的[api](https://www.zabbix.com/documentation/2.4/manual/api/reference), 第三方工具可以通过api获取zabbix的数据. 这里我们就是通过api获取zabbix的数据, 给到我们自己的监控系统, 作为数据参考. 这样我们无需自己再写一个agent, 去获取系统数据了!
<!--more-->

### 使用zabbix [api](https://www.zabbix.com/documentation/2.4/manual/api/reference)
建议浏览一下整个文档, 然后选取自己需要的api.


zabbix架构图
![](http://ww1.sinaimg.cn/large/005OdUDHgw1f6fv4cbh87j30k00i6abv.jpg)

大概就是这样了, zabbix里面有很多机器, 我们对他们分组之后，就有一个groupid, 里面有n多机器, 每个机器都会有一个hostid, 每台机器会有很多监控项目, 每个监控项目, 都会有一个item_id, 需要注意的是不同主机之间，相同的监控项目, item_id也是不一样的. 比如A机器有监控内存item_id为1, B机器也有监控内存, 那么它的id可能是2, 不再可能是1了. 弄清楚这些之后, 后面我们回获取数据都是围绕这几个id来的.

#### api格式
###### 发送格式
```
{
    "jsonrpc": "2.0",
    "method": "api.name",
    "params": {
        "user": "Admin",
        "password": "zabbix"
    },
    "auth": "f7caaa2aaee43a3387a3fd39959569a1",
    "id": 1
}
```
- jsonrpc: 是协议，这个永远都不会变
- method: 接口名称，每个接口都不一样
- params: 传过去的参数
- auth: 登录返回的token(除登录之外, 都要传这个参数)
- id: 你发送的id, 原样返回, 客户端自己维护

###### 返回格式
```
{
    "jsonrpc": "2.0",
    "result": "0424bd59b807674191e7d77572075f33",
    "id": 1
}
```

- jsonrpc: 协议
- result: 返回结果(结构变化, 可能是array, 可鞥是kv, 可能是string等等)
- id: 你发送的id

#### 常用的几个接口
###### 登录 user.login
```
{
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
        "user": "Admin",
        "password": "zabbix"
    },
    "id": 1
}
```
```
# Install the Python Requests library:
# `pip install requests`
import requests
def send_request():
    # user.login（用户登录）
    # POST http://127.0.0.1:8088/zabbix/api_jsonrpc.php

    try:
        response = requests.post(
            url="http://127.0.0.1:8088/zabbix/api_jsonrpc.php",
            headers={
                "Content-Type": "text/plain; charset=utf-8",
            },
            data="{
  \"jsonrpc\": \"2.0\",
  \"method\": \"user.login\",
  \"params\": {
    \"user\": \"Admin\",
    \"password\": \"zabbix\"
  },
  \"id\": 1
}"
        )
        print('Response HTTP Status Code: {status_code}'.format(
            status_code=response.status_code))
        print('Response HTTP Response Body: {content}'.format(
            content=response.content))
    except requests.exceptions.RequestException:
        print('HTTP Request failed')

```

##### 获取所有分组 hostgroup.get
```
{
  "jsonrpc": "2.0",
  "method": "hostgroup.get",
  "params": {
    "output": "extend",
    "filter": {}
  },
  "auth": "f7caaa2aaee43a3387a3fd39959569a1",
  "id": "1"
}
```
```
import requests
def send_request():
    # hostgroup.get（主机组获取）
    # POST http://121.201.7.90:8088/zabbix/api_jsonrpc.php

    try:
        response = requests.post(
            url="http://121.201.7.90:8088/zabbix/api_jsonrpc.php",
            headers={
                "Content-Type": "text/plain; charset=utf-8",
            },
            data="{
  \"jsonrpc\": \"2.0\",
  \"method\": \"hostgroup.get\",
  \"params\": {
    \"output\": \"extend\",
    \"filter\": {}
  },
  \"auth\": \"f7caaa2aaee43a3387a3fd39959569a1\",
  \"id\": \"1\"
}"
        )
        print('Response HTTP Status Code: {status_code}'.format(
            status_code=response.status_code))
        print('Response HTTP Response Body: {content}'.format(
            content=response.content))
    except requests.exceptions.RequestException:
        print('HTTP Request failed')
```

##### 获取组内主机 host.get
```
{
  "jsonrpc": "2.0",
  "method": "host.get",
  "params": {
    "output": [
      "hostid",
      "name",
      "host"
    ],
    "groupids": [
      "9",
      "8"
    ],
    "selectGroups": [
      "groupid",
      "name"
    ],
    "selectInterfaces": [
      "ip"
    ],
    "sortfiled": "hostid"
  },
  "auth": "f7caaa2aaee43a3387a3fd39959569a1",
  "id": "1"
}
```

##### 获取主机监控项目 item.get
```
{
  "jsonrpc": "2.0",
  "method": "item.get",
  "params": {
    "hostids": [
      "10148"
    ],
    "output": "extend"
  },
  "auth": "f7caaa2aaee43a3387a3fd39959569a1",
  "id": "1"
}
```

### 总结
zabbix api非常的灵活, 对第三方平台很是友好, 通过zabbix api 我们能监控到应用外的几乎所有的数据, 这也是我们为什么使用zabbix的原因. 
