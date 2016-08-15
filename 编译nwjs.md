---
title: 编译nwjs
categories: nwjs
tags: [nwjs, node-webkit]
---
#### 编译node-webkit
我用的是vs2013 update5 的版本, 生成的nw 0.12
 1. 按照[这里](https://github.com/nwjs/nw.js/wiki/Building-nw.js) 下载好depot_tools 和 node-web源码
 2. 把depot_tools加入环境变量
 ```
 path=F:\depot_tools;%path%
 ```
 3. copy directxsdk
 ```
mkdir -p /path/to/node-webkit/src/third_party/directxsdk/files
cp -r /c/Program\ Files\ \(x86\)/Microsoft\ DirectX\ SDK\ \(June\ 2010\)/* /path/to/node-webkit/src/third_party/directxsdk/files/
 ```
4. 生成工程文件(ninja 编译文件)
```
python.bat build/gyp_chromium.py content/content.gyp
```
5. 设置环境变量并编译
```
set DEPOT_TOOLS_WIN_TOOLCHAIN=0
set GYP_MSVS_VERSION=2013
set GYP_GENERATORS=ninja
ninja -C out/Release nw -j4
```

##### 直接copy文件编译
经常有同事需要从我这复制整个环境, 编译nwjs, 但是nwjs对环境的依赖比较深, 这里说明怎么复制环境
1. 安装vs2013 update5
2. copy depot_tools 和 node-webkit源码
3. 删除旧的目录(/pathto/node-webkit/src/out)
4. 重新生成工程文件
5. 设置环境变量, 并且编译
第三步非常重要, 因为生成工程文件的时候, 会将depot_tools里面的一些文件目录硬写死在工程文件里面, 导致最后没法编译

有需要编译nwjs帮助的小伙伴欢迎留言->_->
