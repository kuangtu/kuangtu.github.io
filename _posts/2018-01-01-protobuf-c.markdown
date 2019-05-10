---
layout:     post
title:      "protobuf-c安装和使用"
date:       2018-01-01
author:     "莫大"
header-img: "img/protobuf.jpg"
tags:
    - 读书
    - 技术   
    - 消息传输
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

![编译错误](/img/protobuf-c-bug.jpg)



## ProtoBuf-C使用

安装完成之后，基于[示例](https://github.com/protobuf-c/protobuf-c/wiki/Examples) 中的例子对于单个消息、重复field、子消息等不同的情况进行分析。

### 单个消息

基于ProtoBuf语法，定义单个消息为：

```c
syntax = "proto2";
message AMessage {
  required int32 a=1; 
  optional int32 b=2;
}
```

该消息中先通过"syntax"定义了protobuf的协议版本。消息中包含了两个字段，字段a是必须的，b是可选的。

然后通过`protoc-c`生成.c和.h文件。

```c
 protoc-c --c_out=. amessage.proto 
```

其中.h文件主要内容如下：

```c
typedef struct _AMessage AMessage;
/* --- enums --- */
/* --- messages --- */
struct  _AMessage
{
  ProtobufCMessage base;
  int32_t a;
  protobuf_c_boolean has_b;
  int32_t b;
};
/* AMessage methods */
void   amessage__init
                     (AMessage         *message);
size_t amessage__get_packed_size
                     (const AMessage   *message);
size_t amessage__pack
                     (const AMessage   *message,
                      uint8_t             *out);
size_t amessage__pack_to_buffer
                     (const AMessage   *message,
                      ProtobufCBuffer     *buffer);
AMessage *
       amessage__unpack
                     (ProtobufCAllocator  *allocator,
                      size_t               len,
                      const uint8_t       *data);
void   amessage__free_unpacked
                     (AMessage *message,
                      ProtobufCAllocator *allocator);
```

其中，

（1）amessage__get_packed_size得到消息压缩时的大小；

（2）amessage__pack压缩消息；

（3）amessage__unpack解压消息。

.c文件主要对于上面的方法进行定义。后续压缩、解压过程中，需要使用这些方法，因此编译成动态链接库。

```
gcc -I /usr/local/protobuf-c/include/ -c dmessage.pb-c.c
```

### 消息序列化

```c
#include <stdio.h>
#include <stdlib.h>
#include "amessage.pb-c.h"

int main (int argc, const char * argv[]) 
{
  AMessage msg = AMESSAGE__INIT; // AMessage
  void *buf;                     // Buffer to store serialized data
  unsigned len;                  // Length of serialized data
  
  if (argc != 2 && argc != 3)
  {   // Allow one or two integers
    fprintf(stderr,"usage: amessage a [b]\n");
    return 1;
  }
  
  msg.a = atoi(argv[1]);
  if (argc == 3) { msg.has_b = 1; msg.b = atoi(argv[2]); }
  len = amessage__get_packed_size(&msg);
  
  buf = malloc(len);
  amessage__pack(&msg,buf);
  
  fprintf(stderr,"Writing %d serialized bytes\n",len); // See the length of message
  fwrite(buf,len,1,stdout); // Write to stdout to allow direct command line piping
  
  free(buf); // Free the allocated serialized buffer
  return 0;
}
```

初始化消息msg，通过amessage__get_packed_size得到消息序列化后的大小，分配buffer之后，通过 

amessage__pack将Amessage序列化到buf中。

### 消息反序列化

序列化过程中将

## 参考文献
1. <https://blog.csdn.net/zlf19910726/article/details/79911676>
2. <https://blog.csdn.net/kid_2412/article/details/52275582>
3. <https://blog.csdn.net/programmer_h/article/details/8890800>
4. <https://github.com/protobuf-c/protobuf-c/pull/342/commits/080724364a5aad61728f8eba57677467bf329088>



