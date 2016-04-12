---
title: ssh 管理
categories: [工具, 效率]
tags: [ssh, 工具, 效率, ssh-copy-id]
---
### 什么是SSH
Secure Shell(缩写为SSH), SSH为一项创建在应用层和传输层基础上的安全协议, 主要使用在计算机远程登录。我们平常使用的场景主要有登录远程vps, git代码提交等。
<!--more-->
### 为什么要管理SSH
这里我的场景是VPS越来越多了, 平常的时候都是手敲连接命令, 多了之后, 比较难记住。当切换了另一台电脑之后, 杂乱的ssh环境对工作效率尤为影响。

### SSH登录
SSH的常用登录命令
```
ssh username@remote_host -p 9898
```

### SSH的别名功能
SSH允许我们通过使用别名, 让我们脱离记住复杂命令的困扰。在.ssh目录下有一个config文件, 不存在则创建。它的写法如下
```
Host ALY254
	HostName      122.168.1.254
	Port          9898
	User          desmond
	IdentityFile  ~/.ssh/id_rsa
```

配置之后我们就可以通过别名访问
```
ssh ALY254
```
这样就能登录到122.168.1.254的服务器了。

### 自动登录的authorized_keys文件
.ssh目录下有个authorized_keys文件，作用是保存用户的公钥, 前面配置的IdentityFile是私钥, 这里是公钥。 我们只要把用户的公钥追加在这个文件后面, 那么拥有私钥的登录用户就是合法用户。

只有配置好了authorized_keys文件, 才能实现SSH的别名登录。

### ssh-copy-id
这个工具就是为了解决copy公约到远程服务器的authorized_keys的操作，事实上这个个脚本, 大概是这样的
```
ssh username@remote_host -p 9898 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
```
