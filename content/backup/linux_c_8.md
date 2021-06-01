+++
author = "FanHe"
title = "LinuxC编程随笔（输入输出专题-8）"
date = 2021-04-29T12:05:43+08:00
description = ""
categories = [
    "LinuxC编程"
]
libraries = [
    "katex"
]
+++

## 1. 一句话总结

介绍LinuxC中的标准IO函数

---
C语言中的输入输出

## 2. 课程内容
---

### 2.1 printf函数

printf的语法格式如下：

```
int printf ( const char * format, ... );
```
这个原型中的format的内容包括以下几部分：
```
%[flags][width][.precision][length]specifier
```
- specifier是格式字符，包括 %d %s %c %f %g 等
- flags标识左右对齐，添加+(-)前缀，补0对齐等
- width：设置输出宽度，如果输出内容小于width，用空格补充，大于width，无作用
- .precision（注意是点+精度位数），浮点数输出小数点位数；字符串输出字符个数

### 2.2 输出中添加换行符对printf打印调试大法的影响

某些情况下在不方便调试时，printf打印调试信息大法是比较好用的一种除错方法，但是有一些细节需要注意，比如下面的程序

```
#include <stdio.h>
#include <stdlib.h>

int main()
{
	printf("[%s:%d] before while().", __func__, __LINE__);
	while(1);
	printf("[%s:%d] after while().", __func__, __LINE__);

	exit(0);
}
```
在运行过程中，程序会卡住什么也输出不了。原因在于printf是行缓冲模式，遇到换行符或者缓冲区满才会输出，而这里的缓冲区并未满，因此就会输出不了任何内容，导致调试无效

### 2.3 scanf函数在循环内需要小心处理

scanf在循环内时，如果用户给的输入不合法会造成一些问题，导致循环无法结束，因为此时输入的流已经被破坏了。因此为了避免这种情况，需要小心的检查scanf的函数返回值，确保程序确实匹配到用户成功的输入项，比如：

```
while(1)
{
    ret = scanf("%d", &i);
    if (ret != 1)   //判断用户输入真的是一个整数（scanf函数返回值是成功匹配的项数）
    {
        break;
    }
}
```

### 2.4 scanf输入中使用抑制符*

*可以只接受数据但是不给任何赋值，比如下面的代码

```
int i;
char ch;
scanf("%d", &i);
scanf("%c", &ch);
```
如果在输入时，用户先输入 24->回车->a，结果不正确，ch取到回车的值，而不是a

修改为：

```
int i;
char ch;
scanf("%d", &i);
scanf("%*c%c", &ch);
```
让回车字符被接收，但是直接丢弃，这样可以得到正确的i=24，ch='a'

另外一种处理方式是使用getchar()
```
int i;
char ch;
scanf("%d", &i);
getchar();
scanf("%c", &ch);
```

## 3. 动手写代码
---

### 习题一
一个水分子的质量大约为 $$3.0*10^{-23}$$克，1夸脱水大约有950克，编写程序要求从终端输入水的夸脱数，然后显示这么多夸脱水中包含有大约多少水分子？

```
#include <stdio.h>
#include <stdlib.h>

#define WEIGHT 3.0e-23
#define KQ 950

int main()
{
	float num;
	float sum;
 	
	printf("Please input water num:");
	scanf("%f",&num);
		
	sum = num * KQ / WEIGHT;

	printf("total kq is %e\n", sum);


	exit(0);
}
```
其他习题比较简单（略）


## 4. 课堂笔记
---

三、输入输出专题

input & output -> I/O (标准IO，文件IO)

1. 格式化输入/输出 函数： scanf，printf
    int printf(const char *format, ...);
    format: "% [修饰符] 格式字符"

    int scanf(const char *format, ...);
    format:抑制符*
    %s的使用是比较危险的，因为不知道存储空间大小
    scanf放在循环结构中要注意能否接收到正常有效的内容


2. 字符输入/输出 函数： getchar, putchar
3. 字符串输入/输出 函数： gets(!), puts
    gets:十分危险的函数，可以用fgets，getline（Linux GNU的dialect）来替代





## 5. 参考资料
---

[1. printf](https://www.cplusplus.com/reference/cstdio/printf/)