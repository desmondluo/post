---
title: 使用apidoc生成RESTful风格API文档
tags: [apidoc, 文档, 测试]
categories: [nodejs, 工具]
---
### 为什么要写接口文档
PHP是一门更接近业务的脚本,而restful风格的api,意味着与你对接api的客户端,可能不止一个. 良好而完整的接口文档,能帮助我们节省不少对接时间. 平常写惯了C++,养成了不喜欢写注释也随便写注释的习惯,写PHP后愈觉得这种陋习不可取.偶尔有一天，一个同事过来，看了我的代码，说了一句注释都没有，让别人怎么看，更是坚定了我写接口文档的决心。

### 什么是apidoc
[apidoc](http://apidocjs.com)是一个接口文档生成器，通过代码固定格式的注释，从而生成固定格式的文档，同时提供接口测试，这样我们获得的不仅仅是一个接口文档，同时也是一个接口测试文档。

### 安装apidoc
apidoc是一个nodejs工具，使用之前必须先安装node(nodejs真是个好东西)。
```
npm install apidoc -g
```

### 使用apidoc
```
// 简单的运行脚本
apidoc -i myapp/ -o out/ -t mytemplate -f ".php"
// -i 指定输入文件夹
// -o 指定输出文件夹
// -t 指定模板(用来生成静态接口文档页面)
// -f 需要解析的后缀名
```
