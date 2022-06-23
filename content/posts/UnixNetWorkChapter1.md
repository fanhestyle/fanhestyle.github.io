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
#include <arpa/inet.h>
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <unistd.h>

#define SERV_PORT 8899
#define BUF_SIZE 1024

int main(int argc, char **argv)
{
    if (argc != 2)
    {
        fprintf(stderr, "Usage: %s <IPaddress>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0)
    {
        perror("socket()");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in servaddr;

    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(SERV_PORT);
    int ret = inet_pton(AF_INET, argv[1], &servaddr.sin_addr);
    if (ret == 0)
    {
        fprintf(stderr, "Error IPaddress\n");
        exit(EXIT_FAILURE);
    }
    else if (ret < 0)
    {
        perror("inet_pton()");
        exit(EXIT_FAILURE);
    }

    socklen_t servaddr_len = sizeof(servaddr);
    ret = connect(sockfd, (struct sockaddr *)&servaddr, servaddr_len);
    if (ret < 0)
    {
        perror("connect()");
        exit(EXIT_FAILURE);
    }

    char buf[BUF_SIZE];
    int n;

    while (1)
    {
        if ((n = read(sockfd, buf, BUF_SIZE)) < 0)
        {
            if (errno == EINTR)
            {
                continue;
            }
            else
            {
                perror("read()");
                exit(EXIT_FAILURE);
            }
        }
        else if (n == 0)
        {
            break;
        }

        write(STDOUT_FILENO, buf, n);
    }

    exit(EXIT_SUCCESS);
}

```

## 7. 简单的时间获取服务端

```
#include <arpa/inet.h>
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <time.h>
#include <unistd.h>

#define SERV_PORT 8899
#define BUF_SIZE 1024
#define LISTENQ 1024

int main(int argc, char **argv)
{
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0)
    {
        perror("socket()");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in servaddr;
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(SERV_PORT);
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);

    int ret = bind(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));
    if (ret < 0)
    {
        perror("bind()");
        exit(EXIT_FAILURE);
    }

    listen(sockfd, LISTENQ);
    if (ret < 0)
    {
        perror("listen()");
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in raddr;
    while (1)
    {
        socklen_t raddr_size = sizeof(raddr);
        int connfd = accept(sockfd, (struct sockaddr *)&raddr, &raddr_size);
        char raddrstr[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, (void *)&raddr.sin_addr, raddrstr, INET_ADDRSTRLEN);
        fprintf(stdout, "connection from %s:%d\n", raddrstr, htons(raddr.sin_port));

        if (connfd < 0)
        {
            perror("accept()");
            exit(EXIT_FAILURE);
        }

        char buf[BUF_SIZE];
        bzero(buf, sizeof(buf));
        time_t ticks = time(NULL);
        snprintf(buf, sizeof(buf), "%.24s\r\n", ctime(&ticks));

        write(connfd, buf, strlen(buf));

        close(connfd);
    }
}
```


## 8. 本章习题Exercises  

1. 查看网络环境  

书中使用的是 netstat 命令，当前该命令在Linux中已经过时，可以使用ip和ss命令（iproute2）替换 

netstat -i    用于输出计算机的网络接口信息， 对应的是 ip link 命令   

netstat -r    用于输出路由信息   对应的事 ip route 命令  

2. 运行书中给出的客户端和服务端的例子，尝试使用不同的ip运行 

可以使用multihomed的机器进行测试，可以指定任意网口的ip都可以，比如我本机运行的时候可以指定lo的ip地址（127.0.0.1），也可以指定任何一个网口的地址   

3. 把socket的第一个参数修改为9999（AF_INET改成9999），会如何报错？

报错如下： socket(): Address family not supported by protocol  提示协议族不对   

4. 修改客户端的程序，统计while循环中read被调用的次数并输出  

由于程序传输的内容特别少（仅仅只有一个日期）并且在本机上或者局域网环境中测试，因此测试出来的结果都是一次，事实上在TCP传输中每次读到的数据都是不一定的（TCP流式套接字的特性），但是可以保证的是客户端和服务端传输的完整内容是一样的













