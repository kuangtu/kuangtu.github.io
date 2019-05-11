---
layout:     post
title:      "protobuf-c安装使用及多个消息处理方式"
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
初始化消息msg，通过amessage__get_packed_size得到消息序列化后的大小，分配buffer之后，通过 amessage__pack将Amessage序列化到buf中。
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



### 消息反序列化

序列化过程中将消息写入到文件中。然后基于unpack方法将消息反序列化。

```c
#include <stdio.h>
#include <stdlib.h>
#include "amessage.pb-c.h"
#define MAX_MSG_SIZE 1024

static size_t
read_buffer (unsigned max_length, uint8_t *out)
{
  size_t cur_len = 0;
  size_t nread;
  while ((nread=fread(out + cur_len, 1, max_length - cur_len, stdin)) != 0)
  {
    cur_len += nread;
    if (cur_len == max_length)
    {
      fprintf(stderr, "max message length exceeded\n");
      exit(1);
    }
  }
  return cur_len;
}


int main (int argc, const char * argv[]) 
{
  AMessage *msg;

  // Read packed message from standard-input.
  uint8_t buf[MAX_MSG_SIZE];
  size_t msg_len = read_buffer (MAX_MSG_SIZE, buf);

  // Unpack the message using protobuf-c.
  msg = amessage__unpack(NULL, msg_len, buf);	
  if (msg == NULL)
  {
    fprintf(stderr, "error unpacking incoming message\n");
    exit(1);
  }

  // display the message's fields.
  printf("Received: a=%d",msg->a);  // required field
  if (msg->has_b)                   // handle optional field
    printf("  b=%d",msg->b);
  printf("\n");

  // Free the unpacked message
  amessage__free_unpacked(msg, NULL);
  return 0;
}
```

通过标准输入读取序列化之后的数据放入到buf中，然后通过unpack方法反序列化得到消息内容。

需要注意的时候，read_buffer过程中从标准输出读取的数据有最大的限制，但是没有“数据包”的分隔。PS：TCP数据流方式也类似，从socket中读取的数据流没有分隔，需要通过buffer缓存并进行“切割”。



### 多个消息

在实际项目应用过程中，会传输多个不同类型的消息，比如：股票行情数据，可能有成交、订单、指数等不同类型的消息。通过protobuf规范和上面的例子可以看到，protobuf规范中没有定义消息的长度和消息类型。在陈硕的文章[为什么 Protobuf 的默认序列化格式没有包含消息的长度与类型？](<http://www.cppblog.com/Solstice/archive/2011/04/13/144085.html>)中介绍了以下可能的场景：

Protobuf 是经过深思熟虑的消息打包方案，它的默认序列化格式没有包含消息的长度与类型，自然有其道理。哪些情况下不需要在 protobuf 序列化得到的字节流中包含消息的长度和（或）类型？我能想到的答案有：

- 如果把消息写入文件，一个文件存一个消息，那么序列化结果中不需要包含长度和类型，因为从文件名和文件长度中可以得知消息的类型与长度。
- 如果把消息写入文件，一个文件存多个消息，那么序列化结果中不需要包含类型，因为文件名就代表了消息的类型。
- 如果把消息存入数据库（或者 NoSQL），以 VARBINARY 字段保存，那么序列化结果中不需要包含长度和类型，因为从字段名和字段长度中可以得知消息的类型与长度。
- 如果把消息以 UDP 方式发生给对方，而且对方一个 UDP port 只接收一种消息类型，那么序列化结果中不需要包含长度和类型，因为从 port 和 UDP packet 长度中可以得知消息的类型与长度。
- 如果把消息以 TCP 短连接方式发给对方，而且对方一个 TCP port 只接收一种消息类型，那么序列化结果中不需要包含长度和类型，因为从 port 和 TCP 字节流长度中可以得知消息的类型与长度。
- 如果把消息以 TCP 长连接方式发给对方，但是对方一个 TCP port 只接收一种消息类型，那么序列化结果中不需要包含类型，因为 port 代表了消息的类型。
- 如果采用 RPC 方式通信，那么只需要告诉对方 method name，对方自然能推断出 Request 和 Response 的消息类型，这些可以由 protoc 生成的 RPC stubs 自动搞定。

而在之前单个消息的例子中，也没有包含消息的长度。而在实际的网络编程中需要解决长度问题：

- protobuf 打包的数据没有自带长度信息或终结符，需要由应用程序自己在发生和接收的时候做正确的切分。
- 需要确定消息的类型，然后创建正确的Protobuf Message 对象，再做的反序列化。

对于消息类型的处理，陈硕在[一种自动反射消息类型的 Google Protobuf 网络传输方案](<https://blog.csdn.net/solstice/article/details/6300108/>)文章中给出了基于反射(reflection)功能的处理方法。笔者通过protobuf-c还是采用了额外增加消息类型的方式进行处理。

### 多个消息序列化和反序列化

假设数据传输中有两个消息，证券基本信息和证券价格。proto文件如下：

```c
syntax = "proto2";

  
message MktInfo {
  required string stockid = 1;
  required string stockname = 2;
  required string exchange = 3;
}

message MktData {
  required string stockid = 1;
  required int64 price = 2;
}

message Header {
  required uint32 u32MsgLen = 1;
  required uint32 u32MsgType = 2;
}


```

基本流程：

- 在传输过程中，先对于消息长度、消息类型进行压缩，然后序列化消息生成binary数据；
- 解析时收到binary消息，先读取长度字段len，然后读取消息类型type，根据长度分配buffer，再读取len长度的数据，最后根据type类型创建不同的protobuf对象。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "multimsg.pb-c.h"

enum PacketType {
  MKTINFO_TYPE = 1,
  MKTDATA_TYPE = 2,
};

int main (int argc, const char * argv[]) 
{
  void *buf;                      // Buffer to store serialized data
  void *headerbuf;
  size_t len, headerlen;
  FILE *fp = NULL;
  
  Header header = HEADER__INIT;
  MktInfo mktinfo = MKT_INFO__INIT;
  MktData mktdata = MKT_DATA__INIT;
  char *stockid = "600001";
  char *stockname = "pudong";
  char *exchange = "sse";

  fp = fopen("multimsg.dat", "w+");
  
  //写入mktinfo消息
  mktinfo.stockid = strdup(stockid);
  mktinfo.stockname = strdup(stockname);
  mktinfo.exchange = strdup(exchange);
  len = mkt_info__get_packed_size(&mktinfo);
  buf = malloc(len);
  mkt_info__pack(&mktinfo, buf);
  
  //先写入消息长度和消息类型
  fwrite(&len, sizeof(size_t), 1, fp);
  int type = MKTINFO_TYPE;
  fwrite(&type, sizeof(int), 1, fp);
  
  fprintf(stderr, "write mktinfo %zu serialized bytes.\n", len);
  fwrite(buf, len, 1, fp);
  free(buf);
  free(mktinfo.stockid);
  free(mktinfo.stockname);
  free(mktinfo.exchange);
  
  //写入mktdata消息
  mktdata.stockid = strdup(stockid);
  mktdata.price = 1024;
  len = mkt_data__get_packed_size(&mktdata);
  buf = malloc(len);
  mkt_data__pack(&mktdata, buf);
  
  //先写入消息长度和消息类型
  fwrite(&len, sizeof(size_t), 1, fp);
  type = MKTDATA_TYPE;
  fwrite(&type, sizeof(int), 1, fp);
  fprintf(stderr, "write mktinfo %zu serialized bytes.\n", len);
  fwrite(buf, len, 1, fp);
  free(buf);
  free(mktdata.stockid);
  
  fclose(fp);
  
  return 0;
}
```

在反序列化时，先读取长度和消息类型字段。

```c
#include <stdio.h>
#include <stdlib.h>
#include <inttypes.h>
#include "multimsg.pb-c.h"
#define MAX_MSG_SIZE 4096

int main (int argc, const char * argv[]) 
{
  char *buf;
  FILE *fp = NULL;
  size_t len;
  int type;
  
  MktInfo *mktinfo;
  MktData *mktdata;
  fp = fopen("multimsg.dat", "rb");
  
  while(fread(&len, sizeof(size_t), 1, fp) != 0)
  {
    printf("the read len is:%zu\n", len);
    fread(&type, sizeof(int), 1, fp);
    if (type == 1)
    {
      printf("the msg type is:%d, the len is:%zu\n", type, len);
      //read buf
      buf = malloc(len);
      fread(buf, 1, len, fp);
      mktinfo = mkt_info__unpack(NULL, len, buf);
      printf("the stock id is:%s--name is:%s--exchange is:%s\n", mktinfo->stockid, mktinfo->stockname, mktinfo->exchange);
    
    }
    else if (type == 2)
    {
      printf("the msg type is:%d, the len is:%zu\n", type, len);
      //read buf
      buf = malloc(len);
      fread(buf, 1, len, fp);
      mktdata = mkt_data__unpack(NULL, len, buf);
      printf("the stock id is:%s--price is:%"PRIi64"\n", mktdata->stockid, mktdata->price);
    }
    else
    {
      printf("the other msg type:%d\n", type);
    }
  }
  fclose(fp);
  
    
  return 0;
}
```

对于protobuf二进制文件，通过fread不断读取。

在实际生产环境中，通过网络socket发送接收（non-block）时，可以考虑的方式是：

- 需要在read是将收到的数据放入到缓存中；
- 判断缓存的数据长度是否可以读取一个长度字段；
- 如果可以，解析长度字段得到序列化后消息的长度len，然后再检查buffer中数据长度是否达到消息类型长度+len；
- 得到了一个消息原始的binary，反序列化即可。

## 参考文献

1. <https://blog.csdn.net/zlf19910726/article/details/79911676>
2. <https://blog.csdn.net/kid_2412/article/details/52275582>
3. <https://blog.csdn.net/programmer_h/article/details/8890800>
4. <https://github.com/protobuf-c/protobuf-c/pull/342/commits/080724364a5aad61728f8eba57677467bf329088>
5. <http://www.cppblog.com/Solstice/archive/2011/04/13/144085.html>
6. <https://blog.csdn.net/solstice/article/details/6300108/>
7. <https://code.google.com/apis/protocolbuffers/docs/techniques.html#streaming>
8. <https://halfrost.com/protobuf_encode/#>

