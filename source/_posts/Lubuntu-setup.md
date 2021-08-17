---
title: Lubuntu配置C++开发环境
categories:
  - 环境配置
tags:
  - linux系统
abbrlink: 3876e13f
date: 2021-08-16 14:25:17
---

最近想要研究嵌入式Linux的开发，所以想要配置一套基于Linux桌面的开发环境。Linux发行版选用Ubuntu 20.04LTS版本，但是我又不喜欢Ubuntu原生的非洲红，所以选用了Lubuntu，LXQt桌面环境，能够更省内存，界面看着也还比较舒适。本文会介绍如何安装环境，并配置相应的C/C++开发环境。

<!--more-->

## 系统基础组件配置
1. 更换国内的软件源，并更新源信息
   
    `sudo apt update`


2. 安装基础管理工具, net-tools, curl, ssh

    突然发现ifconfig指令不存在，所以根据提示安装`net-tools`这个包
   
    `sudo apt install net-tools curl`
    
    安装ssh服务和客户端
   
    `sudo apt install openssh-server openssh-client`

    启用ssh服务并开机自启

    `sudo systemctl enable ssh`

    `sudo systemctl start ssh`

    

## 安装配置C语言开发环境
1. 安装vscode
    安装比较好用的编辑器VSCode，Ubuntu上可以直接通过snap商店安装，或者官网下载deb安装，也可以添加apt源进行安装。

    这里选择添加apt源的方式进行安装，方便后续升级操作。

    `sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"`

    发现apt会报security警告，所以需要安装gpg密钥

    `curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg`

    `sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg`

    再更新一下安装vscode就可以运行, `sudo apt update && apt install code`.安装完成在终端输入code就可以打开vscode编辑器了。

2. 安装C/C++编译器
    
    安装C++相关的工具包

    `sudo apt install build-essential make cmake automake autotools-dev autoconf m4`

    安装完成后我们编译一个demo.c程序进行测试

    ```c
    #include <stdio.h>

    int main()
    {
        printf("Hello World!\n");
        return 0;
    }
    ```

    在命令行输入`gcc demo.c -o demo`编译得到可执行文件，然后再终端运行可以得到输出
    ```
    $./demo
    Hello World
    ```

至此基本的环境就配置好了，等后续有时间再更新具体的嵌入式开发环境

