---
title: electron 使用tcp
categories: [js, electron, nodejs]
tags: [js, nodejs, electron, bytebuffer]
---

### electron 使用nodejs获得tcp能力
浏览器在js层是不能使用原生的tcp套接字的, 但是electron使用了node作为中间层, 所以我们可以依托node从而获得tcp能力.


### nodejs 的net模块
nodejs中的socket, 即可以作为server, 也可以作为client, 有时候我们需要在页面上动态的展示下面多个client的连接状态, 数据发送状态等, 这个时候我们使用server。 更经常使用的是作为client, 我们的electron app连接某个tcp服务器, 然后使用tcp作为底层数据传输协议, electron展示消息, 并作为交互.

#### server
创建一个tcp服务器

```
var server = net.createServer();
server.listen(PORT, HOST);
console.log('Server listening on ' + server.address().address + ':' + server.address().port);

server.on('connection', function(sock) {
    console.log('CONNECTED: ' +
         sock.remoteAddress +':'+ sock.remotePort);
    // 处理业务
    // 为这个socket实例添加一个"data"事件处理函数
    sock.on('data', function(data) {
        console.log('DATA ' + sock.remoteAddress + ': ' + data);
        // 回发该数据，客户端将收到来自服务端的数据
        sock.write('You said "' + data + '"');
    });

    // 为这个socket实例添加一个"close"事件处理函数
    sock.on('close', function(data) {
        console.log('CLOSED: ' +
            sock.remoteAddress + ' ' + sock.remotePort);
    });
});
```

#### client
作为客户端
```
var net = require('net');
var HOST = '127.0.0.1';
var PORT = 6000;

var client = new net.Socket();
client.connect(PORT, HOST, function() {

    console.log('CONNECTED TO: ' + HOST + ':' + PORT);
    // 建立连接后立即向服务器发送数据，服务器将收到这些数据
    client.write('Hello!');

});

// 为客户端添加“data”事件处理函数
// data是服务器发回的数据
client.on('data', function(data) {

    console.log('DATA: ' + data);
    // 完全关闭连接
    client.destroy();
});

// 为客户端添加“close”事件处理函数
client.on('close', function() {
    console.log('Connection closed');
});
```

### 更多
创建一个server或是client是非常简单的, 但是实际使用中我们会遇到各式各样的问题, 其中最重要的问题是封包和buffer的问题， 下一篇我们说一下使用bytebuffer解决buffer问题.
