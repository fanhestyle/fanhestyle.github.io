+++
author = "FanHe"
title = "LinuxC编程随笔（编码建议-3）"
date = 2021-04-27T22:10:30+08:00
description = ""
categories = [
    "LinuxC编程"
]
+++

## 1. 一句话总结
---

编码中的一些小建议

## 2. 课程内容
---

### 2.1 引入函数时一定要包含头文件

在C语言中有一个叫做 "implicit function declaration"（隐式函数声明）的特性（或者说时缺陷），在C89中是这样处理的，但是在C99中已经明确移除了这个缺陷，但是GCC在编译的时候即使加上C99的标识仍然不报错[^1]

基本的描述是：

** 当编译器发现一个没有声明的函数调用时，它会假设这个函数返回 int 类型的返回值 **

在实际编码中，最好是不要使用隐式的函数声明，在使用一个函数时明确引入包含它的头文件（在GCC中默认会给出警告，可以通过设置 -Werror=implicit-function-declaration 参数让GCC遇到隐式函数声明时报错）

### 2.2 main函数没有显式给出返回值

视频中的讲解在未给出返回值时，提及返回的是printf的返回值，但是与我个人的验证并不相同。

我的环境：Ubuntu 18.06使用GCC的版本 gcc (Ubuntu 8.4.0-1ubuntu1~18.04) 8.4.0

在编写下面的代码时：

```
#include <stdio.h>
#include <stdlib.h>

int main()
{
	printf("hello,world\n");
//	return 0;
}
```
生成可执行程序，执行可执行程序后，通过在Bash上打印出$? 结果仍然是 0.

安装参考资料中的提示，这个应该后续的C标准有做更改，在此写出我的验证。

### 2.3 注释编写方式
（1）短注释可以用双斜线 //
（2）文件最前面的解释，可以用 /* */
（3）大段的注释建议用
    #if 0
    #endif



## 3. 动手写代码
---

### 3.1
由于未引入头文件造成段错误例子

```
#include <stdio.h>
#include <stdlib.h>	
//#include <string.h>
#include <errno.h>

int main(void)
{
	FILE *fp;
	
	fp = fopen("tmp","r");

	if (fp == NULL)
	{
		fprintf(stderr,"fopen error: %s\n", strerror(errno));
		exit(-1);
	}

	return 0;	
}	
```


## 4. 课堂笔记
---

一、基本概念

1. 以helloworld为例对写程序的思路提出如下要求：

（1）头文件正确包含的重要性
（2）以函数未单位来进行进程程序编写
（3）声明部分+实现部分【先声明后使用】（新的C版本反倒建议随时定义随时使用，类似于C++，选择一种风格保持一致即可）
（4）main 函数记得return
（5）多用空格和空行，增强代码可读性
（6）添加适量的注释，未来某个时候你会感谢当时的你

2. 算法：解决问题的方法（流程图、NS图、有限状态机(FSM)）

3. 程序：用某种语言实现算法

4. 进程：

5. 防止写越界、防止内存泄漏、谁打开谁关闭，谁申请谁释放



## 5. 参考资料
---

- [关于gcc内置函数和c隐式函数声明的认识以及一些推测](https://www.cnblogs.com/hujichen/p/5612826.html)
- [C语言隐式声明与GCC内建函数](https://blog.csdn.net/neuq_jtxw007/article/details/78224297)
- [c语言中，如果main函数的末尾没有return语句将会有什么影响?](https://www.zhihu.com/question/338814178)



[^1]:[stackoverflow: Implicit function declarations in C](https://stackoverflow.com/questions/9182763/implicit-function-declarations-in-c/9182835#:~:text=An%20implicitly%20declared%20function%20is,type%20of%20the%20arguments%20match).)