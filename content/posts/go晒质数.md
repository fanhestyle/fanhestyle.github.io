+++
author = "FanHe"
title = "Go筛质数"
date = 2022-11-03T16:39:20+08:00
description = ""
categories = [
 "Go"
]
draft = false

+++

# 1. 概述

在阅读Go编程语言开发时，发现文中提及一个筛质数的程序，初看感觉很难理解，经过一些分析之后，记录于此



整段代码如下：

```go
// send 2,3,4, ... to channel 'ch'
func Generate(ch chan<- int) {
	for i := 2; ; i++ {
		ch <- i //send 'i' to channel 'ch'
	}
}

// copy values from channel 'in' to channel 'out'
// remove those divisible by 'prime'
func Filter(in <-chan int, out chan<- int, prime int) {
	for {
		i := <-in //receive from 'in'
		if i%prime != 0 {
			out <- i //send 'i' to 'out'
		}
	}
}

func main() {
	ch := make(chan int)
	go Generate(ch)

	for i := 0; i < 10; i++ {
		prime := <-ch
		fmt.Print(prime, "\n")
		ch1 := make(chan int)
		go Filter(ch, ch1, prime)
		ch = ch1
	}
}

```

算法的思想如下：



假设有一个数`n`,已知小于`n`的所有质数的集合是`arr`,如果`arr`中的每个质数都不能够整除`n`，则`n`就是质数。这是因为任意一个合数都可以分解成质数的乘积。

例如，

因为 2 是质数，初始化 arr=[2]；

当n=3，3 % 2 != 0, 则 n=3是质数, arr=[2,3]；

当n=4，4 % 2 == 0, 不满足要求，则4不是质数， arr不变；

当n=5,，5 % 2 != 0, 5 % 3 != 0, 则 n=5是质数， arr=[2,3,5];

... 依此类推



# 2. 程序运行逻辑

1. 初始情况下启动Generate协程，这个协程会一直产生自然数，从2开始，依次是2,3,4,5...

2. Filter是一个死循环的函数，它用来将可以被自身整除的数过滤掉，转到下一个Filter去处理

3. 我们可以把Filter中的参数prime想象成是Filter这个”对象“的属性，这个属性是一个质数，Filter就是用来过滤可以被这个prime整除的数值

4. 我们需要搞清楚，在Filter函数中的in是由谁传入的，out传出之后又是由谁来取得的



看代码逻辑

```
for i := 0; i < 10; i++ {
    prime := <-ch           //[1]
    fmt.Print(prime, "\n") 
    ch1 := make(chan int)   //[2]
    go Filter(ch, ch1, prime) //[3]
    ch = ch1 //[4]
}
```

代码从[1]开始，首先从ch中取出的数是2，由于2是质数，因此会新建一个Filer过滤器协程，并且把prime=2传入进去，那么in是怎么传入的呢？第一个协程Filter的ch是由Generate传入的，它的值是 ch=3

之后再Prime=2的Fileter协程中，它会产生一个out参数（也就是ch1），这个ch1的第一个值一定是3，（因为3不能被2整除，于是会传出一个ch1=3的值），并且这个3一定是质数【原因我们上面提到的算法思想中已经描述过了】，于是3被赋值给prime，程序进行下一轮的循环，接着3被打印出来

然后我们再启动下一个Fileter，传入prime=3，也就是这个新的Filter协程是 prime=3的协程，之后这个协程传入的ch从何而来呢？我们发现这个新的ch是来自于上一个协程的ch1，也就是prime=3的Filter协程待处理的ch元素来自于上一个协程prime=2的Filter协程，由于第一个值已经被prime读取了，因此prime=3的协程等待的是下一个值

但是这下一个值又是从源头Generate过来的ch=4，这个值会首先经过prime=2的Filter过滤一下，然后再经过prime=3的Filter，由于4可以被2整除，于是再prime=2的Filter中就已经被剔除了4，因此Generate会产生下一个值5，在这段时间内，prime=3的Filter是出于阻塞状态的，它的协程由于channel获取不到值而处于阻塞状态。当5经过prime=2的Filter之后，会传入到prime=3的Filter，又会从prime=3的Filter被传出去到out，此时程序进入到下一轮的循环之中，于是会产生新的prime=5的Filter，以此类推...



我们发现整个程序运行过程中，产生了一个从源头Generate的数，经过每一个质数Filter协程的过滤，最终如果成功通过，那么又会产生一个新的已这个通过的值作为prime的Fitler，依次进行，知道程序循环终止



# 3. 总结

- `ch=ch1`的时候，原先的`ch`并没有消失，还被之前启动的`fliter`协程保存着。
- 巧妙使用通道的性质，将多个协程进行串联。
- 最开始以为是一个并行的筛选质数的代码，结果并不是，其实是**串行**执行





# 参考资料

1. [一个有趣的求解质数的golang代码 - 简书 (jianshu.com)](https://www.jianshu.com/p/b98b68987b20)

2. [golang并发素数筛-并发真的会快吗？ - 张小凯的博客 (jasonkayzk.github.io)](https://jasonkayzk.github.io/2020/06/25/golang%E5%B9%B6%E5%8F%91%E7%B4%A0%E6%95%B0%E7%AD%9B-%E5%B9%B6%E5%8F%91%E7%9C%9F%E7%9A%84%E4%BC%9A%E5%BF%AB%E5%90%97%EF%BC%9F/)
