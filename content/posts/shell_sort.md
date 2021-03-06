+++
author = "FanHe"
title = "排序算法之希尔排序(ShellSort)"
date = 2021-06-01T08:17:59+08:00
description = ""
categories = [
    ""
]
draft = false
+++

# 1. 简介

希尔排序是希尔(Donald Shell)于1959年提出的一种排序算法，它是插入排序的一种改进算法，在插入排序中，我们每一次比较的是待插入数和已排好序的子数组。这样在插入的时候每一次进行相邻间元素的一次交换，希尔排序则不一样，它每次排序以一个固定的间隔进行（这个间隔被称为gap），并且最后一次的gap一定是1（也就是最后一次的排序是一次普通的插入排序）

为什么要如此设计呢？我们考虑这样一个情形，比如下述的待排序数组

[5,3,7,4,8,9,6,2,1]

当我们插入排序的时候，在最后两个元素2和1进行排序的时候，我们会发现需要大量的移动操作才能把2和1放到它们应有的位置（也就是数组的开头），这样就会萌生一个想法，能不能尽快的使得2和1这样的数迅速的移动到前面呢？而不用每次一个一个数的比较前移。Shell排序就是这种想法，它用非1的间隔来进行比较，假设我们用4的间隔进行比较，在第一趟中元素4和2比较，发现2在后面，会把2移到元素4的位置，这样元素2一下子就能往前走好多步。并且当我们缩小间隔gap的时候，之前比较大间隔拍好的序列不会白费（也就是之后减少间隔的排序还是会保留之前排序的成功，相当于前人做的努力没有白费，成果一直保留在那儿），直到gap是1的最后一次排序操作，最后一次的gap为1的排序一定能保证整个数组排好序（因为它就是一次简单的插入排序嘛），而可以预期最后一次的排序需要移动的元素一定不会太多。

另外参考资料2中给出了一个解释（也是数据结构与算法分析这本书提及的），我们把数组中逆序的数对找出来，比如上面我们举例的数组中逆序的数对包括（5，3）（5，4）（5，2）（5，1）（3，2）（3，1）... 这些数对期望的对数是 $$ \frac {n(n-1)} 4 $$ ，如果我们采取的是逐个比较相邻两个数，通过相邻两个数来决定调整相邻两个数之间的位置，这样我们一次操作最多能纠正一对逆序的数对，也就是说像插入排序、冒泡排序、选择排序这种每次只通过比较相邻的元素而进行交换的排序算法，它们的时间复杂度一定是 $$ O(n^2) $$，那么怎样提高这个排序效率呢？

关键点就是我们不能一直比较相邻的数，而应该比较相距较远的数，这样有可能在一次交换之后能够期望达到纠正超过一个逆序对的效果，可是有人又会问：如果一次相聚较远的数，那会不会导致交换的数对中增加了额外的逆序数呢？下面我们来证明通过交换间隔较远的数对不会造成逆序数的增加。

假设有位置i和j，i < j，但是a[i] > a[j]，我们假设数组排好序之后是从小到大排列，很显然（i，j）是一个逆序对，我们想知道交换i，j会不会造成逆序对的增加，因为i和j的交换会影响到i和j之间的数（i和j之前和之后的数和i、j的相对关系不变，因此不会影响），考虑到i和j之间的任意一个数，假设是k（i < k < j)，我们讨论下列几种情况：

1. a[k] > a[i] 
2. a[k] < a[j]
3. a[i] < a[k] < a[j]

也就是说a[k]的值小于最小的，位于最小和最大之间以及大于最大的，这样就完全覆盖了a[k]的取值情况，这样我们来分析它们交换之后的效果：（我们用1，2，3来标识a[i] a[k] a[j]这样的值） 

- 情况1：初始是 2 3 1 ，交换i和j之后变为 1 3 2，逆序对从2变为1
- 情况2：初始是 3 1 2 ，交换i和j之后变为 2 1 3，逆序对从2变为1
- 情况3：初始是 3 2 1，交换i和j之后变为 1 2 3，逆序对从3变为0

于是我们可以说当使用间隔较大的数对交换时，一定是会减少逆序数的，有可能减少一对，也有可能减少多对，这样就至少比每次减少一对的算法要好，虽然我们不知道好的上限是好到多少，但是至少不比插入、冒泡、选择这些排序效果要差。

事实上希尔排序的时间复杂度的证明相当的复杂，需要使用数论等非常高级的数学理论来证明，我个人还没有这个能力提供这些证明，看懂也暂时不可能，记住这个思想和为什么比普通的二次排序要好对于非理论研究的开发者来说要应该是足矣。


# 2. 实现

希尔排序的实现不是特别的复杂，以下的实现包括我个人初次学习写下的一些代码

## 2.1 我个人的代码

```
template<typename T>
void shellSort(T arr[], int n)
{
	for (int gap = n / 2; gap >= 1; gap /= 2)
	{
		for (int i = 0; i < n; i++)
		{
			for (int j = i+gap; j < n; j += gap)
			{
				int k = j;
				T tmp = arr[k];
				for (; k >= gap && arr[k] < arr[k - gap]; k -= gap)
				{
					arr[k] = arr[k - gap];
				}
				arr[k] = tmp;
			}
		}
	}
}
```

我给出的程序也是可以正确得到排序结果的，但是通过和其他标准Shell写法的程序相对比，发现了我给出来的程序会做一些额外无用的操作，从代码一眼就能看出来我写的代码有多层的循环，事实上我在阅读Shell排序的说明之后，第一时间写出来的就是上面的代码，我写的代码中第3层的循环 `for(int j=i+gap; j<n; j+=gap)` 想法是把所有间隔为gap的所有元素都找出来，然后进行一次常规的插入排序，从思路上来说是没错，但是确有点冗余，比如下面的数列

```
[3,7,2,8,1,9,6,5,4]
```
当遍历到1时，代码会找出来子序列 [3,1,4]，然后用插入排序把所有的数排好，下次遍历到1时，还是会找出来[1,4]又再排一次，这样很明显是有重复操作的。

问题的关键在于：shell排序是会每次比较子序列，但却不是向我这样生硬的把所有子序列都找出来排一遍。而是逐个元素遍历，找到一个元素之后和之前的相隔gap的元素进行比较，在一次最外层循环之后，每个内层的循环把新的数添加到以前遍历的都是相隔gap位置的子序列中，最终完成排序。


## 2.1 Shell给出的实现

希尔最早给出的实现是选取gap的值为 $$ \lfloor \frac {N} {2^k} \rfloor $$ ,实现的伪码如下：

```
# Start with the largest gap and work down to a gap of 1
foreach (gap in gaps)
{
    # Do a gapped insertion sort for this gap size.
    # The first gap elements a[0..gap-1] are already in gapped order
    # keep adding one more element until the entire array is gap sorted
    for (i = gap; i < n; i += 1)
    {
        # add a[i] to the elements that have been gap sorted
        # save a[i] in temp and make a hole at position i
        temp = a[i]
        # shift earlier gap-sorted elements up until the correct location for a[i] is found
        for (j = i; j >= gap and a[j - gap] > temp; j -= gap)
        {
            a[j] = a[j - gap]
        }
        # put temp (the original a[i]) in its correct location
        a[j] = temp
    }
}

```

C++代码实现如下：

```
template<typename T>
void shellSort(T arr[], int n)
{
	for (int gap = n / 2; gap >= 1; gap /= 2)
	{
		for (int i = gap; i < n; i++)
		{
			T tmp = arr[i];
			int j = i;
			for (; j >= gap && tmp < arr[j - gap]; j -= gap)
			{
				arr[j] = arr[j - gap];
			}
			arr[j] = tmp;
		}
	}
}
```

# 3. 时间复杂度分析

Shell排序的时间复杂度非常的繁琐，需要大量的高等数学的知识，我暂时还没能力读懂这些证明（毕竟不是数学相关专业，也对这些内容不感兴趣），因此贴一张别人论证的结论在此

![shellsort_complicity](/img/shellsort_complexity.png)


图中也给出了一些相对表现较好的gap的取值（如果gap是递增方式的，那么我们在编码时只需要找到一个比数组长度小的最大的数，逆序直到1，作为我们的gap序列即可）


# 4. 参考资料

[1. WiKi:Shellsort](https://en.wikipedia.org/wiki/Shellsort)

[2.希尔排序为什么会那么牛那么快，能够证明吗？](https://www.zhihu.com/question/24637339)

3.Mark Allen Weiss:Data Structures and Algorithm Analysis in C++ (Fourth Edition) 7.4 ShellSort
