---
title: Ubuntu安装配置Thrift
tags:
  - thrift
  - rpc
  - linux
categories:
  - 环境配置
abbrlink: c22935cd
date: 2021-08-26 17:26:22
---


Thrift最早是由Facebook开发的一个RPC项目，目前已经捐献给了Apache基金会。本文将会记录如何在Ubuntu Linux上安装Thrift，并进行简单的测试。
<!--more-->

## 按照Apahce Thrift官网的指示进行安装

1. 安装依赖项
```
sudo apt-get install automake bison flex g++ git libboost-all-dev libevent-dev libssl-dev libtool make pkg-config
```

2. 下载Thrift的源代码
```
wget https://downloads.apache.org/thrift/0.14.2/thrift-0.14.2.tar.gz
```
`tar -zxf`解压之后,进入到源码目录进行config设置
```
$ ./configure
```
运行之后会输出相应的配置信息

```
thrift 0.14.2

Building ActionScript3 Library : no
Building C (GLib) Library .... : no
Building C++ Library ......... : yes
Building Common Lisp Library.. : no
Building D Library ........... : no
Building Dart Library ........ : no
Building .NET Standard Library : no
Building Erlang Library ...... : no
Building Go Library .......... : no
Building Haskell Library ..... : no
Building Haxe Library ........ : no
Building Java Library ........ : no
Building Lua Library ......... : no
Building NodeJS Library ...... : no
Building Perl Library ........ : no
Building PHP Library ......... : no
Building Python Library ...... : yes
Building Py3 Library ......... : yes
Building Ruby Library ........ : no
Building Rust Library ........ : no
Building Swift Library ....... : no

C++ Library:
   C++ compiler .............. : g++ -std=c++11
   Build TZlibTransport ...... : yes
   Build TNonblockingServer .. : yes
   Build TQTcpServer (Qt5) ... : no
   C++ compiler version ...... : g++ (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0

Python Library:
   Using Python .............. : /usr/bin/python3
   Using Python version ...... : Python 3.8.10
   Using Python3 ............. : /usr/bin/python3
   Using Python3 version ..... : Python 3.8.10
```
突然发现Java Library是木有的，但是我们的系统装了Java。经过一番搜索之后发现，如果想支持java的话，还需`Apache Ant`。
之后再进行configure的时候就能看到java相关的信息了。

由于java库还需要gradle，会自动下载，可能需要等待一段时间。

输出配置信息之后，就可以三连
```
sudo make
sudo make check
sudo make install
```

## 安装完成后的hello world测试
一顿操作之后，感觉安装完成了。想写个java的服务端和客户端程序做验证，结果失败。。。
后来发现是`libtrift.jar`依赖于annotation和slf4j：

以下是我选用的几个jar包
1. javax.annotation.jar
2. slf4j-api-1.7.31.jar
3. slf4j-simple-1.7.31.jar
添加这几个jar包之后就能够完成编译thrift官方的教程demo了

如果是使用C++的话，就需要连接libthrift.so这个动态链接库，相比java的demo前期简单一些。
