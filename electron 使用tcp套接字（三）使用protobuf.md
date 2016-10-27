---
title: electron 使用protobuf
categories: [js, electron, nodejs]
tags: [js, nodejs, electron, protobuf]
---

### protobuf
protocol Buffers简称protobuf, 是一种数据储存格式, 将数据的序列化, 然后反序列化，非常高效而且与语言平台无关, 通常被用于数据存储或是RPC数据格式交换。 pb和json一样, 属于业务层协议.

### protobufjs
[protobufjs](https://github.com/dcodeIO/protobuf.js) 是probobuf的js非官方版本. 提供了基本的序列化与反序列化的接口.

### 基本使用
这里演示一下js使用probobuf

#### 首先来一点proto
```
syntax = "proto2";
package HISRPC;

// 消息类型枚举
enum enum_method_type
{
  server_ping_req = 1;                                // ping
	server_pong_rep = 2;                                // pong
}

// 根消息结构体
message rpc_root_msg
{
	required enum_method_type method = 1;					// 请求类型
	optional bytes            body   = 2;					// 请求正文
	optional uint32           seq    = 3;					// 请求流水(客户端可以用来保证请求和回应是否对应)
}

// 心跳包
message server_heart_req_msg
{
    required bool ping = 1;                                      // 心跳（true）
}

// 心跳返回包
message server_heart_rep_msg
{
    required bool pong = 1;                                      // 心跳（true）
}

```

#### 调用代码
```
/**
 * Auther: Desmond
 * Email: DesmondJie@gmail.com
 * Date: 2016/9/18
 */
var queue = require('./queue.js');
class history extends queue{
    // 基础构建
    constructor() {
        super();
        this.path             = require('path');
        this.protobuf         = require('protobufjs');
        this.bytebuffer       = require('bytebuffer');
        this.long             = require('long');
        this.builder = this.protobuf.loadProtoFile(this.path.join(__dirname, '..', 'proto', 'common.proto'));
        this.hisrpc  = this.builder.build('HISRPC');
        this.seq     = 1;
    }

    get_seq() {
        return ++this.seq;
    }

    get_ping_rpc(data, cbSuccess, cbFailed) {
        let method = this.hisrpc.enum_method_type.server_ping_req;
        let pingRpc = new this.hisrpc.server_heart_req_msg();
        let seq = this.get_seq();
        pingRpc.ping = data.ping;
        let rootRpc = this.get_root_prc(method, pingRpc, seq, cbSuccess, cbFailed);
        return rootRpc;
    }

    get_root_prc(method, body, seq, cbSuccess, cbFail) {
        let rootMsg = new this.hisrpc.rpc_root_msg();
        rootMsg.method = method;
        rootMsg.body = body.encode();
        if (seq != undefined) {
            rootMsg.seq = seq;
            if (seq != undefined) {
                this.add(seq, cbSuccess, cbFail);
            }
        }
        return rootMsg.encode();
    }

    // 回调回去, 回调回去之后要删除对应的回调, 防止这个回调越来越大
    ondata(data) {
        //let rootMsg = this.hisrpc.decode(data);
        let rootMsg = new this.hisrpc.rpc_root_msg.decode(data);
        let repMsg  = undefined;
        let seq     = rootMsg.seq;
        if (rootMsg.method == this.hisrpc.enum_method_type.server_pong_rep) {
            repMsg = this.hisrpc.server_heart_rep_msg.decode(rootMsg.body);
        }
        console.log(repMsg);
        if (seq && this.successQueue[seq] && repMsg) {
            this.successQueue[seq](repMsg);
        }

        if (seq && this.failQueue[seq] && !(repMsg)) {
            this.failQueue[seq](repMsg);
        }

        if (seq && this.successQueue[seq]) {
            delete this.successQueue[seq];
        }

        if (seq && this.failQueue[seq]) {
            delete this.failQueue[seq];
        }
    }
}

module.exports = {
    history: history
};
```
