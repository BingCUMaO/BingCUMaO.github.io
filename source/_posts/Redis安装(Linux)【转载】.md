---
title: Redis安装(Linux)【转载】
date: 2020-09-12 21:30:40
categories:
- 技术博客
tags:
- redis
- linux
---







转载内容：



[Linux安装Redis，设置开机自启动,并修改端口和密码](https://blog.csdn.net/y532798113/article/details/82426643)



注意，若在执行`make`命令之前，需提前确认是否按照了c++的编译器，若在没有相关编译器的支持下执行了`make`命令，之后即使是执行了`yum -y install gcc-c++`命令（用来安装c++编译器的命令），也会报一些奇怪的错误。



因此，保险起见最后在执行`make`命令之前就执行`yum -y install gcc-c++`命令。（另外地，`yum -y install gcc-c++`命令安装c++编译器的位置是全局的）