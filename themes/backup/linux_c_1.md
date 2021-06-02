+++
author = "FanHe"
title = "LinuxC编程随笔（C语言学习大纲-1）"
date = "2021-04-14T08:24:03+08:00"
description = ""
categories = [
    "LinuxC编程"
]
+++

## 1. 一句话总结
---

本课主要介绍C语言的发展史以及C语言学习的内容大纲



## 2. 课程相关扩展
---

### 2.1 C语言的可移植性
  C语言的可移植性并不是类似与Java那种在可执行层面的可移植性，更多的是体现在源代码级别的可移植性。而Java的可移植性更多的是指Java可执行文件的移植性。
  
  比如我们可以在Windows平台上有Java编写的一个可执行程序，在Linux或者MacOS X上也可以直接运行这个编译好的程序（前提是在Linux和MacOS X上有安装Java虚拟机），而C程序更多的是指用它编写的源代码，可以把在Windows上用C编写的源代码放到Linux和Mac OS X上去编译

 ### 2.2 课程学习开发环境
  老师提到课程中她使用的是CentOS 6 (64位)，我个人使用的测试环境是 VMWare Station 上安装的CentOS 7.4.1708 以及Ubuntu 18.04.5 虚拟机（都是amd64位版本）


## 3. 动手写代码
---

（略）

## 4. 本课摘要
---

### C语言发展史
    1960    原型A语言 -> ALGOL语言
    1963    CPL语言
    1967    BCPL语言
    1970    B语言
    1973    C语言

### C语言特点
    1. 基础性语言
    2. 语法简介，紧凑，方便，灵活
    3. 运算符，数据结构丰富
    4. 结构化，模块化编程
    5. 移植性好，执行效率高
    6. **允许直接对硬件操作** （FPGA DSP）

### C语言学习建议
    1. 概念的正确性
    2. 动手能力
    3. 阅读优秀的程序段
    4. 大量练习，面试题

### C课程讲解思路
    1. 基本概念
    2. 数据类型，运算符和表达式
    3. 输入输出专题（标准库IO）
    4. 流程控制（顺序、分支、循环）
    5. 数组
    6. 指针
    7. 函数
    8. 构造类型（结构struct、Union）
    9. 动态内存管理
    10. 调试工具和调试技巧（gdb，make）
    11. 常用库函数

### 平台介绍
    64位的redhat6，vim，gcc(make)





## 5. 参考资料
---

- [1. C programming language](https://en.wikipedia.org/wiki/C_(programming_language))
- [2. C语言](https://zh.wikipedia.org/wiki/C%E8%AF%AD%E8%A8%80)



