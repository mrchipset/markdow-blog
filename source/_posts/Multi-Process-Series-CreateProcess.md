---
title: 多进程编程系列（1）- 创建子进程并对其进行简单操作
tags:
  - 多进程
  - 架构
categories:
  - 多进程编程
abbrlink: f02ac829
date: 2020-12-31 11:03:44
---

本文是多进程变成系列的第一篇博文，主要介绍在Window系统如何创建子进程并进行相应简单的操作。本系列将对多进程架构的相关知识如父子进程，进程间通信，锁等知识做简单介绍。
<!--more-->

## 什么是进程？
根据*《深入理解Linux内核3rd》*一书中对进程有如下描述，
> 进程是任何多道程序设计的操作系统中的基本概念。通常把进程定义为程序执行的一个实例，
因此如果16个用户同事运行vi, 那么就有16个独立的进程（尽管它们共享同一个可执行代码）。

其余更深入的内容，读者可以参照*《深入理解Linux内核3rd》*或者操作系统相关书籍，在此不再赘述。

## 最简单的代码创建一个进程
通常创建一个进程最简单的方式是随意打开一个程序。哦，不对，你只想要一个，可能得到很多，比如打开个chrome（狗头保命）。

当然我们不会介绍这么搞笑的方式。看这节的小标题就知道肯定是编码实现啦。

下面直接上代码。

创建进程的代码
```cpp
#include <iostream>
#include <cstdlib>

#include <Windows.h>

int main()
{
    std::cout << "Current Process Id (parent_process): " <<  GetCurrentProcessId() << std::endl;
    std::system("test.exe");
    return 0;
}
```

测试进程代码
```cpp
#include <iostream>
#include <windows.h>

int main()
{
    std::cout << "Current Process Id (child process): " <<  GetCurrentProcessId() << std::endl;
    std::cout << "Test program process!" << std::endl;
    return 0;
}
```

运行上述代码，可以得到如下输出：
```
Current Process Id (parent_process): 12752
Current Process Id (child process): 16092
Test program process!
```

由上述代码，可以了解到调用另一个进程最简单的方法是使用
`sysem`函数调用任意的可执行文件或者DOS命令。

但是这种调用方式是阻塞的，也就是说我们必须等待子进程返回后才能拥有当前进程的控制权。
并且我们无法通过API的方式将这个子进程关闭掉。下一节中将对Win32API提供的更全面的子进程过程控制函数进行简单介绍。

## 进程相关的Win32API应用
Win32 API提供了多种进程相关的功能如创建、销毁、获取状态等。
本节主要介绍`CreateProcess`, `TerminateProcess`以及`WaitForSingleObject`这三个常用函数。

1. `CreateProcess`创建一个进程

函数原型如下，
```cc
BOOL CreateProcessA(
  LPCSTR                lpApplicationName,      //应用程序名称，默认为nullptr
  LPSTR                 lpCommandLine,          //需要创建进程的命令行参数
  LPSECURITY_ATTRIBUTES lpProcessAttributes,    //进程属性
  LPSECURITY_ATTRIBUTES lpThreadAttributes,     //线程属性
  BOOL                  bInheritHandles,        //是否继承父进程的句柄
  DWORD                 dwCreationFlags,        //进程创建标记
  LPVOID                lpEnvironment,          //环境变量，如果为nullptr，则继承父进程
  LPCSTR                lpCurrentDirectory,     //指定当前运行目录
  LPSTARTUPINFOA        lpStartupInfo,          //启动信息
  LPPROCESS_INFORMATION lpProcessInformation    //新进程的信息
);
```

需要注意的是通常`lpApplicationName`为空指针，如该字符串不为空，则会用其作为可执行程序，
`lpCommandLine`则作为传入的命令行参数。

2. `TerminateProcess`终止一个正在运行的进程

函数模型
```cc
BOOL TerminateProcess(
  HANDLE hProcess,      //进程句柄
  UINT   uExitCode      //要使用的退出代码
);
```

结束进程函数相对比较简单，因为其不需要过多额外的设置。但是调用该函数首先需要对句柄拥有进程终止权限。
其中指定的`uExitCode`会作为该进程的退出代码。

3. `WaitForSingleObject`等待一个进程结束

函数模型
```cc
DWORD WaitForSingleObject(
  HANDLE hHandle,           //需要等待的句柄
  DWORD  dwMilliseconds     //等待时间
);
```

需要注意`hHandle`需要由同步权限。
`dwMilliseconds`如果是个自然数，则会等待相应的时间。如果是`INFINIT`,则会等到该句柄发出信号。


### 改写上一节中调用测试进程的函数
首先我们需要定义两个函数来创建和关闭子进程，
```cc
/**
 * @brief Create a Child Process
 *
 * @param cmd_str the command line string
 * @param process_info the pointer to the process info struct
 * @return true create the process successfully.
 * @return false catch a problem.
 */
bool CreateChildProcess(char* cmd_str, PROCESS_INFORMATION* process_info)
{
    STARTUPINFOW start_info;
    BOOL flag;
    wchar_t* wCmdStr = new wchar_t[strlen(cmd_str) + 1];
    size_t len;
    mbstowcs_s(&len, wCmdStr, strlen(cmd_str) + 1, cmd_str, strlen(cmd_str) + 1);
    // initialize the start up information struc
    ZeroMemory( &start_info, sizeof(start_info) );
    // set the size of the struct
    start_info.cb = sizeof(start_info);
    // Initialize the process information struct
    ZeroMemory(process_info, sizeof(process_info) );

    flag = CreateProcessW(
               NULL,           // nullptr defualt
               wCmdStr,        // command line
               NULL,           // default process attributes
               NULL,           // default thread attributes
               FALSE,          // no inherited
               0,              // default create flag
               NULL,           // default system environment varivable
               NULL,           // use the parent cwd
               &start_info,    // start info
               process_info );// process info
    delete[] wCmdStr;

    if ( !flag ) {
        // failure
        printf( "Error: (%d).n", GetLastError() );
        return 0;
    }
    return true;
}

/**
 * @brief Close the handle of child process
 *
 * @param process_info the pointer to process information
 * @return true
 */
bool CloseChildProcess(PROCESS_INFORMATION* process_info)
{
    // close the handles of process and thread.
    CloseHandle( process_info->hProcess );
    CloseHandle( process_info->hThread );
    return true;
}
```

然后我们将主函数该成如下形式，
```cc
int main()
{
    PROCESS_INFORMATION process_info;
    CreateChildProcess("test.exe", &process_info);
    CloseChildProcess(&process_info);
    return 0;
}
```

此时，我们观察到的控制台输出结果大概率是这样的，
```
Current Process Id (parent_process): 23152
```
因为创建进程后是非阻塞的，代码直接关闭了我们的子进程句柄，并退出。
但经过测试在不同控制台下运行时，也有可能会获得正确的结果。这就与控制台的进程参数有关了，我们不做讨论。
正确的做法应当等待子进程完成后再退出，将代码修改为
```cc
int main()
{
    PROCESS_INFORMATION process_info;
    CreateChildProcess("test.exe", &process_info);
    WaitForSingleObject(process_info.hProcess, INFINITE );    // wait for the subprocess finished
    CloseChildProcess(&process_info);
    return 0;
}
```
这是能够退出的情况，那么我们假设子进程是个死循环，或者有阻塞操作不能退出时，就需要使用terminate方法，强制结束。

我们在调用的子进程中加一个死循环。
此时再运行主进程发现，在正确输出调试信息后，控制台窗口无法退出了。这就是死循环的威力了。
为了正确关闭控制台窗口,我们把`WaitForSingleObject`注释掉，查看结果发现又变成了子进程没有输出，主进程被关闭了。
通过任务管理器，可以发现子进程依然在运行，所以子进程其实并没有被成功关闭。
此时我们就需要先将子进程终止掉，再关闭其句柄。

修改后的代码如下
```cc
int main()
{
    PROCESS_INFORMATION process_info;
    CreateChildProcess("test.exe", &process_info);
    Sleep(1000);
    TerminateProcess(process_info.hProcess, 0);
    CloseChildProcess(&process_info);
    return 0;
}
```

此时再运行，我们发现父子进程都能够正常退出，而且输出也正确了。
这是因为我们引入了休眠函数，使得标准输入输出有足够的时间被自动刷新。

## 小结
1. 在本文中介绍了创建子进程的两种方法
2. 介绍了Windows 关于进程创建的部分API
3. 实现了创建子进程的部分代码

代码请见[Github仓库](https://github.com/mrchipset/tutorial-notes/tree/master/Cpp/P9_Process_Series/S1_CreateChildProcess)

## 参考
1. [Windows API 教程 （四） 进程编程](https://lellansin.wordpress.com/2013/04/28/windows-api%E6%95%99%E7%A8%8B%EF%BC%88%E5%9B%9B%EF%BC%89-%E8%BF%9B%E7%A8%8B%E7%BC%96%E7%A8%8B)
2. [Microsoft Win32 API document](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa)