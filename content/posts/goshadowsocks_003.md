+++
author = "FanHe"
title = "Go-Shadowsocks源码解析2"
date = 2022-09-22T07:25:20+08:00
description = ""
categories = [
 "GoShadowsocks2"
]
draft = false

+++

# 1. 概述

本文继续解析源码，解析的源码是 cipher.go文件

```textile
Initial commit
443dd019075b8367fd6db8a07d2967ecacbb855a
cipher.go
```

cipher.go主要提供一个函数 pickCipher，简单来说就是选择一个加密算法用于通信。关于加密算法这一块又是一个比较大的话题

密码学是现代网络应用的基石，我们在如今的网络世界中几乎无时无刻不在使用者加密，比较常见的是日常上网使用的https协议就是典型的密码学的应用，本文不打算大篇幅的从头开始论述整个加密/解密的理论，网络上有一篇非常完成的文章可以阅读([常用密码技术 | 爱编程的大丙](https://subingwen.cn/golang/cryptology/)) ，这篇文章对整个我们目前能够用到的密码学方面的知识进行了完整系统的介绍，相信比我在这里引用论述一遍更好

我们先看一下这个函数的原型

```go
func pickCipher(name string, key []byte) (core.StreamConnCipher, core.PacketConnCipher, error)
```

可以看到这个函数的参数是：

1. name：指定一种解密算法的名称，我们日常使用的加密名称列表：
   
   - xchacha20-ietf-poly1305
   - chacha20-ietf-poly1305
   - aes-256-gcm
   - aes-192-gcm
   - aes-128-gcm

2. key：指定一个密钥

由于算法都需要密钥这个参数，因此我们需要指定一下（其实类似于我们平常登录网站的密码）

由于后续代码中都设计到core模块中的加解密部分，因此我们先跳转到去剖析 core包中的加解密模块

# 2. 解析core包

```textile
Initial commit
443dd019075b8367fd6db8a07d2967ecacbb855a
对应文件：
core/doc.go
core/packet.go
core/stream.go
```

## 2.1 doc.go

该文件就一行代码，package core

应该是作者想写注释来着

## 2.2 packet.go

这个包提供一个定义，把PacketConnCipher定义成一个函数，这个函数的作用是对参数中的net.PacketConn进行一次封装

```go
type PacketConnCipher func(net.PacketConn) net.PacketConn
```

我们可以看看这个文件中的另一个函数

```go
func ListenPacket(network, address string, ciph PacketConnCipher) (net.PacketConn, error) {
    c, err := net.ListenPacket(network, address)
    return ciph(c), err
}
```

可以看到它的作用是将一个标准的UDP的连接，加上了一层封装，从字面值的意思来看应该是将一个标准的UDP的连接转换成加密的UDP连接。目前这里面涉及的都是抽象接口层的操作，具体使用的时候我们需要传入具体的加密算法构建的PacketConnCipher函数

## 2.3 stream.go

既然有报式套接字的UDP加密，那么怎么能少的了流式套接字TCP呢，就在stream.go这个文件中

该文件中定义了一个类型，类似于packet.go文件中定义的 PacketConnCipher，都是为标准的TCP或者UDP提供一层薄薄的加密加密封装

```go
type StreamConnCipher func(net.Conn) net.Conn
```

接着这个类实现了TCP进行通信的3个常用函数

- Listen

- Accept

- Dial

# 3. 网络编程基础知识环节

在看到上面的core包之后，如果初学者可能对此一头雾水，完全不知道如何下手了。事实上这是由于缺失网络知识这一环节，如果是作为100%的新手，那么需要恶补的知识可就多了，首先建议去找一本《计算机网络》的知识去学习，本文就不赘述这部分内容了。先打好基础

如果对网络有所了解，那么需要了解TCP-UDP编程的内容，特别是针对golang中如何进行tcp/udp编程，其实golang中的tcp-udp编程基本上就是 一个标准库 net 库，如果需要加强了解这方面的内容，可以去阅读 《Network Programming with go》[Introduction | Network Programming with Go (golang)](https://ipfs.io/ipfs/QmfYeDhGH9bZzihBUDEQbCbTc5k5FZKURMUoUvfmc27BwL/index.html)

关于TCP-UDP这部分编程的内容我就不讲了，因为讲解也是从网络上搬运一些内容到这里，意义不大，我写这一系列文章是引导读者去一步步了解自己完成整个项目需要哪些知识点，以及如何获取这些知识点，我不可能把所有内容都聚集于此，这样一方面内容太繁杂，脱离了本项目的中心，另一方面还是引导读者去自己发现学习。 建议不熟悉的读者一定要恶补这一块知识点，才能接着往下学

# 4. shadowaead包解析

```textile
Initial commit
443dd019075b8367fd6db8a07d2967ecacbb855a
对应文件：

shadowaead/doc.go
shadowaead/packet.go
shadowaead/stream.go
```

## 4.1 doc.go

在doc.go中作者给出了一大段的关于这个包中是如何封装TCP和UDP的，简单论述一下。首先本包实现了对TCP-UDP协议中数据的AEAD保护，关于AEAD是什么，可以在网络上搜索一下，也可以查看下面的视频了解一下 [What Are AEAD Ciphers? - YouTube](https://www.youtube.com/watch?v=od44W45sCQ4)

- AEAD 对TCP报文的封装

（1）TCP报文以一个随机数开始，随机数每发送一次自增1

（2）每一个记录都是加密的，一个报文中可以包含多个记录

（3）记录中的长度是加密的，记录也是加密的

- AEAD对UDP报文的封装

（1）UDP是一个报文一个报文独立加密，和TCP不同，这些报文之间没有任何关系

（2）一个TCP报文加密以一个随机数开头

（3）接下来就是加密的载荷体

整体的加密格式如下图所示

- TCP

<img src="/img/goshadow/AEAD-TCP.png" title="" alt="架构图" data-align="center">

- UDP

<img src="/img/goshadow/AEAD-UDP.png" title="" alt="架构图" data-align="center">

## 4.2 stream.go

这个文件主要是用来对流式套接字进行封装，将未加密的流按照4.1中给出的设置方法进行封装，主要的两个方法是reader提供的Read和writer提供的Write

```go
func (r *reader) Read(b []byte) (int, error) 


func (w *writer) Write(b []byte) (int, error) 
```





## 4.3 packet.go

这个文件主要是用来对报式套接字进行封装，主要定义了一个结构体，该结构体仅仅对读写的套接字和加密方法进行简单的封装

```go
type packetConn struct {
    net.PacketConn
    cipher.AEAD
}
```

之后使用Pack和Unpack对于即将发送和接收到的报文进行加密和解密，加密和解密的方法在4.1 doc.go 中有详细的说明

```go
func Pack(dst, plaintext []byte, aead cipher.AEAD) ([]byte, error) 

func Unpack(dst, pkt []byte, aead cipher.AEAD) ([]byte, error) 
```
