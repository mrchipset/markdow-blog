---
title: Lubuntu配置docker环境
tags:
  - linux系统
  - Docker
  - Container
categories:
  - 环境配置
abbrlink: 4fa8b96
date: 2021-08-19 10:58:02
---

由于开发内容比较繁杂，可能会用到各种各样奇怪的开发环境和生产环境部署模拟。以前一直用虚拟机去做，后来发现虽然可以快照虚拟机，但是安装还是比较复杂。因此一直想尝试一下Docker容器化各种环境和牛逼的K8S。这次就下定决心开干，首先先部署好Docker的环境，为后续工作做准备。
<!--more-->

## Docker
Docker可以看作是一个轻量级的虚拟机环境，同时Docker社区提供了大量的配置好的应用镜像可以供开发者使用。因此相对于虚拟机环境，需要自己配置一个开发环境而言方便一些。

## 安装
废话不多说，下面我们开始在Lubuntu上安装docker环境。

1. 安装apt仓库依赖
    ```bash
    $ sudo apt-get update
    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```

2. 获取GPG密钥

    `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg`

3. 添加Docker的稳定仓库地址
    ```shell
    $ echo \
    "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

4. 安装Docker引擎
    ```
    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```

5. 安装完成后跑个hello world验证、

    执行以下命令，docker会自动下载hello world镜像并运行
    `sudo docker run hello-world`

