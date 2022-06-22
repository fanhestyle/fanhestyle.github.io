+++
author = "FanHe"
title = "第1课 Unix网络开发概述"
date = 2022-06-22T22:10:25+08:00
description = ""
categories = [
 "Unix网络编程"
]
draft = false
+++

## 1. 概述

本章主要介绍Unix网络编程这一系列文章的相关介绍


## 2. 环境  

平台： Linux平台，使用Ubuntu 22.04作为测试平台，Linux内核版本 5.15.0-39-generic   
参考书籍：主要是三本书  
1. 《Unix网络编程第三版卷一》  
2. 《Unix高级编程第三版》  
3. 《TCP-IP详解答-卷一》  

另外有部分Linux相关的内容参考 《The Linux Programming Interface》    

## 3. 总体介绍  

本系列文章主要已编码为主，重点内容以及课后的习题也列出，便于日后查阅，以后尽可能查阅使用这些文章即可，减少翻书的可能性。    

## 4. 本章内容  

主要介绍Unix编程的基本内容，包括：  

- 网络编程基本的模型（客户端-服务器模型）  

- OSI七层架构  

- 编写一个基本的时间获取客户端和服务端  


## 5. errno简介  

errno是Unix中的错误标志码，大部分都函数出错都会返回一个errno，我们可以查询errno得知出错的原因，操作errno的主要有两个函数  

1. perror传入一个说明，这个说明会作为出错原因的前缀   
2. strerror传入一个errno的错误码，得到详细的错误字符串  

```
#include <stdio.h>
void perror(const char *s);
```
```
#include <string.h>
char *strerror(int errnum);
```

另外在很早的时候errno是作为一个全局变量定义的，一般定义为 `extern int errno` 但是目前版本中定义为一个线程范围的变量，它的定义

```
extern int *__errno_location (void) __THROW __attribute_const__;
# define errno (*__errno_location ())

```
可以看到errno被定义成一个函数调用的解引用形式，它调用的函数返回值是一个int*类型，因此最后的结果也是一个int类型的，当使用多线程的时候，它调用的这个函数应该是被设置为线程内的一个函数，因此返回值是Pthread-Local的变量，因此多线程之间不会共享这个变量，不会造成问题  

## 6. 简单的时间获取客户端  

```

```

## 7. 简单的时间获取服务端

```

```

## 8. 稍微修改程序的IPV6客户端 

```

```

## 9. 稍微修改程序的IPV6服务端 

```

```

## 10. 本章习题Exercises  














