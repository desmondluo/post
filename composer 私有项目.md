---
title: composer 私有项目
tags: [php, composer]
categories: [php, composer, 工具]
---

### 接触composer 
composer 是php的包管理工具, 有了composer我们可以轻易的从网上获取需要的组件, 构建我们的项目. 最近在写一个软件项目, 需要从C++行情服务器获取一些数据, 准备写一个模块专门用来对接C++接口. 考虑到公司大量的php项目需要获取行情数据, 如果把此模块做成composer包, 将会有很大的帮助.
<!--more-->
### composer 原理

#### 类的自动加载
...(不做阐述)

#### 安装原理
##### 项目地址
项目地址可以在任意可访问地址, 最好使用https. 我们在此地址提交我们的项目, 然后创建好分支(发布), 也可以直接使用master作为开发分支.

##### packagelist 
储存项目名称以及地址等元数据作为索引, 这样我们可以通过索引下载需要的包.

##### 私有项目
私有项目是不提交到packagelist的, 那么我们需要指定项目路径.

```
    "repositories":[
        {
            "type":"vcs",
            "url":"http://gitlab.private.cn/cplus/quote.git"
        }
    ],
```

### 搭建一个私有项目
#### 创建一个私有项目
使用composer init创建一个项目, 然后生成composer.json文件
```
{
    "name": "soft/quote",
    "description": "行情数据读取组件",
    "authors": [
        {
            "name": "desmond",
            "email": "demo@gmail.com"
        }
    ],
    "autoload":{
        "psr-4":{
            "zoft\\quote\\":"src/"
        }
    }
}
```

#### 另一个地方引用此项目
```
{
    "name": "yiisoft/yii2-app-advanced",
    "minimum-stability": "dev",
    "config" : {
        "secure-http": false
    },
    "repositories":[
        {
            "type":"vcs",
            "url":"http://gitlab.private.cn/cplus/quote.git"
        }
    ],
    "require": {
        "soft/quote": "dev-master"
    }
}
```

#### 需要注意的地方
引用的时候require使用的name, 要与创建项目composer.json的name一致.

如果使用的是非http, 需要在config的块添加"secure-http": false