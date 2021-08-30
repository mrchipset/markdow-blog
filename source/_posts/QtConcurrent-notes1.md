---
title: QtConcurrent使用当中的一次Bug记录
tags:
  - Qt
  - Concurrent
  - 多线程
categories:
  - Qt笔记
abbrlink: 8f93a29c
date: 2021-08-30 16:40:05
---

QtConcurrent是Qt提供的一个简单易用的多线程开发库，笔者平时非常喜欢使用这个库进行开发。但是有一次遇到了一个非常诡异的Bug，在此记录下来方便日后排查。
<!--more-->

## QtConcurrent的简单使用
> *引用字Qt官方文档* <br> 
> The QtConcurrent namespace provides high-level APIs that make it possible to write multi-threaded programs without using low-level threading primitives such as mutexes, read-write locks, wait conditions, or semaphores. Programs written with QtConcurrent automatically adjust the number of threads used according to the number of processor cores available. This means that applications written today will continue to scale when deployed on multi-core systems in the future

Qt官方文档的介绍说QtConcurrent是一个高等级的多线程编程库，可以避免底层的互斥、读写锁等等云云。
基础概念不多说，我们直接上一个简单的demo给大家演示这个库用起来放不方便。


想要开个线程就这么简单
```cc
#include <QCoreApplication>
#include <QtConcurrent>
#include <QThread>

#include <QDebug>

static int foo()
{
    QThread::msleep(500);
    qDebug() << "Print info in another thread:" << QThread::currentThreadId();
    return 0;
}

int main(int argc, char *argv[])
{
    QFuture<int> future = QtConcurrent::run(foo);
    // 尝试注释18-19行的代码观察效果
    future.waitForFinished();
    qDebug() << "Concurrent result:" << future.result();
    qDebug() << "Print info in main thread:" << QThread::currentThreadId();
    return 0;
}

```

上面的代码，我们用`run`启动传入工作函数，启动新的任务线程；然后用`future`获取函数执行后返回的结果；而`waitForFinished()`方法则是等待任务执行完毕。


一切的代码都是这么的丝滑，直到有一天我使用了`mapper`和`synchronizer`之后，我的主线程卡住了。。。
## 一个神奇的Bug

废话不多说，大家来看一个剥离了实际计算方案的耗时任务模拟

```cc
#include <QCoreApplication>
#include <QtConcurrent>
#include <QThread>
#include <QDebug>
#include <QFutureSynchronizer>
#include <QVector>

static int foo(int id)
{
    QVector<double> data(4096);
    QtConcurrent::map(data, [=](double& val){
        for(int loop=0;loop<1024;++loop){
            val = loop;
        }
    }).waitForFinished();
    qDebug() << "QThread id:" << QThread::currentThreadId() << "Id:" << id;
    return id;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QtConcurrent::run([=](){
        QFutureSynchronizer<int> synchronizer;
        for (int id = 0; id < 128; ++id)
        {
            synchronizer.addFuture(QtConcurrent::run(foo, id));
        }
        synchronizer.waitForFinished();
    }).waitForFinished();
    QThread::msleep(100);
    qDebug() << QThreadPool::globalInstance()->maxThreadCount();
    qDebug() << "Main Thread id:" << QThread::currentThreadId() << "Finished";
    return a.exec();
}

```

此时，我们就发现软件在某些核数不多的系统上就卡死了。后来发现是因为`waitForFinish`会发生类似递归嵌套的情况,将我们ThreadPool默认最大的线程数量给占满了。
## 分析

1. 在使用当中还是要避免随意对QtConcurrent的使用,尤其是在核数较少的平台上使用时更要注意。
2. 当然还有一个方法是不要随意的调用`waitForFinish`这个函数，而是使用信号槽来实现。
3. 要理性地使用map函数，这个函数会将工作任务对列表进行并行化，很有可能将线程池资源耗尽。