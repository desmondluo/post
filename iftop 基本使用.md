---
title: iftop的基本使用
tags: [iftop, linux, 调优, 监视端口, 网络统计]
categories: [liunx, 工具]
---

### iftop
  iftop是一个网络监测工具, 可以统计到端口的流量情况. 平常我们使用nload统计服务器流量也是够了, 但是当服务器部署了多个应用的时候, 需要知道是每个应用的流量的时候, 这个工具就会显得比较厉害了.

### 安装iftop
```
sudo yum install flex byacc  libpcap ncurses ncurses-devel libpcap-devel
// 需要从源码进行编译
wget http://www.ex-parrot.com/pdw/iftop/download/iftop-0.17.tar.gz
tar zxvf iftop-0.17.tar.gz
cd iftop-0.17
./configure
sudo make && sudo make install
```  

### 使用
  切换到root用户, 然后iftop
```
su root
iftop
```
然后我们就会看到下面的图
![](http://ww4.sinaimg.cn/large/005OdUDHgw1f2swm27yvzj30ij0c4jvw.jpg)

### 参数
  直接在命令行的时候, 指定参数并不管用(^_^), 需要在页面上指定参数
```
iftop -N // it doesn't work
```
  运行期指定参数 打开之后 按p

  ![](http://ww4.sinaimg.cn/large/005OdUDHgw1f2swstv54gj30ie0bkq8a.jpg)

  这样就会显示端口了(也有可能是服务名称如SSH), 如果再按 shift + n, 就会显示全是端口

### 使用filter
  iftop 会显示所有的网络信息, 而某些时候，我们只要看需要的信息，比如我只想看9700端口.

  按f

  ![](http://ww3.sinaimg.cn/large/005OdUDHgw1f2sx1deeztj30ib01awek.jpg)

  然后输入port 9700

  ![](http://ww4.sinaimg.cn/large/005OdUDHgw1f2sx365sxej30ig05n0ut.jpg)

  怎么写filter，参考下面的连接

  [简单点的](https://tournasdimitrios1.wordpress.com/2011/01/20/iftop-monitor-and-analyze-your-network-traffic-on-linux/)
  [有点复杂](https://tournasdimitrios1.wordpress.com/2011/01/20/iftop-monitor-and-analyze-your-network-traffic-on-linux/)

### 注意
使用filter命令之后， 这个命令只是慢慢的生效，就是会观察到连接慢慢的在减少，然后最后剩下你想要的那个.
