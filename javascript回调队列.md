---
title: javascript 回调队列
categories: [javascript]
tags: [javascript, 回调队列]
---

### 回调队列
异步回调是一种高性能做法, 可以有效的解决IO等待, 防止程序阻塞。回调队列指的是一个队列里面有很多回调, 当条件满足的时候, 里面的回调将会一个一个触发. 类似于C++里面的信号槽的概念, 或者说事件注册机制.
<!--more-->
画个图描述一下

![](http://ww1.sinaimg.cn/large/005OdUDHgw1f6h1qmibisj30l808yaaw.jpg)

当条件满足的时候里面的每个回调都会执行

### javascript 原生支持回调
javascript [闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)的概念使得回调非常容易实现, 我们只需要创建一个闭包, 放入回调队列, 就可以了。 js的垃圾回收机制和原型链使得上下文环境保存非常好。

### 使用回调队列的场景
不同的项目, 会产生不同的场景, 这里举例我遇到的场景.

![](http://ww1.sinaimg.cn/large/005OdUDHgw1f6h2hkozfkj30nq0fztah.jpg)

做一个单页应用, 有三个模块同时用到数据X, 如果数据X发生了变化, 那么我们同时要通知三个模块, 这个时候回调队列作用就体现了, 我们每个模块都向数据中心注册一个回调, 当数据变化的时候, 依次执行回调, 进而实现数据更新。

### 遇到一些问题
实际编码过程中遇到一些问题, 当某个模块(假设模块A)重新load的时候, 我们会再次向数据中心发起注册请求, 这个时候原来已经保留了一份过时的回调, 因为js的原型链等原因, 我们去执行这份回调的时候, 并不会发生错误, 如果我们继续保留这份回调，势必会产生队列膨胀、效率低下等一些问题.

![](http://ww1.sinaimg.cn/large/005OdUDHgw1f6h2z14z3rj30o10fb0uo.jpg)

这就意味这我们必须删除原来这个回调, 然后重新push一个新的回调, 那么判断两个回调相等成为解决这个问题的关键.

#### 判断两个闭包相等
例举一段代码
```
$scope.refreshItempros = function() {
        var data = {};
        $scope.itempros = [];
        net.post('x', data, function(msg, params){
            if (params.id != undefined)
                return;
            // hostController itempros_list
            $scope.itempros = msg.itempros;
        }, function(msg){
            // hostController itempros_list
            $rootScope.$broadcast("info", msg);
        });
    };
$scope.refreshItempros();
```
当模块A执行的时候, 我们向队列X发起了一个注册请求, 然后发送了一个闭包函数
```
function(msg, params){
    if (params.id != undefined)
        return;
    // hostController itempros_list
    $scope.itempros = msg.itempros;
}
```
也就是说, 判断这个闭包在同一个地方在不同的时候, 被post出去, 要认为这两个闭包相等。

#### 一个不好的方法toString()
用toString的方法, 这个闭包会打印出这个函数的代码, 如果两个代码相等, 那么我们可以简单的认为这两个闭包是相等的.
```
callback.toString() === success.toString()
```
这就引出一个问题, 如果两个不同的模块, 用的闭包函数代码一样, 那么势必会冲突, 当项目大的时候，这种问题肯定会发生。

当然我们可以加模块索引, 即给闭包添加一个所属模块的属性, 最后用这个属性去判断。感觉这样解决并不好, 那为什不直接在代码里面加模块名称呢。

### 构建回调队列
例举一段代码, 抛砖引玉
```
net.post = function(api, params, success, error) {
    // 构建回调队列
    if (net.callbacks[api] == undefined)
        net.callbacks[api] = {};
    if (net.callbacks[api].success == undefined)
        net.callbacks[api].success = [];
    for (var i = 0; i < net.callbacks[api].success.length; ++i) {
        if (net.callbacks[api].success[i].callback.toString() === success.toString()) {
            net.callbacks[api].success.splice(i, 1);
            break;
        }
    }
    var back = {
        callback: success,
        msg : params
    }
    net.callbacks[api].success.push(back);
};

```

### 总结
回调队列只是一种思想, 一种异步处理数据，多点更新数据的方法, 具体怎么实现, 是否需要用的都视项目而定, 我们应该了解这种思想, 而不拘泥它！
