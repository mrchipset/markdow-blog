---
title: Lubuntu上安装Vitis 2020.1集成开发环境
tags:
  - FPGA
  - Xilinx
  - IDE
  - Setup
categories:
  - FPGA
abbrlink: a7d89769
date: 2021-08-18 11:42:14
---


前段时间在学习Xilinx FPGA的开发，一开始装的Vivado，配上Linux虚拟机安装Petalinux。有时候感觉不是非常方便，所以这次准备直接在Lubuntu上安装Vitis+Petalinux。本文会记录一些Lubuntu上的坑，具体安装请参阅Xilinx的User Guides。
<!--more-->

## 居然打不开Vitis的安装包
由于我们使用的发行版是Lubuntu，安装包会认为我们不是受支持的操作系统，然后安装程序就会不停报Splash相关的异常。

等了很久，也没再进入图形安装界面。后来通过搜索资料发现，此时可以使用命令行生成配置文件进行安装，具体操作方法如下：

1.生成配置文件

`./xsetup -b ConfigGen`

通常会生成到~/.Xilinx/install_config.txt目录下。可以直接编辑生成的文件，修改安装配置。

2.选择刚才生成的配置文件，并进行安装

`sudo ./xsetup -c ~/.Xilinx/install_config.txt --agree XilinxEULA,3rdPartyEULA,WebTalkTerms --batch Add`

3.等待安装完成即可

## 打开Vivado居然报错
安装完成后，载入环境变量，欣喜地打开vivado，发现居然会报错。。。。

`application-specific initialization failed: couldn't load file "librdi_commontasks.so": libtinfo.so.5: cannot open shared object file: No such file or directory`

查看错误信息应该是因为没有安装`libtinfo`，因此清除万能地`apt`大法

`sudo apt install libtinfo-dev`

由于我的版本是20.04，安装的会是`libtinfo.so.6`，所以需要额外创建一个软连接

`sudo ln -s /lib/x86_64-linux-gnu/libtinfo.so.6 /lib/x86_64-linux-gnu/libtinfo.so.5`

## 安装Petalinux的一个小坑
一开始使用了`sudo`进行Petalinux的安装,后来用普通用户权限一直报错，log没有看仔细，改了好几次目录权限。实际上我们要做的应该是删除Petalinux生成的安装Log。