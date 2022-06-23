+++
author = "FanHe"
title = "第1课 传输层协议"
date = 2022-06-23T21:18:25+08:00
description = ""
categories = [
 "Unix网络编程"
]
draft = true
+++

## 1. 概述

本章主要介绍Unix网络编程七层架构中的传输层，主要讨论传输层的协议（TCP和UDP），文中的SCTP目前暂不讨论   



## 2. 传输层协议

主要包括有 
- tcp
- udp
- sctp

其中tcp和sctp说可靠的传输协议，而udp这不是

### 2.1 TCP

### 2.2 UDP

### 2.3 SCTP


## 3. 习题

1. 我们一直在讨论IPv4 和 IPv6 那 IPv5 呢？

IPv5是一个协议名 Internet Stream Protocol, IPv0并不存在

2. 为什么在TCP传输的时候，如果对端没有发送MSS，我们这一端会封包536字节大小的Segment？

首先，IPV4需要传输的最小单元是576个字节，为什么是576呢？


576 bytes was the standard buffer size in DECnet Phase IV use inside DEC.

Why 576? Because a FILES-11 disk block was 512 bytes, you needed a few more bytes for protocol overhead, and the allocation granularity on a PDP-11 memory management unit was 64 bytes.

于是我们用 576 - 20（一般的IPV4报头）-20（一般的TCP报头） = 536 






