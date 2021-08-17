---
title: Ubuntu 基于VSCode配置Spring开发环境
categories:
  - 环境配置
tags:
  - linux系统
  - Java
  - Spring
abbrlink: 4b039ca6
date: 2021-08-17 17:28:08
---

本文将记录如何在Ubuntu系统上配置基于VSCode的springboot开发环境，将会涉及到java sdk、maven工具的安装和配置。
个人比较喜欢VSCode这个编辑器，因此也会介绍如何在vscode当中配置插件进行spring的开发。
<!-- more -->

## 安装openjdk-11
直接利用`apt`工具进行相应的安装操作，

`sudo apt update && sudo apt install openjdk-11-jdk`

安装完成后，编辑/etc/environment这个文件进行配置`JAVA_HOME`和`CLASSPATH`两个环境变量

添加以下内容：
```
JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64/bin/java"
CLASSPATH=".:$JAVA_HOME/lib:$JAVA_HOME/jre/lib"
```
之后重启或者source一下文件即可生效。

借用一个HelloWorld.java进行测试
```java
public class HelloWorld {
    // Java程序的入口方法，程序将从这里开始执行
    public static void main(String[] args) {
        // 向控制台打印一条语句
        System.out.println("Hello World");
    }
}
————————————————
版权声明：本文为CSDN博主「微步_ym」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/yiminghd2861/article/details/107271699
```
使用`javac`进行编译，之后运行的结果如下
```shell
$ vim HelloWorld.java
$ javac HelloWorld.java
$ java HelloWorld
Hello World
```
如果输出正常，，至此JAVA的配置就结束了。
## 配置VSCode进行Spring开发
**首先安装`maven`**

`sudo apt update && sudo apt install maven`

在终端中输入`mvn --version`测试maven是否正确安装即可。

**安装vscode对应的插件包**
1. *Extension Pack for Java*
2. *Spring Boot Exitension Pack*

**可选的配置**
1. 配置`mvn`可执行文件的路径
2. 配置`mvn`使用的配置文件
3. 在配置文件中配置`JAVA`相关的参数，如JAVA_HOME
4. 配置`maven`的镜像为阿里云镜像，加速工程创建

**新建一个Spring的程序**
1. 使用VSCode的Spring插件创建项目
    
    选择DevTools可以热更新代码，无需重新启动
    
    选择Web，继承了tomcat、SpringMVC

    选择Lombok，智能生成setter、getter和toString等接口，不需要额外手动实现代码
2. 修改pom.xml，使用阿里云镜像
```xml
<repositories>
	<repository>
		<id>alimaven</id>
		<name>aliyun maven</name>
		 <layout>default</layout>
		<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
	</repository>
</repositories>

<pluginRepositories>
	<pluginRepository>
		   <id>alimaven</id>
		   <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
	   </pluginRepository>
</pluginRepositories>

```

3. VSCode Debug，配置Launch，运行demoApplication

    运行之后可以看到控制台开始有输出，如果使用浏览器打开[http://localhost:8080](http://localhost:8080) 会报错。
    此时可以确认springboot有运行。

4. 新建一个Controller响应请求，能够输出一些信息

    新建一个Controller包，在里面新建一个HomeController.java文件，输入如下代码：
    ```java
    package com.example.demo.Controller;

    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;


    @RestController
    public class HomeController {
        @RequestMapping("/")
        public String home()
        {
            int value = 10;
            return "Hello Spring Boot!";
        }
    }
    ```

    直接刷新浏览器页面，就可以看到Hello Spring Boot的字符串输出字样






