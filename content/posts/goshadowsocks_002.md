---
author = "FanHe"
title = "Go-Shadowsocks源码解析1"
date = 2022-09-20T20:42:20+08:00
description = ""
categories = [
 "GoShadowsocks2"
]
draft = false

---

# 1. 简介

```
git提交版本号 443dd019075b8367fd6db8a07d2967ecacbb855a

Initial commit

解析文件：main.go
```

本文件的主要作用是应用程序的入口地址main函数，代码主要是解析用户传入的命令行参数，基于这些参数来构建客户端/服务端

本课主要内容：

- golang标准库 flag

- golang标准库 time

- golang标准库 os/signal

- golang标准库 encoding/hex

- golang标准库strings

# 2. 标准库之flag

标准库flag是用来解析命令行参数的，这个库对于绝大多数不那么复杂的项目来说足矣，如果需要更加强大的命令行解析库，可以使用以下这些golang的三方库

1. Cobra

2. Kingpin

3. go-flags

等等，本文仅仅论述标准库中flag的用法

## 2.1 flag命令行语法

golang标准flag库支持的命令行比较简单，支持以下几种：

```textile
-flag
--flag   // double dashes are also permitted
-flag=x
-flag x  // non-boolean flags only
```

也就是说golang支持的 -和--（单划线和双划线）的一样的效果，这一点可能和Linux中一般短命令和长命令不同有所区别，另外golang的flag也不支持非中划线的命令（比如 iproute2中类似子命令等方式的命令 ：如ip route）

另外对于命令的参数：

- 布尔类型支持：1, 0, t, f, T, F, true, false, TRUE, FALSE, True, False（也就是指定bool类型的时候用 -flag=1 -flag=true等都是同样的效果）

- 日期类型支持所有time.ParseDuration可以解析的类型

## 2.2 flag库的使用

一般来说我们可以有两种捕获变量设值的方式：

方式一：返回类型的指针（比如示例中返回int*）

```go
    var nFlag = flag.Int("n", 1234, "help message for flag n")
```

方式二：直接传入指针

```go
    var flagvar int
    flag.IntVar(&flagvar, "n", 1234, "help message for flag n")
```

建议用方式二去处理，相对来说比较简单，并且我们可以把结构体内的变量传过去，方便我们将多个命令行参数组织成一个结构体去管理

## 2.3 flag库的实例

```go
var config struct {
    A string
    B int
    C float64
    D time.Duration
}

func main() {

    flag.StringVar(&config.A, "a", "hello", "param A using -a and default value hello")
    flag.IntVar(&config.B, "b", 10, "param B using -b and default value 10")
    flag.Float64Var(&config.C, "c", 3.5, "param C using -c and default value 3.5")
    flag.DurationVar(&config.D, "d", time.Duration(time.Second*5), "param A using -d and default value 30s")

    //我们可以在用户给出错误参数的时候使用flag.Usage打印出用法提示
    //flag.Usage()

    flag.Parse()

    fmt.Println("Your Input:")
    fmt.Printf("A = %v\n", config.A)
    fmt.Printf("B = %v\n", config.B)
    fmt.Printf("C = %v\n", config.C)
    fmt.Printf("D = %v\n", config.D)
}
```

# 3. 标准库之time

time库是用来处理时间的库，时间包含有“时钟时间和单调时间”两类，在golang中当需要打印输出时间的时候，会使用时钟时间，但是当需要做计时时，使用的是单调时间。在golang中的time里面一般会包含有两种计时类型（时钟时间+单调时间），但是某些time包中的函数调用会导致这两种类型中的单调时钟被隐式的丢弃，这一点需要注意。time.Time结构体的定义如下：

```go
type Time struct {
    wall uint64    //挂钟
    ext  int64     //单调时钟
    loc *Location  //时区
}
```

在内部实现中，wall的64位的最高位是标识是否存在单调时钟，如果最高bit位是1，那么表示存在单调时钟，否则表示不存在单调时钟（当不存在单调时钟时，字段ext的含义会发生变化，不再表示单调时钟，而是作为挂钟从公元0年开始的秒数）

## 3.1 时间库的使用

- 1. 获取当前时间
  
  ```go
  t := time.Now()
  ```
  
  使用Now()函数获取的时间包含有挂钟+单调时钟2部分，但是使用time.Unix、time.Parse、time.Date获取的时钟是没有单调时钟的

- 2. 获取特定时区的当前时间
  
  方法一：设置TZ环境变量为某个时区比如（America/New_York）之后再运行 time.Now
  
  就可以获得当前设置了环境变量TZ是时区时间
  
  方法二：
  
  ```go
  func main() {
  
      t := time.Now()
      fmt.Println(t)
  
      loc, err := time.LoadLocation("America/New_York")
      if err != nil {
          fmt.Println("load time location failed:", err)
          return
      }
  
      t1 := t.In(loc)
      fmt.Println(t1)
  }
  ```
  
  可以使用这种方式完成任意时区时间的转换

- 3. 时间的比较
  
  由于时间的 == 和 != 会比较结构体成员，因此可能会出现两个地球上相同的时间，却由于处于不同的时区导致结果不相等的情况（这有违于人们的直觉），因此时间不适合作为map的键值存在
  
  在time包中提供了比较两个时间的方法 time.Equal 以及 time.Before  time.After这几个方法，比较逻辑如下：
  
  （1）如果两个时间都包含单调时间，那么直接比较单调时间，返回
  
  （2）如果不都包含单调时间，那么比较挂钟的秒和纳秒的部分
  
  另外计算时间间隔的函数Sub也是同样的逻辑

- 4. 定时器的使用
  
  time包中最有价值的内容是提供了定时器，golang中提供2种定时器：
  
  （1）timer：只触发一次
  
  （2）ticker：周期性触发
  
  定时器创建有3种方式：
  
  （1）使用AfterFunc
  
  ```go
  func create_timer_by_afterfunc() {
      _ = time.AfterFunc(1*time.Second, func() {
          fmt.Println("timer created by after func fired")
      })
  }
  ```
  
  （2）使用NewTimer
  
  ```go
  func create_timer_by_newtimer() {
      timer := time.NewTimer(2 * time.Second)
      select {
      case <-timer.C:
          fmt.Println("timer created by newtimer fired")
      }
  }
  ```
  
  （3）使用After
  
  ```go
  func create_timer_by_after() {
      select {
      case <-time.After(2 * time.Second):
          fmt.Println("timer created by after fired")
      }
  }
  ```

# 4. 标准库之Signal

## 4.1 信号分类

- 无法捕获的信号 SIGKILL和SIGSTOP

- 同步信号：SIGBUS, SIGFPE, SIGSEGV，golang处理方式runtime panic

- 异步信号：
  
  - 丢失控制终端：SIGHUP
  
  - SIGINT：用户在控制终端按下 Ctrl+C
  
  - SIGQUIT： 用户按下 Ctrl + \ （反斜杠）

## 4.2 go信号默认行为

    一般遇到的大部分信号都是转换为run-time panic，进而导致程序直接退出。我们为什么要处理信号呢？

原因大体是因为我们大部分时间用go都是编写后台程序，后台程序一般来说是Daemon程序，因此有许多通信都是通过系统发送过来的信号来完成的，比如我们的后台程序在捕获到系统给的信号之后（比如用户键入Ctrl+C等操作），如果我们使用系统默认对于信号的处理，那么很可能导致我们程序正常结束需要的一些清理工作没能完成，从而造成数据的不一致等问题。

因此在实际生产环境中，不要忽略对于系统信号的处理，采用捕捉信号后退出的方式执行自定义的收尾工作

## 4.3 Go的信号包 os/signal

（1）Go会将异步信号捕获之后，将信号写入到channel，然后我们应用程序就可以拿到这个channel对信号进行处理

（2）Go会将同步信号捕获之后，触发panic，然后我们应用程序决定如何处理这个panic

一般来说同步信号是非常严重的硬件/总线等错误，go就无法简单的通过channel传递，而是直接panic，如果用户层没有recovery的代码，那么GO程序默认就会异常退出

我们尝试通过一个实例来展示同步、异步、无法捕获信号的运行机制

```go
func catchAsyncSignal(c chan os.Signal) {
    for {
        s := <-c
        fmt.Println("收到异步信号", s)
    }
}

func triggerSyncSignal() {
    time.Sleep(3 * time.Second)

    defer func() {
        if e := recover(); e != nil {
            fmt.Println("恢复panic: ", e)
            return
        }
    }()

    var a, b = 1, 0
    fmt.Println(a / b)
}

func main() {
    var wg sync.WaitGroup

    c := make(chan os.Signal, 1)

    signal.Notify(c, syscall.SIGFPE, syscall.SIGINT, syscall.SIGKILL)

    wg.Add(2)

    go func() {
        catchAsyncSignal(c)
        wg.Done()
    }()

    go func() {
        triggerSyncSignal()
        wg.Done()
    }()

    wg.Wait()
}
```

如果我们的应用程序在收到信号之后，没来得及处理，那么是否可以之后接收到所有的信号呢？（也就是问：信号是否会排队）

事实上我们由于使用了channel，如果channel是没有缓冲的，那么信号就只能够发送一次，否则信号回去填满缓冲区，之后我们从channel中也可以读到所有写入到缓冲区的信号

## 4.4 使用信号让程序优雅退出

主要是使用Notify来捕获我们最常见的几个信号 

SIGINT

SIGTERM

SIGQUIT

SIGHUP

```go
func main() {

    quit := make(chan os.Signal, 1)

    signal.Notify(quit, syscall.SIGINT,
        syscall.SIGTERM, syscall.SIGQUIT, syscall.SIGHUP)

    <-quit
}
```

这里仅仅给出一个例子，我们可以基于这个quit的channel，通过读取它来在我们自定义的函数内做更多的工作

# 5. 标准库之encoding/hex

encoding/hex是go提供的用来进行十六进制编码/解码的工具包，首先我们了解一下十六进制如何进行编码：

比如以下字符串： 1fg*

（1）字符串用它的ASCII表示，因为每一个字符都是8位，必定在0-255之间，于是肯定可以写成两个16进制的数，比如 1在ASCII中的值是49，转换成十六进制就是31，f是ASCII中的102，转换成十六进制就是66，因此以此类推，都可以转换

```go
func main() {
    someString := "1fg*"                          // input:  [1 f g *]
    str := hex.EncodeToString([]byte(someString)) // 内部实现:AsciiDecimalToHexadecimal
    fmt.Println(str)                              // output: 3166672a
}
```

反过来，我们可以反向的将一段文本转换成字符串

```
func main() {
    someString := "0a"    // input: 0a
    data, _ := hex.DecodeString(someString)// 内部核心实现核心代码AsciiHexadecimalToDecimal
    fmt.Println(data)    // output: [10]
}
```

- 小结

我们可以看出原始的字符串由于可打印的字符，其实它的取值范围更小（小于127），转码成十六进制之后里面的字符只有0-9+a-f这些数字和字母的组合

反向的解码之后的字符串中就是包含有数字、字母、符号等可被打印的字符

# 6. string包

由于string包就是字符串的包，这一个包在各种语言中几乎都有，并没有什么特别之处，因此不再这里详细论述

至此本文件中涉及的所有golang依赖的知识点都已经讲解完成，后续我们看详细的代码！

# 7. 整体分析

基础知识看完之后，我们分析源码main.go中的内容

整个main文件用来解析命令行参数，并根据不同的参数有不同的调用，整个框架如下图所示：

<img src="/img/goshadow/overview.png" title="" alt="架构图" data-align="center">

在程序的最后一段代码是监听用户按下的Ctrl+C等信号，方便我们结束程序

```go
    sigCh := make(chan os.Signal, 1)
    signal.Notify(sigCh, syscall.SIGINT, syscall.SIGTERM)
    <-sigCh
```

# 参考资料

1. # [golang的hex包](https://daysleep666.github.io/blog/%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/2018/11/12/hex.html)
