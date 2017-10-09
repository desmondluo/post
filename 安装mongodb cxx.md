---
title: 安装mongodb cxx
categories: [mongodb]
tags: [mongodb, c++]
---

### 安装环境centos7
#### 安装libmongoc
```
sudo yum install pkg-config openssl-devel cyrus-sasl-devel
wget https://github.com/mongodb/mongo-c-driver/releases/download/1.8.0/mongo-c-driver-1.8.0.tar.gz
tar xzf mongo-c-driver-1.8.0.tar.gz
cd mongo-c-driver-1.8.0
./configure
sudo make -j4
sudo make install
```

#### 安装mongocxx
```
wget https://github.com/mongodb/mongo-cxx-driver/archive/r3.0.1.tar.gz
tar xzf r3.0.1.tar.gz
cd mongo-cxx-driver-r3.0.1/build
#需要先安装好git，yum -y install git，不然会报错
PKG_CONFIG_PATH=/usr/local/lib/pkgconfig cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local/ ..
sudo make EP_mnmlstc_core
sudo make -j4
sudo make install
```
