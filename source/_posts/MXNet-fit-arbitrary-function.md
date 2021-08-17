---
title: 使用MXNet构建神经网络对任意函数进行拟合
tags:
  - MXNet
  - 神经网络
categories:
  - 人工智能
mathjax: true
abbrlink: f5fc5b7d
date: 2020-07-24 13:38:49
---

本文将介绍使用MXNet实现任意函数的拟合功能。本文将从最基础的 $f(x)=a*x+b$ 开始一步一步阐述神经网络的原理。在实践环节当中还将尝试拟合一个复杂的函数以达到对神经网络拟合函数系数有一个更全面、更工程化的理解。
<!--more-->

## 人工神经网络和Apache MXNet


人工神经网络(Artificial Neural Network, ANN)<sup>[1]</sup>，简称神经网络(Neural Network, NN)，是一种模仿生物神经网络的结构和功能的数学模型。将传统的数据输入馈入构建好的神经网络模型，模仿神经元之间通过放电来交换信息的功能，以实现对函数进行评估或者近似。通过大量的人工神经元联结多层可以根据已知的输入、输出集合对神经元内部的参数进行改变。因此训练神经网络的过程就是配合优化算法，能够在已知的数据集上在有限次的迭代过程中，使得神经网络模型能够更好地适配数据集。

Apache MXNet<sup>[2]</sup>是一个开源深度学习框架，用于训练和部署深度神经网络。MXNet具有很好的拓展性和灵活的编程模型，能够快速构建并训练模型。

## 利用单层神经网络拟合一元一次方程
在本节当中，将通过拟合一元一次方程对MXNet训练神经网络地过程做必要的介绍。

## Reference
- [1] [人工神经网络](https://en.wikipedia.org/wiki/Artificial_neural_network)
- [2] [Apache MXNet](https://zh.wikipedia.org/zh-hans/Apache_MXNet)
