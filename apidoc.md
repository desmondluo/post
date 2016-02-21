---
title: 使用apidoc生成RESTful风格API文档
tags: [apidoc, 文档, 测试]
categories: [nodejs, 工具]
---
### 为什么要用apidoc写接口文档
[apidoc](http://apidocjs.com)是一个接口文档生成器, 通过代码固定格式的注释, 从而生成固定格式的文档, 同时提供接口测试, 这样我们获得的不仅仅是一个接口文档, 同时也是一个接口测试文档.

### 什么是apidoc
[apidoc](http://apidocjs.com)是一个nodejs的工具, 解析注释, 然后生成html.

### 安装apidoc
[apidoc](http://apidocjs.com)是一个nodejs工具，使用之前必须先安装node(nodejs真是个好东西)。
```
npm install apidoc -g
```
<!--more-->

### 使用apidoc
首先我们要有一个注释
```
/**
* @api {get} /admin/admin/snap 获取行情
* @apiName /admin/admin/snap
* @apiDescription 获取某一只股票的一年的行情
* @apiGroup admin
* @apiParam {String} stock 股票代码
* @apiParam {String} api="/admin/admin/snap" 接口api
* @apiUse SapiSign
* @apiUse apiError
* @apiUse apiSuccess
* @apiSampleRequest  http://apidoc.example.cn/admin/admin/snap
*/
```
简单的运行脚本
```
apidoc -i myapp/ -o out/ -t mytemplate -f ".php"
// -i 指定输入文件夹
// -o 指定输出文件夹
// -t 指定模板(用来生成静态接口文档页面)
// -f 需要解析的后缀名
// 这个命令表示解析myapp下的所有.php文件，然后生成文档到out目录下，使用的生成模板是mytemplate(不指定，会有默认的模板)
```
最后生成[文件](http://apidocjs.com/example/)

![](http://ww4.sinaimg.cn/mw690/005OdUDHgw1f0cta3kdywj30z50o4gpu.jpg)
### 接口测试
接口都可以同步生成测试

![](http://ww4.sinaimg.cn/mw690/005OdUDHgw1f0ctd4l94wj30sn0gcq54.jpg)

### 更好的使用apidoc
1. 注释模板化-使用apiDefine和apiUse
    restful风格api我们都定义了很多相同的地方，比如输入输出，计算sign所需的参数等. 如果我们每个接口都写这些参数，势必导致代码臃肿。

    我们定义一个Comment.php,把我们要的共有参数放在这
    ```
    /**
     * @apiDefine SapiSign
     * @apiParam {String} pid 产品ID
     * @apiParam {String} key  客户端验证key
     * @apiParam {String} sign 加密之后的sign
     * @apiParam {String} uid  用户的id
     * @apiParam {String} t    请求发起的时间
     * @apiParam {String} token 登录获取的key
     */

    /**
     * @apiDefine apiError
     * @apiError {json} Error-Response:
     *      HTTP/1.1 200 OK
     *      {
     *          "code" : -1
     *      }
     */
    ```
    然后在需要的文件里面使用@apiUse
    ```
    /**
     * @api {get} /admin/admin/snap 获取行情
     * @apiName /admin/admin/snap
     * @apiDescription 获取某一只股票的一年的行情
     * @apiGroup admin
     * @apiParam {String} stock 股票代码
     * @apiParam {String} api="/admin/admin/snap" 接口api
     * @apiUse SapiSign
     * @apiUse apiError
     * @apiUse apiSuccess
     * @apiSampleRequest  http://apidoc.example.cn/admin/admin/snap
     */
    ```
    这样我们就能使用较少的注释，生成更为详细的文档
2. 使用自己的template-让测试更简单
    计算sign, 我们通常使用了比较复杂的计算流程，而apidoc只是简单的提供输入输出，并不帮你复杂的计算，这样会导致我们需要自己算sign,然后写入测试. 流程麻烦，而且一旦修改数据，都要重算。

    但是apidoc提供了template, 我们可以通过修改template里面的生成文件，最后达到计算的目的。

    比如，我修改了template/utils/send_sample_request.js
    ```
    if (param['sign'] != undefined) {
          // 这个时候说明要处理sign
          param['t'] = Date.now() / 1000 | 0;
          var str = "";
          if (group == "sapi") {
              str = /*截取参数*/ + param['api'];
          } else if (group == "admin") {
              str = /*截取参数*/ + param['api'];
          } else if (group == "webapi") {
              // 没有计算sign,但是要做跳转,直接打开一个url
          }
          param['sign'] = md5(str);
      }
    ```
    这个地方只是一个简单的实例，我们甚至可以默认参数等等。
### 我的一些不成熟的想法
1. 前段时间，我一直纠结于接口测试问题，我想实现接口的批量测试，我可以写nodejs或是py脚本, 可是这些并不能实现可视化测试, 于是我想使用node-webkit实现测试. node-webkit本身就可以使用node库,webkit写页面,然后底层用curl做调用,理论上应该是完美解决的,可是node-webkit坑太多了,我写测试的被恶心到了(因为node的js执行环境和浏览器js的执行环境要分开,否则调用会出问题. 当时我又在玩typescript和angular2, 里面又是一堆的坑),最后作罢.听说atom-electron解决了js调用问题,我想如果用atom-electron做内核,这个想法应该是完美的.
2. apidoc还有一个确定,就是有可能会有跨域问题,你懂的. 这种情况下,我们一般会把文档部署在当前工程下面,或是允许跨域,但是这样并不是一个好的方法,比如不能实现文档的集中管理等.所以我想如果用php做一层转接，即把所有的接口发送到php服务里面，然后php再发到我们要的测试接口，因为php用的是curl，跨域问题就不存在了，同时我们还可以把加密逻辑写到php里面，岂不是更好。
