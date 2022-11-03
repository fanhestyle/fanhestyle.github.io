+++
author = "FanHe"
title = "Go语言项目结构"
date = 2022-11-03T17:47:12+08:00
description = ""
categories = [
 "Go"
]
draft = false

+++

# 1. 概述

项目的组织结构是一个工程项目中在设计之初就需要决定的内容，在Go语言中有一些列关于工程代码组织的良好实践，本文列举一些相关的内容，方便在实际开发中使用

# 2. 最小标准布局

这个是一个经典的最小化的布局方式，如下图所示：

```textile
//Go项目根目录

- go.mod
- LICENSE
- xx.go
- yy.go
- ...
```

或者

```textile
- go.mod
- LICENSE
- package1
        - package1.go
- package2
        - package2.go
```

# 3. 构建可执行程序为目的的Go结构

  ![](/img/go-structure.png)

# 4. 以库为目的的Go项目结构

![](/img/go-structure-lib.png)
