+++
author = "FanHe"
title = "LinuxC编程随笔（C语言流程控制-9）"
date = 2021-05-01T17:52:54+08:00
description = ""
categories = [
    "LinuxC编程"
]
+++

## 1. 一句话总结
---
介绍C语言中的顺序、选择、循环流程控制相关内容

## 2. 课程内容
---

### 2.1 goto使用注意事项

goto语句会造成程序的跳转混乱，如果确实goto可以带来代码的简洁（比如多重循环需要跳出时，使用goto直接从最内侧跳出是一个不错的使用方式）那么可以去用。但是其他场景最好不要用。

goto使用时不能够跳过变量的定义，比如下面的代码：

```
#include <stdio.h>
#include <stdlib.h>

int main(){

	goto Label;

	int j = 7;

Label:	

	printf("%d\n",j);
}

```
Label跳转语句掠过了j和k的定义，应该是有问题的。

不过上述代码我在测试的时候发现在GCC中把文件名后缀改成.c编译不会报错，但是改成.cpp使用g++编译时会报错


基本的控制语句，内容比较简单，略


## 3. 动手写代码
---

## 4. 课堂笔记
---

流程控制

顺序、选择、循环
NS图，流程图（工具Dia）
简单结构与复杂结构：自然流程

顺序：语句逐句执行
选择：出现了一种以上的情况
循环：重复执行某个动作（在某个条件成立的情况下）

关键字：
选择： if...else   if..elseif..else  switch-case
循环： while, do...while  for if-goto
辅助控制： continue, break

如果写if...else， else与最近的if进行匹配
超过多条语句的if或者else需要用花括号括起来

if...goto 可以构成循环（慎用：goto实现的是无条件的跳转，且不能跨函数跳转）



死循环：
	while(1);
	for(;;);
ctrl+c杀死死循环


## 5. 参考资料
---