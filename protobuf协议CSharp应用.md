---
title: protobuf协议CSharp应用
categories: [协议, google, C#]
tags: [protobuf, 协议, google, C#, .net, RPC]
---

最近在整改服务端的私有协议, 使用了protobuf作为新的RPC通信交换格式。以前手机客户端走的是http协议, 连接PHP, 然后PHP又通过[ICE](https://zeroc.com/products/ice)与C++服务器做连接, 现在直接用tcp然后用protobuf传输。最近接手了整个手机行情服务器的开发工作，我需要一个稳定的界面测试数据，C++写界面太麻烦，想着直接用C#的winform拖拖控件就好了，于是就有了用C#接protobuf的这篇博文。
<!--more-->
### 什么是protobuf
protocol Buffers简称protobuf, 是一种数据储存格式, 将数据的序列化, 然后反序列化，非常高效而且与语言平台无关, 通常被用于数据存储或是RPC数据格式交换。

### C#的protobuf库
我用的是pb2, 没有官方的库，可是社区有。[库地址](https://github.com/jskeet/protobuf-csharp-port)，这个库后面被官方采纳，成为了pb3的官方C#[库](https://github.com/google/protobuf/tree/master/csharp)。社区的东西好是好，就是没有太多的文档，总是让我们踩了不少坑, 希望需要用的pb2的小伙伴读了这篇博文可以少踩点坑。

### 编译生成cs类文件的protogen.exe
首先下载好整个源码，源码提供了生成的脚本，在build目录下。
![](http://ww3.sinaimg.cn/large/005OdUDHgw1f2al7jrv28j30ti0cz770.jpg)

通过运行命令
```
build.bat NET40 Build Release
```
最后在out_build目录下会生成exe文件，以及两个需要依赖的dll.

### 生成.cs文件
用protogen.exe 然后指定目录就可以生成cs文件

我的编译脚本
```
start ./build_output/tools/protogen.exe --proto_path=./rpc_protos/src/ -output_directory=./RPC ./rpc_protos/src/common.proto ./rpc_protos/src/auth.proto

```
需要特别注意的地方是：当你引入了另一个pb文件的时候，一定要一起编译，否则，否则，否则就会编译不过去(即使你设定了include目录)

### 使用protobuf
首先要引入生成的那两个DLL
```
using Google.ProtocolBuffers;
using Google.ProtocolBuffers.Serialization;
using Google.ProtocolBuffers.Serialization.Http;
using Google;
```

#### pb文件
```
message client_login_msg
{
    required string name     = 1;
    required string password = 2;
}
```

#### 首先要构建build
```
client_login_msg.Builder msgbuild = client_login_msg.CreateBuilder();
msgbuild.SetName(Encoding.UTF8.GetString(Encoding.UTF8.GetBytes(username)));
msgbuild.SetPassword(Encoding.UTF8.GetString(Encoding.UTF8.GetBytes(password)));
```

#### 从build里面获得实体类
```
client_login_msg loginMsg = msgbuild.Build();
```
这样才能构建一个pb类

#### 序列化成二进制
```
using (MemoryStream stream = new MemoryStream())
{
    loginMsg.WriteTo(stream);
    bytes = stream.ToArray();
}
```
通过转成二进制之后, 后面的东西就都与pb无关, 通常我们就把它直接从网络发送出去了。
```
netStream.Write(bytes, 0, bytes.Length);
netStream.Flush();
```

#### 从二进制转pb
```
client_login_msg msg = client_login_msg.ParseFrom(bytes);
```

### 结语
事实上没有什么难的，就是对初次使用的人来说，生成protogen.exe,以及生成cs文件容易踩坑。 后面使用的时候，不能直接使用pb类， 而要通过一层build的转换，不知道的时候，容易抓狂。

Enjoy it~
