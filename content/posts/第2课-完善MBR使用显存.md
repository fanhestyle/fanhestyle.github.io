+++
author = "FanHe"
title = "第2课 完善MBR并使用显存"
date = 2022-04-18T11:09:28+08:00
description = ""
categories = [
 "从零打造操作系统"
]
draft = false
+++

## 

## 1. 概述

本文完善之前编写的MBR，使用操作显存的方式来输出字符



## 2. 实模式下的内存布局

实模式下我们使用的内存是1MB的内存空间，这是从8086继承下来的遗产，基本上我们最后使用的是80386上的特性，由于需要使用这1MB的内存，介绍以下这1MB内存的布局



<img src="/img/osdev/realmode-1mb.png" title="" alt="1MB布局" data-align="center">



## 3. 操作显存

通过上面的内存布局，我们可以在显存中写入一些数据，这样显存中写入的数据最终会在屏幕上被打印出来，由于我们编写一个简易的操作系统，因此只关心显存中的文本显示区域，也就是内存地址在(0xB8000--0xBFFF)这段区域



在显存中写入数据的时候，我们一般是以2个字节为单位。低字节是字符的ASCII码，高字节是字符属性元信息。在高字节中，低4位是字符前景色，高4位是字符的背景色。颜色用RGB红绿蓝三种基色调和，第4位用来控制亮度，若置1则呈高亮，若为0则为一般正常亮度值。第7位用来控制字符是否闪烁（不是背景闪烁）



<img title="" src="/img/osdev/video-char.png" alt="显存字符" data-align="center">

## 4. 编码

有了上述基础，那么就可以不用借助BIOS中断提供的功能来打印字符，而是直接操作显存来打印，代码如下



```asm6502
.code16

.section .text

movw %cs, %ax
movw %ax, %ds
movw %ax, %es
movw %ax, %fs
movw $0x7c00, %sp
movw $0xb800, %ax
movw %ax, %gs

//清屏
movw $0x600, %ax
movw $0x700, %bx
movw $0x0, %cx
movw $0x184f, %dx
int $0x10

//显存中写入数据
movb $'1', %gs:0x0
movb $0xA4, %gs:0x1
movb $' ', %gs:0x2
movb $0xA4, %gs:0x3
movb $'M', %gs:0x4
movb $0xA4, %gs:0x5
movb $'B', %gs:0x6
movb $0xA4, %gs:0x7
movb $'R', %gs:0x8
movb $0xA4, %gs:0x9

jmp .

.org 510
.word 0xaa55


```
