---
title: codecept api测试初步使用
tags: [codecept, 测试, api]
categories: [codecept, api]
---

### 关于codecept
codecept是PHP的测试框架, 我们可以用它进行单元测试, 功能测试, api测试等. 一般的小公司很少会涉及到测试, 可是随着项目的增加, 我们可能很随意的就改了以前的代码, 如果没有相关的测试, 很容易发生问题.
<!--more-->
### api测试
api测试不同于一般的单元测试, 我们测试的是对外的接口, 而不是单纯的一段代码.

### 安装codecept
```
composer global require "codeception/codeception=2.1.*"
composer global require "codeception/specify=*"
composer global require "codeception/verify=*"
composer global require "flow/jsonpath=*"

# 加入bin目录
export PATH="/home/user/.config/composer/vendor/bin:$PATH"
```

### 初始化
```
codecept bootstrap
```

初始化完成之后, 生成tests目录与codeception.yml文件

### build
tests目录下创建api.suite.yml文件
```
class_name: ApiTester
modules:
    enabled:
        - REST:
            url: http://themeinvestad.jingzhuan.dev
            depends: PhpBrowser
            part: Json
        - Yii2
```

运行
```
codecept build
```

创建测试文件
```
#通过命令创建
codecept generate:cept api getProcess

```

直接在test/api目录下创建文件getProcessCept.php

```
// 自动创建的代码
<?php use backend\tests\ApiTester;
$I = new ApiTester($scenario);
$I->wantTo('perform actions and see result');
// 开始我们的业务
I->sendPOST("/site/login", ["userform[username]"=>"desmond", "userform[password]" => "desmond"]);
$I->seeResponseIsJson();
$I->seeResponseContainsJson(["code"=>"0"]);
$I->seeResponseJsonMatchesJsonPath("$.response.token");
```

### 运行测试
```
codecept run api
```

![](https://encrt.com/wp-content/uploads/2017/01/QQ图片20170118145338.png)
