---
title: electron 使用bytebuffer
categories: [js, electron, nodejs]
tags: [js, nodejs, electron, bytebuffer]
---

### buffer
每个socket拥有自己的read buffer 和 send buffer, 但是这里我们讨论的不是socket的buffer, 而是我们业务层的buffer. 在tcp套接字中, 当我们一次发送过量的数据, 或是其他情况的时候, socket底层可能会分为多个包发送, 这个时候我们就需要用buffer粘包.

### 粘包
![](http://ww1.sinaimg.cn/large/005OdUDHgw1f96t6nar6wj30fz02n0su.jpg)

一个Header + Data组成一个Pack, Header我们定义为一个4个自己的int, Data为数据. 当我们判断收到一个Pack的时候, 我们开始处理业务.

### bytebuffer
[bytebuffer](https://github.com/dcodeIO/bytebuffer.js), 是js的buffer库, 这里我们用bytebuffer代替node提供的buffer.

#### bytebuffer 的数据模型
data： 数据array

offset: 开始读的偏移量

limit： 数据最尾段的位置

#### bytebuffer 数据定义
bytebuffer帮我们定义了我们常用的数据类型, 避免了无类型数据的长度问题.

int, long, short, int8, int16, int32, int64, uint8, uint16, uint32, uint64, float, float32, float64, string, utf8string等.

同时bytebuffer帮我们处理了编码问题, 大小端的问题.

#### bytebuffer api
为了更好的读数据, bytebuffer提供了丰富的[api](https://github.com/dcodeIO/bytebuffer.js/wiki/API)。

readInt(), readFloat()等(接口非常丰富, 请参考api文档)。

#### bytebuffer 一些问题
append(source, encoding=, offset=)等合并操作: 追加数据并不正确, 当多个buffer需要合并的时候, 我们经常会使用append, 我们最好使用for循环暴力copy, 然后手动调整offset和limit.

### 一个完整的粘包程序
```
/**
 * Auther: Desmond
 * Email: Desmondjie@gmail.com
 * Date: 2016/9/17
 */

class connection{

    constructor(){
        this.path       = require("path");
        this.net        = require('net');
        this.bytebuffer = require('bytebuffer');

        var self = this;
        this.socket = new this.net.Socket();
        this.socket.on('error',      function(data){self.error(data);});
        this.socket.on('disconnect', function(data){self.disconnected(data);});
        this.socket.on('connect',    function(data){self.connected(data);});
        this.socket.on('data',       function(data){self.ondata(data);});

        // 队列的长度(2M)
        this.buffer    = new this.bytebuffer(2 * 1024 * 1024, true);
        this.buffer.flip();
        this.need      = 4;
        this.ishead    = true;
        this.datacb    = undefined;
    }

    log(log) {
        console.log(log);
    }

    connect(ip, port, callback) {
        this.socket.connect({port:port, host:ip}, callback);
    }


    connected(data) {
        this.log(data);
    }


    disconnected(data) {
        this.log(data);
    }

    error(data) {
        this.log(data);
    }

    set_data_cb(cb) {
        this.datacb = cb;
    }

    ondata(data) {
        // 先把原始的buffer转成bytebuffer
        let buf = new this.bytebuffer(data.length, true);
        buf.append(data.buffer);
        buf.flip();
        //this.buffer.flip();
        // 粘包(这个地方用for循环, 因为append等函数总是不对)
        for(let i = buf.offset; i < buf.limit; ++i)
        {
            this.buffer.buffer[ this.buffer.limit + i] = buf.buffer[i];
        }
        this.buffer.limit += (buf.limit - buf.offset);
        //this.buffer.flip();
        // 处理队列
        while (this.buffer.remaining() >= this.need) {
            // 头
            if (this.ishead) {
                // 获得新的长度
                this.need = this.buffer.readInt();
                // 置换
                for (let i = 0; i < this.buffer.limit - this.buffer.offset; ++i)
                {
                    this.buffer.buffer[i] = this.buffer.buffer[this.buffer.offset + i];
                }
                this.buffer.limit = this.buffer.limit - this.buffer.offset;
                this.buffer.offset = 0;
                this.ishead = false;
            } else {
                // 获得buf
                let subbuf = this.buffer.copy(this.buffer.offset, this.need);
                this.buffer.offset += this.need;
                // 反射回去
                if (this.datacb != undefined) {
                    this.datacb(subbuf);
                }
                // 置换
                for (let i = 0; i < this.buffer.limit - this.buffer.offset; ++i)
                {
                    this.buffer.buffer[i] = this.buffer.buffer[this.buffer.offset + i];
                }
                this.buffer.limit = this.buffer.limit - this.buffer.offset;
                this.buffer.offset = 0;
                this.ishead = true;
                this.need = 4;
            }
        }
    }

    send(msg) {
        let msgArrayBuffer = msg.toArrayBuffer();
        let length         = msgArrayBuffer.byteLength;
        let sendBuffer = new this.bytebuffer(4 + length, true);
        sendBuffer.writeUint32(length).append(msg).flip();
        return this.socket.write(sendBuffer.toBuffer(true));
    }
}

module.exports = {
    connection: connection
};
```
