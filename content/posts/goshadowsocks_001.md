+++

author = "FanHe"
title = "Go-Shadowsocks源码解析0"
date = 2022-09-20T20:27:25+08:00
description = ""
categories = [
 "GoShadowsocks2"
]
draft = false

+++


# 

# 0. 缘起

说起学习golang有一段经历简单聊一下，我平常有听podcast的习惯，之前一直对科技类podcast比较感兴趣，偶然间听到了由吴涛和Rio主持的KernelPanic的节目，之后一直跟着听过来，感觉节目非常的不错，自己作为一个从事IT相关的从业者也是从其他专业转过来的，在节目中了解到了非常多的工具、开发方法开阔了自己的视野。

在节目中Rio经常会提到的一门语言就是golang，其实我自己很早就比较好奇。但是由于自己的开发工作大部分都是使用C++，并且平常的工作之余也有其他事务缠身，因此一直放下来没去了解。最近抽了些时间对golang整体上学了一遍，感觉大体上的语法和简单的使用是会了，不过缺少项目的历练。之后发现Rio自己有维护GoShadowSocks2的项目，加上我平常由于查阅资料等的需要，因此也接触了类似的工具。一直比较好奇这些底层是如何实现的，因此借此作为一个练手项目自己从头开始把整个项目走一遍，同时记录学习过程中的一些经历，如果有哪位朋友误打误撞访问到这个站点并且也对此感兴趣，并且碰巧也是一位小白的话，我希望自己的这点感悟对你有所帮助

话说我自己使用vim编辑器也是受了二位的布道，O(∩_∩)O哈哈~

# 1. 概述

这是一系列的Blog，从一个golang的初学者一步步跟着goshadowsocks2(https://github.com/shadowsocks/go-shadowsocks2) 这个项目来剖析它的源码，期间涉及到的标准库、其他第三方的知识，都会一一详细的给出解释，算是自己记录学些golang的历程
