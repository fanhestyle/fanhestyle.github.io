+++
author = "FanHe"
title = "第3课 基本的网络编程函数"
date = 2022-06-23T22:18:25+08:00
description = ""
categories = [
 "Unix网络编程"
]
draft = true
+++

# 1. 简介

在网络编程中有一些函数是有必要进行一些封装的，这是由于网络编程大部分时候都在读取套接字，而套接字中的数据是不稳定的，时有时无，依赖于具体的网络环境。因此对常用的系统调用 read和write进行必要的封装是非常有用的

# 2. 基本的系统调用

基本上两个最重要的系统调用是 read 和 write

```
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
```

## 2.1 read语义  

假设调用如下  

```
read(fd, buf, n); //备注n大于0,否则报参数错误
```

read的语义是：从fd描述符指向的设备中尝试读取n个字符，保存在buf缓冲区中  ，可能出现的情况如下：

1. 读取到n个字符，这时候会返回n，函数成功
2. 读取到m个字符(m<n)，这时候返回m，函数也算成功（有可能是遇到了EOF文件描述符结尾，最后的那次读取没有n个字符了）  
3. 返回值 == 0，说明读完了文件，遇到EOF了
4. 返回值 < 0，说明出错了，出错又分两种情况：
   4.1 假错：有可能是读取被信号打断，或者是读取到时候使用非阻塞模式读取，但是正好fd中无数据，前一种情况返回的是EINTR,后一种情况返回的是 EAGAIN(或则EWOULDBLOCK)  
   4.2 真错：返回相应错误，设置在errno上 

## 2.2 write语义

write语义基本上和read一样，但是一般情况下write基本都会成功，所以编码中的判断情况较read要松散一点  


# 3. 封装的函数  

封装的函数有：

```
readn
writen
readline

```

## 3.1 readn

readn的一种实现如下：

```
ssize_t readn(int fd, void *buff, size_t n)
{
    size_t nleft = n;
    ssize_t nread;
    char *ptr = buff;

    while (nleft > 0)
    {
        if (nread = read(fd, ptr, nleft) < 0)
        {
            if (errno == EINTR)
                nread = 0;
            else
                return -1;
        }
        else if (nread == 0)
            break;

        nleft -= nread;
        ptr += nread;
    }

    return (n - nleft);
}

```
语义如下：尝试读足nbytes个字符（否则一直循环read读取直到出错或者读完整个fd，或者读足n字符）

情况有：
1. fd的字符足够，因此一直读取直到读足够n个字符，返回值 == n
2. fd的字符不太足够(<n)，会一直读取直到读取到EOF（fd读完了），返回值 == m （m是读到的字符数）
3. 返回值 < 0（出错了）

总结：readn的返回值：（1）n  (2) m  (3) 0（说明文件就是EOF）(4) < 0 出错了

## 3.2 writen 

writen的实现如下：

```
ssize_t writen(int fd, const void *buff, size_t n)
{
    size_t nleft;
    ssize_t nwrite;

    const char *cur = buff;

    while (nleft > 0)
    {
        if ((nwrite = write(fd, cur, n)) <= 0)
        {
            if (nwrite < 0 && errno == EINTR)
                nwrite = 0;
            else
                return -1;
        }
        nleft -= nwrite;
        cur += nwrite;
    }

    return (n);
}

```
语义如下： writen一直尝试写出到fd中，直到写足n个字符，相比较于readn的实现，有几个细节需要注意：

1. writen的返回值只有2个：要么是n要么是-1,也就是说要么写足够n个字符，要么报错（因为一般来说写是都可以成功的，如果设备无错的话）  
2. 在write的时候，如果返回值是0（如果fd设备不是文件的话，那么行为是未定义的，因此也将其归纳为错误，也就是返回-1【也就是write没有nwrite==0退出的情况】

以上2点是编写writen需要特别注意的地方   


## 3.3 readline

(1)naive的实现，我们可以每次用read读取1个字符，知道遇到`\n`为止  

```
/* extremely slow version */
ssize_t readline(int fd, void *buff, size_t maxline)
{
    ssize_t n, rc;
    char c;
    char *ptr = buff;

    for (n = 1; n < maxline; n++)
    {
    again:

        if ((rc = read(fd, &c, 1)) == 1)
        {
            *ptr++ = c;
            if (c == '\n')
                break;
        }
        else if (rc == 0)
        {
            *ptr = 0;
            return (n - 1);
        }
        else
        {
            if (errno == EINTR)
                goto again;
            else
                return -1;
        }
    }
    *ptr = 0;
    return (n);
}
```
函数的作用如下：

尝试从fd中读取一行文本（如果读到maxline-1个字符的时候，仍然没有出现换行符，那么就返回读到的maxline-1个字符+尾部的0，一共是maxline个字符），如果在读到 maxline-1个字符之前遇到了换行符'\n'，那么返回带'\n'和尾部0的字符串， 另外在读取到maxline-1字符之前遇到了文件结束符，那么也返回读到的字符个数（返回的个数不包括0,但是返回后buff字符串中有尾0） 如果出错返回-1  


（2）一个更好点的版本  

由于我们需要大块文本之后，再判断内部的状态，因此我们需要得知文本缓冲区的内部状态，于是我们要自己维护一个内部缓冲区（而不是使用类似于stdio这样的不透明的缓冲区）   

```
#define MAXLINE 1024

static int read_cnt;
static char *read_ptr;
static char read_buf[MAXLINE];

static ssize_t my_read(int fd, char *ptr)
{
    if (read_cnt <= 0)
    {
    again:
        if ((read_cnt = read(fd, read_buf, sizeof(read_buf))) < 0)
        {
            if (errno == EINTR)
                goto again;
            else
                return -1;
        }
        else if (read_cnt == 0)
            return 0;
        read_ptr = read_buf;
    }
    read_cnt--;
    *ptr = *read_ptr++;
    return (1);
}

ssize_t readline_1(int fd, void *vptr, size_t maxline)
{
    ssize_t n, rc;
    char c;
    char *ptr = vptr;

    for (n = 1; n < maxline; n++)
    {
        if ((rc = my_read(fd, &c)) == 1)
        {
            *ptr++ = c;
            if (c == '\n')
                break;
        }
        else if (rc == 0)
        {
            *ptr = 0;
            return (n - 1);
        }
        else
            return -1;
    }

    *ptr = 0;
    return (n);
}

//暴露内部的状态给外部查看
ssize_t readlinebuf(void **vptrptr)
{
    if (read_cnt)
        *vptrptr = read_ptr;

    return (read_cnt);
}

```

使用我们自己写的myread来替换read函数，基本原理就是一次先读一大块数据，但是内部返回的时候在缓冲区中逐个传给外部的调用，一旦读完，那么再次读一大块  


# 4. golang中的网络IO接口

以下是golang中相同作用的函数  

C   ---->   Go

read        io.Reader中的Read方法
write       io.Writer中的Write方法
readn       io.ReadFull
writen      bufio.Writer.Write方法
readline    bufio.Reader.ReadSlice或者bufio.Scanner


# 5. 部分socket标准库函数一览

以下socket函数使用的并不多，但是还是需要了解一下  

getsockname 用户获取SOCKET中绑定的本地地址

getpeername 用于获取SOCKET中绑定的远端地址

```
#include <sys/socket.h>

int getsockname(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
int getpeername(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```










