---
title: vim配置编译生成gvim
date: 2021-08-21 23:00:52
tags: vim gvim gtk3
---

# 源代码编译生成gvim

## 环境

笔者使用RedHat7.6版本进行开发工作，gcc升级之后为8.5.0，glibc为2.25。

系统默认的vim版本是：，如果需要安装默认的gvim，使用命令：

```bash
yum install vim-X11
```

输入```gvim```，可以看到版本是：

![默认gvim版本](/img/gvim-compile/默认gvim版本.jpg)

**但是发现启动gvim之后不能输入中文。输入法切换至拼音输入法之后，无法在insert模式下输入。**



## 源代码编译vim8.0

通过 [FTP网站](https://ftp.nluug.nl/pub/vim/unix/) 下载vim8.0版本上传至主机，解压。configure参数如下：

```bash
./configure --prefix=/home/softinstall/local --enable-luainterp --enable-cscope --with-x --enable-gui=gtk3  --with-features=huge --enable-xim  --enable-multibyte >config.txt
```

打开config.txt，发现X11header文件没有，没有GUI支持。

```bash
 checking if X11 header files can be found... no
 checking --enable-gui argument... no GUI support
```



## gui编译方法

阅读了INSTALL，其中有说明通常会安装devel包。configure配置中gui方式基于gtk3。安装以下包：

```bash
yum install libX11-devel
yum install gtk3-devel
yum install libXt-devel
```

gtk3-devel同时会安装部分依赖包：

![gtk3安装](/img/gvim-compile/gtk3安装.jpg)

configure之后，可以看到enable-gui已经支持了。

```bash
checking if X11 header files can be found... yes
checking for _XdmcpAuthDoIt in -lXdmcp... no
checking for IceOpenConnection in -lICE... yes
checking for XpmCreatePixmapFromData in -lXpm... no
checking if X11 header files implicitly declare return values... no
checking size of wchar_t is 2 bytes... no
checking --enable-gui argument... GTK+ 3.x GUI support
```

然后编译，此时可以生成gvim了。