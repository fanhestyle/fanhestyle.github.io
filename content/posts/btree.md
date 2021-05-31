+++
author = "FanHe"
title = "B树(B-tree)"
date = "2021-05-11T17:06:15+08:00"
description = ""
categories = [
    "数据结构与算法"
]
+++


## 1. 简介

B树有两种分类的方式：

（1）按度来定义（degree）

这种定义方法在算法导论一书中提及的，
一棵度为t的B树：
定义为：非根内节点的最少孩子数是t，并且强制非根内节点的最大孩子数是2t


（2）按阶来定义（order）
这种定义方法是在The Art of Computer Programming 一书中定义的，
一棵m阶的B树：
定义为：非根内节点的最大孩子数量是m，非根内节点的最小孩子数量是 m/2 向上取整


这两种方式定义下的最简单B数就有所差异了，按度定义的话最小的B树是2-3-4树，按阶的方式定义最小的B树是2-3树





## 参考资料

[1. B-tree](https://en.wikipedia.org/wiki/B-tree#cite_note-FOOTNOTEKnuth1998483-9)

[2.stackoverflow: What is the difference btw “Order” and “Degree” in terms of Tree data structure](https://stackoverflow.com/questions/28846377/what-is-the-difference-btw-order-and-degree-in-terms-of-tree-data-structure/45826413#45826413)