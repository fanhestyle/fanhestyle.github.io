+++
author = "FanHe"
title = "LinuxC编程随笔（C语言中的常量-5）"
date = 2021-04-28T23:00:24+08:00
description = ""
categories = [
    "LinuxC编程"
]
+++

## 1. 一句话总结
---

介绍C语言中各种类型的常量值

## 2. 课程内容
---

### 2.1 数组名是一个常量

```
面试提醒：以下哪些是非法的常量： '\012' '\345' '\138`

结论是 '\138'，因为8进制不包含8这个数，三位8进制的数加上反斜杠用来转义得到一个字符常量，因此是错误的

```

### 2.2 宏替换带来的一些问题

观察以下程序的输出

```
#include <stdio.h>
#define PI 3.14
#define MAX(a,b) ((a) > (b) ? (a) : (b))

int main()
{
	int i = 5;
	int j = 3;

	printf("i=%d\tj=%d\n", i,j);
	printf("Max: %d\n", MAX(i++, j++));
	printf("i=%d\tj=%d\n",i,j);

	return 0;
}
```
程序输出的结果

```
i=5	j=3
Max: 6
i=7	j=4

```
在替换MAX(i++,j++)后，表达式变成了

```
 printf("Max: %d\n", ((i++) > (j++) ? (i++) : (j++)));
```
较大的那个数一定会自增2次，这个可能并不是我们想要的结果。因此在使用宏替换时要特别小心

解决方式，标准C没有办法解决这样的情形，使用GCC的扩展
```
#define MAX(a,b) ({typeof(a) A=a,B=b; ((A)>(B) ? (A):(B));})
```
下面这段代码在GCC中没有问题，可以正常编译通过

```
#include <stdio.h>
#define PI 3.14
//#define MAX(a,b) ((a) > (b) ? (a) : (b))

#define MAX(a,b) ({int  A=a,B=b; ((A)>(B) ? (A):(B));})

int main()
{
	int i = 5;
	int j = 3;

	printf("i=%d\tj=%d\n", i,j);
	printf("Max: %d\n", MAX(i++, j++));
	printf("i=%d\tj=%d\n",i,j);

	return 0;
}
```

但是在Visual Studio 2019 中却报语法错误

![error](/img/linux_c_5/define_error.png)




## 3. 动手写代码
---
无

## 4. 课堂笔记
---

2. 常量与变量

常量：在程序执行过程中值不会发生变化的量

常量分类：
- 整型常量： 1， 33， 1234
- 实型常量： 3.134， 23.23，9.0
- 字符常量：由单引号引起来的单个的字符或转移字符（'a', '\n', '\1'，'\012', '\x7f'）
- 字符串常量：有双引号引起来的一个或多个字符组成的序列，如 "af3es", ""
- 标识常量: #define (只做替换，不会进行类型检查),处理在程序的预处理阶段，占用编译时间，一改全改




## 5. 参考资料
---