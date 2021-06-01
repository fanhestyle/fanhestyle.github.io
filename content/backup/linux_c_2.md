+++
author = "FanHe"
title = "LinuxC编程随笔（GCC和Vim的基本用法-2）"
date = 2021-04-14T10:00:12+08:00
description = ""
categories = [
    "LinuxC编程"
]
draft=false
+++

## 1. 一句话总结
---
主要介绍GCC编译工具和Vim编辑器的基本用法

## 2. 课程内容
---

### 2.1 vim使用技巧

vim编辑器的配置文件在/etc/vim/vimrc中（环境：Ubuntu18.06下配置文件），这个文件是一个全局的配置，建议在自己的家目录中配置自己使用的vim环境脚本，配置文件 .vimrc文件（注意文件名以点开头）

基本的vim使用方式参考vim官方手册，可以尝试安装一些vim的插件来扩展vim功能，比如


### 2.2 GCC使用参数

GCC是GNU提供的一系列编译器的集合，可以支持多种编程语言的编译，其中最常用的是C/C++的编译。

GCC的命令行和参数众多，GCC官网提供的GCC手册有上千页，想要全部了解清楚难度较大，一般来说我们只需要了解GCC基本的用法即可[^1]

1. -o 参数指定生成文件

当不指定任何参数时，默认生成 `a.out` 文件

```
# 将main.c编译生成hello可执行文件
gcc main.c -o hello
```
2. -Wall 打开所有警告

3. -E 输出预处理文件
在C/C++中通过 #include 语句会引入很多其他文件，通过 #ifdef 等预处理命令可以实现启用或者禁用一些代码。在使用 -E 参数时，可以让编译器为我们生成预编译之后的文件，在某些情况下便于我们观察出错的原因。

默认 -E 输出到控制台，可以通过重定向命令写入到文件中

```
$  gcc -E main.c > main.i
```

4. -S 输出汇编代码

```
$  gcc -S main.c > main.s
```

5. -C 产生编译后的.o文件和可执行文件 

```
# 输出main.o和a.out两个文件
$ gcc -C main.c
```
当使用 -c（小写字母c）时，只产生main.o文件

6. -l参数链接共享库

-l参数是一个常用的链接外部库的命令

```
$ gcc  main.c -o main -lCPPfile
```
代码会链接 libCPPfile库

7. 使用-D参数可以使用编译时的宏

```
$ gcc -DMY_MACRO main.c -o main
```
这个命令可以激活源代码中定义的 MY_MACRO宏

8. 使用参数-I指定头文件的文件夹
```
$ gcc -I/home/code/include main.c
```

### 2.3 打印额外信息
---

在C语言编程过程中，可以利用一些预定义的宏打印出一些有用的信息，帮助我们更好的调试程序，比如以下几个宏

```
    __func__   函数名
    __FILE__   文件名
    __DATE__   日期
    __TIME__   时间
    __LINE__   行号
```
使用方式如下：

```
// main.c 文件

#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("__func__:%s\n", __func__);
    printf("__FILE__:%s\n", __FILE__);
    printf("__DATE__:%s\n", __DATE__);
    printf("__TIME__:%s\n", __TIME__);
    printf("__LINE__:%d\n", __LINE__);

	return 0;
}

```
输入内容

```
__func__:main
__FILE__:main.c
__DATE__:Apr 27 2021
__TIME__:06:59:25
__LINE__:10
```


## 3. 动手写代码
---
略

## 4. 课堂笔记
---

hello.c
### 编译器gcc
	C源文件 --> 预处理 --> 编译 --> 汇编 --> 链接 --> 可执行文件

### 编辑器vim
	vim /etc/vimrc
	cp /etc/vimrc ~/.vimrc
	vim ~/.vimrc
	vim配置脚本以及常用快捷方式


## 5. 参考资料
---
- [GCC编译的全部参数](https://gcc.gnu.org/onlinedocs/gcc/Option-Summary.html)
- [Vim 配置入门](https://www.ruanyifeng.com/blog/2018/09/vimrc.html#:~:text=Vim%20%E7%9A%84%E5%85%A8%E5%B1%80%E9%85%8D%E7%BD%AE%E4%B8%80%E8%88%AC,%E4%B8%80%E4%B8%AA%E5%86%92%E5%8F%B7%EF%BC%8C%E5%86%8D%E8%BE%93%E5%85%A5%E9%85%8D%E7%BD%AE%E3%80%82)
- [vim插件管理器：Vundle的介绍及安装（很全）](https://blog.csdn.net/zhangpower1993/article/details/52184581)


[^1]:[15个最常用的GCC编译器参数](https://colobu.com/2018/08/28/15-Most-Frequently-Used-GCC-Compiler-Command-Line-Options/)

