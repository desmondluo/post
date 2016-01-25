---
title: NodeJs Node-Webkit C++ addon 环境搭建
categories: nodejs
tags: [nodejs, node-webkit, nwjs]
---
#### 使用Visual Status 2013创建node modules模块
    学习视频：https://www.youtube.com/watch?v=KvjHn59C-uQ
#### 1：下载并编译源码
    下载地址：https://nodejs.org/download/
    编译命令：vcbuild.bat nosign Release  //编译release版本
     vcbuild.bat nosign Debug                // 编译Debug版本
#### 2：设置环境变量
    将当前node环境设置到环境变量里面
    set NODE_HOME=C:\Libs\node
    set NODE_ROOT=C:\Libs\node

#### 3:创建vs工程
    保证创建一个空的工程
    修改工程属性：
    1：Configuration Type: Dynamic Libary(.dll)            //node的实质是dll
    2：Target Extension .node                         //将扩展名改成node
    3：$(NODE_ROOT)\src\               //添加工程include文件
    4：添加v8与uv的include目录   // $(NODE_ROOT)\deps\v8\include   $(NODE_ROOT)\deps\uv\include
    5：$(NODE_ROOT)\$(Configuration)\            //添加node.lib的lib文件
    6：link 下的input添加node.lib

#### 上面说的是搭建了使用nodejs创建C++模块，可是node-webkit创建的C++模块稍有些麻烦


#### 创建node-webkit C++
首先node-webkit写的C++模块是等同于nodejs写的C++模块的，但是由于编译的方式不一样，最后会导致直接用nodejs编译的node文件在node-webkit里面无法使用
 1.  下载安装与node-webkit相同版本的nodejs(iojs)，建议使用installer安装，方便使用npm模块，并且避免一段的环境配置的麻烦
 2.  安装nw-gyp    npm install -g nw-gyp
 3.  进入工作目录，配置对应的版本nw-gyp configure --target=0.12.0   // 工作目录就是需要编译的模块目录 target 版本号就是对应的node-webkit版本号
 4.  运行nw-gyp build进行编译，这个使用要有对应的building.gyp文件，或是说直接打开上一步运行生成的工程，这样就生成了.node文件
 5.  将生成的.node文件，放到node-webkit的一个目录，最后在js引用的使用，使用js的目录为当前目录，然后进行引用，就能运行了（var binding = require("../../../binding.node");）
