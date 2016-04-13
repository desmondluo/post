---
title: 性能调优
tags: [linux, 调优, pprof, perf, top]
categories: [liunx, 工具]
---
这篇博文记录今天压测服务器的状况, 里面用到了一些工具, 来显示程序的状况, 写下博文, 总结今天的成果.

### 前言
首先介绍我要测试的程序 - reportserver, 这是最近开发的手机行情服务器, 走的是TCP连接, 然后应用层用了pb协议. 我们的用户主要是手机用户, 正常情况下, 用户会以5秒每次的频率请求数据.  我的测试程序是一个用C#写的windows程序, 建立起连接之后, 我会不断的发送请求, 而不是每隔5秒发发送一次.

### 测试DEMO是怎么写的
```
public static ArrayList threadlist = new ArrayList();    // 这里是线程池
public static ArrayList netlist = new ArrayList();       // 这里是每个线程执行的环境
// 创建threadcount个线程， 发起数据请求
// threadcount 线程的个数
// package 指每次发包的个数
for (int i = 0; i < threadcount; ++i)
{
   StressNetThread temp = new StressNetThread();
   temp.count = packagecount;
   temp.index = i;
   netlist.Add(temp);
}
for (int i = 0; i < netlist.Count; ++i)
{
   Thread thread = new Thread(((StressNetThread)netlist[i]).Run);
   thread.IsBackground = true;
   threadlist.Add(thread);
}
for (int i = 0; i < netlist.Count; ++i)
{
   ((Thread)threadlist[i]).Start();
}

// 文件里面配置, 开10个线程, 每个每次发包20个(当20个全部返回之后, 立马再发送20个, 一直循环下去)
[Setting]
threadcount=10
packagecount=20
type=report
time=1000
```
### 服务器状态
当测试程序跑起来之后, 通过top查看程序状态
```
top
```
![](http://ww1.sinaimg.cn/large/005OdUDHgw1f2u453dd7nj30im06g787.jpg)

很快CPU就上来了

### top查看进程及其线程的状况
```
top -H -p 10848
```

![](http://ww1.sinaimg.cn/large/005OdUDHgw1f2u48974asj30he0etwpg.jpg)

    观察到前面有几个线程CPU耗的厉害
    分别是: 10859 10861 10865 10863 将近在50%左右
    然后是: 10864 10858 10862 10860 将近在15%左右

### 查看这几个线程分别是什么线程
我们通过程序创建的时候记录查看

![](http://ww2.sinaimg.cn/large/005OdUDHgw1f2u4f478u0j30ai079di4.jpg)
```
这个时候我们观察到
50% 的线程是quote_read服务
15% 的线程是QRector服务
quote_read 是copy行情数据的线程
QRector 是网络层
```
也就是说quote_read线程占了绝大部分的CPU

### perf 查看函数CPU调用
```
perf top -t 10861
```

![](http://ww3.sinaimg.cn/large/005OdUDHgw1f2u4ltzrplj30yd04zq5q.jpg)

这个时候我们观察到大部分的CPU占用被一个set_report_value和tcmalloc给占用了

### 总结
这个时候已经很明了, 最后的赋值和内存创建是最耗cpu的操作, 如果我们发起的请求足够大, 要应如此的请求, 这么耗CPU也是正常的. 所以现在分析来看, 程序并没有太大的性能瓶颈.

### 看一下网络
```
nload
```
![](http://ww3.sinaimg.cn/large/005OdUDHgw1f2u4rk71mcj30i309g7by.jpg)

网络流量非常高, 而网络线程在15%的使用率, 说明网络层是没有问题的.
