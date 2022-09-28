+++
author = "FanHe"
title = "Redis事件处理"
date = 2022-09-28T09:49:20+08:00
description = ""
categories = [
 "Redis源码解析"
]
draft = false

+++

## 1. 概述

本文主要介绍Redis中的事件处理机制，使用的源码是Redis 0.07，也就是Redis第一个提交的版本



Redis中的事件处理在最初版本的时候比较粗糙，比如它使用的IO多路复用的机制是select，相对于之后Redis在Linux平台改进使用epoll的方式，select由于需要在用户态和内核态交互次数太多，效率比较低，另外select可以同时支持的监听事件也有上限。尽管如此，作为第一个实现，我们可以从中一窥redis事件处理的机制



## 2. 事件处理原理



在Redis中事件处理是在 `void aeMain(aeEventLoop *eventLoop);` 函数中实现的，它的代码比较简单

```go
void aeMain(aeEventLoop *eventLoop)
{
    eventLoop->stop = 0;
    while (!eventLoop->stop)
        aeProcessEvents(eventLoop, AE_ALL_EVENTS);
}

```

真正的代码在 aeProcessEvents 之中，并且传入的标签是 AE_ALL_EVENTS



### 2.1 Redis File事件

在Redis中的事件处理机制支持2种类型的事件：文件事件（File Event）和 时间事件（Time Event），文件事件说白了就是 套接字Socket事件，基本上是套接字的几种类型事件：

- 连接到达（监听listen套接字，等待客户端连接）

- 读取客户端请求

- 写入客户端回复



### 2.2 Redis Time事件



Time事件是真正的驱动事件循环一直进行下去的关键，通过不断的修改时间事件发生的时刻，驱动循环不停的进行下去



## 3. Redis事件的流转过程

Redis的事件的初始化是在redis.c 文件中，这个是redis服务端的可执行程序，redis维护着一个全局的 server变量（代表着redis服务端），这个server结构体中包含一个时间循环的 aeEventLoop 对象（就是事件队列），aeEventLoop中包含有File事件队列和Time事件队列



```c
typedef struct aeEventLoop {
    long long timeEventNextId;
    aeFileEvent *fileEventHead;
    aeTimeEvent *timeEventHead;
    int stop;
} aeEventLoop;

```

在Redis中的事件对象同样类似于Reactor中的方式，有一个处理函数和事件绑定，在发送事件之后会调用这个处理函数



### 3.1 初始化

初始化的过程在 `redis.c` 文件中的main函数中（也就是redis服务端可执行程序的入口点），初始化会加入两个事件对象（添加到aeEventLoop的链表中），这两个事件很显然一个是服务端的Listen套接字，另一个是时间事件对象

- Listen套接字的处理函数是：acceptHandler （很显然是阻塞在套接字accept调用）

- Time事件的处理函数是：serverCron（一个定时循环的事件处理函数，会进行内存统计、RDB文件的保存、同步主从节点等各种后台操作，定时1000ms执行一次，也就是1秒钟执行一次）



### 3.2 执行

事件的执行流转是在 `ae.c` 文件中的 `aeProcessEvents` 函数中进行的， 函数代码较多，贴一部分过来分析



```c

int aeProcessEvents(aeEventLoop *eventLoop, int flags)
{
    int maxfd = 0, numfd = 0, processed = 0;
    fd_set rfds, wfds, efds;
    aeFileEvent *fileEventHead = eventLoop->fileEventHead;
    aeTimeEvent *timeEventHead;
    long long maxId;
    AE_NOTUSED(flags);

    /* Nothing to do? return ASAP */
    if (!(flags & AE_TIME_EVENTS) && !(flags & AE_FILE_EVENTS)) return 0;

    FD_ZERO(&rfds);
    FD_ZERO(&wfds);
    FD_ZERO(&efds);

    /* Check file events */
    if (flags & AE_FILE_EVENTS) {
        while (fileEventHead != NULL) {
            if (fileEventHead->mask & AE_READABLE) FD_SET(fileEventHead->fd, &rfds);
            if (fileEventHead->mask & AE_WRITABLE) FD_SET(fileEventHead->fd, &wfds);
            if (fileEventHead->mask & AE_EXCEPTION) FD_SET(fileEventHead->fd, &efds);
            if (maxfd < fileEventHead->fd) maxfd = fileEventHead->fd;
            numfd++;
            fileEventHead = fileEventHead->next;
        }
    }
    /* Note that we want call select() even if there are no
     * file events to process as long as we want to process time
     * events, in order to sleep until the next time event is ready
     * to fire. */
    if (numfd || ((flags & AE_TIME_EVENTS) && !(flags & AE_DONT_WAIT))) {
        int retval;
        aeTimeEvent *shortest = NULL;
        struct timeval tv, *tvp;

        if (flags & AE_TIME_EVENTS && !(flags & AE_DONT_WAIT))
            shortest = aeSearchNearestTimer(eventLoop);
        if (shortest) {
            long now_sec, now_ms;

            /* Calculate the time missing for the nearest
             * timer to fire. */
            aeGetTime(&now_sec, &now_ms);
            tvp = &tv;
            tvp->tv_sec = shortest->when_sec - now_sec;
            if (shortest->when_ms < now_ms) {
                tvp->tv_usec = ((shortest->when_ms+1000) - now_ms)*1000;
                tvp->tv_sec --;
            } else {
                tvp->tv_usec = (shortest->when_ms - now_ms)*1000;
            }
        } else {
            /* If we have to check for events but need to return
             * ASAP because of AE_DONT_WAIT we need to se the timeout
             * to zero */
            if (flags & AE_DONT_WAIT) {
                tv.tv_sec = tv.tv_usec = 0;
                tvp = &tv;
            } else {
                /* Otherwise we can block */
                tvp = NULL; /* wait forever */
            }
        }

        retval = select(maxfd+1, &rfds, &wfds, &efds, tvp);
        if (retval > 0) {
            fileEventHead = eventLoop->fileEventHead;
            while(fileEventHead != NULL) {
                int fd = (int) fileEventHead->fd;

                if ((fileEventHead->mask & AE_READABLE && FD_ISSET(fd, &rfds)) ||
                    (fileEventHead->mask & AE_WRITABLE && FD_ISSET(fd, &wfds)) ||
                    (fileEventHead->mask & AE_EXCEPTION && FD_ISSET(fd, &efds)))
                {
                    int mask = 0;

                    if (fileEventHead->mask & AE_READABLE && FD_ISSET(fd, &rfds))
                        mask |= AE_READABLE;
                    if (fileEventHead->mask & AE_WRITABLE && FD_ISSET(fd, &wfds))
                        mask |= AE_WRITABLE;
                    if (fileEventHead->mask & AE_EXCEPTION && FD_ISSET(fd, &efds))
                        mask |= AE_EXCEPTION;
                    fileEventHead->fileProc(eventLoop, fileEventHead->fd, fileEventHead->clientData, mask);
                    processed++;
                    /* After an event is processed our file event list
                     * may no longer be the same, so what we do
                     * is to clear the bit for this file descriptor and
                     * restart again from the head. */
                    fileEventHead = eventLoop->fileEventHead;
                    FD_CLR(fd, &rfds);
                    FD_CLR(fd, &wfds);
                    FD_CLR(fd, &efds);
                } else {
                    fileEventHead = fileEventHead->next;
                }
            }
        }
    }
    /* Check time events */
    if (flags & AE_TIME_EVENTS) {
        timeEventHead = eventLoop->timeEventHead;
        maxId = eventLoop->timeEventNextId-1;
        while(timeEventHead) {
            long now_sec, now_ms;
            long long id;
            //跳过无效事件（理论上无法产生）
            if (timeEventHead->id > maxId) {
                timeEventHead = timeEventHead->next;
                continue;
            }
            //如果当前事件大于或等于事件的事件，那么执行这些事件
            aeGetTime(&now_sec, &now_ms);
            if (now_sec > timeEventHead->when_sec ||
                (now_sec == timeEventHead->when_sec && now_ms >= timeEventHead->when_ms))
            {
                int retval;

                id = timeEventHead->id;
                retval = timeEventHead->timeProc(eventLoop, id, timeEventHead->clientData);
                /* After an event is processed our time event list may
                 * no longer be the same, so we restart from head.
                 * Still we make sure to don't process events registered
                 * by event handlers itself in order to don't loop forever.
                 * To do so we saved the max ID we want to handle. */
                if (retval != AE_NOMORE) {
                    aeAddMillisecondsToNow(retval,&timeEventHead->when_sec,&timeEventHead->when_ms);
                } else {
                    aeDeleteTimeEvent(eventLoop, id);
                }
                timeEventHead = eventLoop->timeEventHead;
            } else {
                timeEventHead = timeEventHead->next;
            }
        }
    }
    return processed; /* return the number of processed file/time events */
}


```



整个过程是这样的：

（1）首先需要搜集一下文件队列中是不是存在需要select的事件

（2）如果存在的话，我们需要找一个合适的阻塞时间，这里就涉及到select函数的参数中最后一个timeval，它的含义如下：

select最后一个参数timeout的设置情况：

1. 永远等待，设置 NULL
2. 等待一个固定的时长，正常设置值即可
3. 完全不等待，只是轮询一遍（timeval的两个成员都必须设置为0）

这里的做法是：寻找距离当前时间最近的下一个时间事件之间的空隙，依次作为我们想阻塞的时长，这样就避免了我们干扰后续是时间事件的执行（但是实际上是会干扰的，因为我们的文件事件的处理不可能不耗时），但是如果阻塞期间一直没有事件过来，那么我们在到时间后也会继续往下执行，这样正好下一个时间事件也可以被执行了（相当于我们没有阻塞时间事件）

（3）如果我们监听到了文件事件（socket事件）一般是有新的连接到达，新的客户端连接的请求过来，这样我们会调用之前注册的函数处理客户端socket事件

（4）处理之后我们会进入到时间处理部分

时间处理中我们会查看已经超过当前时间是事件，并处理它们

（5）如果处理函数返回的是-1（NO_MORE），那么我们删除该时间事件（表明只有一次）

> 事实上在0.07的代码中这种类型的事件不存在

（6）如果处理函数返回一个值，那么我们把这个值加到处理过的事件上，生成一个新的“未来”的事件，于是保证了时间事件队列中永远有事件存在，因此程序会一直被驱动无限循环下去

> 那么时间事件循环是如何跳出的呢？如果我们检测到当前事件小于时间事件队列中的值，说明时间事件还未到来，此时我们跳出了时间事件循环部分，进入到外层的循环 mainLoop中，只要stop != 0，会继续开始事件循环


































































