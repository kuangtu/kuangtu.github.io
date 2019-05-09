---
layout:     post
title:      "protobuf-c安装和使用"
date:       2018-01-01
author:     "莫大"
header-img: ""
tags:
    - 读书
    - 技术
---

> 记录技术成长的点点滴滴。


## 前言
Google Protocol Buffer（简称protobuf）是Google内部混合语言数据标准，protobuf是一种紧凑的可扩展的二进制消息格式，适合做网络数据传输、数据存储的消息格式。与 XML 和 JSON 相比体积更小，同时它与平台和语言无关的，便于数据交换。

Protobuf 协议提供了多种语言支持。本文对于protobuf C语言使用进行说明。



## ProtoBuf-C库安装

protobuf-c是 [Google Protocol Buffers](https://developers.google.com/protocol-buffers/)的C语言实现。包含了libprotobuf-c纯C库，以及protoc-c代码生成器，将.proto文件转换为代码。

### 安装

protobuf-c依赖于： C 、C++ 编译器, [protobuf](https://github.com/google/protobuf) 和pkg-config。即：需要先安装protobuf。

从github上面下载protobuf源代码，通过如下命令进行安装。

```
./autogen.sh && ./configure && make && make install
```

需要注意的是：

（1）在configure生成makefile文件的时候，我定义了--prefix=/usr/local/protobuf。

（2）autogen.sh脚本执行依赖autoconf, automake, libtool。如果没有安装的话先进行安装。我使用的环境为CentOS 7.5。通过yum源进行安装。

```
yum install autoconf

yum install automake

yum install libtool
```

安装完成protobuf之后，下载protobuf-c源代码编译安装。

```
./autogen.sh && ./configure && make && make install
```

需要注意的是：

（1）在configure生成makefile文件的时候，我定义了--prefix=/usr/local/protobuf-c，与之前的protobuf路径不同。

（2）protobuf-c源代码cxx-generate-packed-data.cc:1001行有个bug，需要修改如下所示：



## ProtoBuf-C使用

安装完成之后，基于[示例](https://github.com/protobuf-c/protobuf-c/wiki/Examples) 中的例子对于单个消息、重复field、子消息等不同的情况进行分析。



## 参考文献

1. <https://blog.csdn.net/zlf19910726/article/details/79911676>
2. <https://blog.csdn.net/kid_2412/article/details/52275582>
3. <https://blog.csdn.net/programmer_h/article/details/8890800>
4. <https://github.com/protobuf-c/protobuf-c/pull/342/commits/080724364a5aad61728f8eba57677467bf329088>



