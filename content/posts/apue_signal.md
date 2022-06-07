+++
author = "FanHe"
title = "Linux信号"
date = 2022-06-07T22:01:01+08:00
description = ""
categories = [
 "Linux系统编程"
]
draft = false
+++

## 

## 1. 概述

信号是Linux系统中一种异步的机制，在Linux系统中信号种类繁多，定义在 <signal.h>头文件中，信号的编号是一个正整数（没有0号信号），信号产生的情况非常多，列举几种常用的信号：

（1）键盘上操作快捷键的几个信号：    
- Ctrl+C：产生信号 SIGINT
- Ctrl+\（反斜杠)：SIGQUIT
- Ctrl+Z：产生信号：SIGSTOP

（2）使用kill函数    
kill包含在Manual手册的(1)(2)中，它既是一个命令也是一个系统调用，系统调用如下：    
```
#include <sys/types.h>
#include <signal.h>

int kill(pid_t pid, int sig);
```

## 2. 信号的处理

主要有3种方式：    
- 忽略信号（写一行代码）【有两个特殊信号无法被忽略：SIGKILL和SIGSTOP】
- 捕获信号（写一个函数）【有两个特殊信号无法被捕获：SIGKILL和SIGSTOP】
- 默认的动作（不用写任何代码）   








