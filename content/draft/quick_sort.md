+++
author = "FanHe"
title = "排序算法之快速排序(QuickSort)"
date = 2021-06-02T09:02:42+08:00
description = ""
categories = [
    ""
]
draft = false
+++


# 1. 简介

快速排序是一种使用非常广泛并且高效的排序算法，它和归并排序（Merge Sort）类似，一般也是采用递归的方式来实现，它们都是分治算法(Divide-and-Conquer)算法的一些典型的应用

# 2. 算法思路

快速排序的算法思路如下：

对于一个无序的数组，我们要将其按从小到大的顺序排列，过程如下：

1. 如果数组元素是1，那么直接返回
2. 从数组中挑选一个元素，一般称之为Pivot，以它为界，把小的元素都放在左侧，大的元素放在Pivot值的右侧
3. 这样数组被分成了三部分：小于pivot值  等于pivot值 大于pivot值，我们分别用集合S1，S2，S3代替，很显然合并S1-S2-S3就是最终排好序的结果
4. 递归的对S1和S3进行快速排序操作


# 3. 算法实现

## 3.1 我个人的实现与分析

快速排序的算法思想比较简单，在看过一篇文章就理解了，但是在实作层面，发现有非常多的细节需要处理，稍不留意就会导致数组越界或者是死循环的情况，本文再回头对这个算法进行全面的手术刀式的分析，以期望对算法的每一个细节完全理解透彻。

原始的代码：

```
template<typename T>
void QuickSort_Impl(T arr[], int beg, int end)
{
	if (beg >= end)
		return;

	T pivot = arr[beg];
	int i = beg;
	int j = end;

	while (i <= j)
	i
		while (arr[i] <= pivot)
		{
			i++;
		}

		while (arr[j] >= pivot)
		{
			j--;
		}
		
		std::swap(arr[i], arr[j]);
	}

	QuickSort_Impl(arr, beg, i - 1);
	QuickSort_Impl(arr, i + 1, end);
}

template<typename T>
void quickSort(T arr[], int n)
{
	QuickSort_Impl(arr, 0, n - 1);
}
```

先了解一下写算法时候的思路： 每次选取待排序数组的第一个元素值作为pivot，尝试将待排序数组中的元素以pivot值为界分为左右两侧，左侧的元素都小于或等于pivot，右侧的元素都大于或者等于pivot，在进行处理的时候，选取了下标i和j分别指向数组的开头与结尾的索引值。i从前往后递增（如果arr[i] <= pivot），j从后往前递减（如果arr[j] >= pivot），如果i卡在一个大于pivot的元素处，同时j卡在一个小于pivot的元素处，那么我们交换arr[i]和arr[j]的值，这样可以把i位置的元素变成小于pivot的，j位置的元素变成大于pivot的，这样i和j又可以继续朝指定方向前进，直到i超过了j或者i和j在同一个位置，说明已经遍历完了整个数组，把数组的元素分好类了。因为i位置的元素是pivot，那么pivot是排好序的位置，接着递归的对pivot左侧和pivot右侧的子数组再进行排序，直到整个排序结束，就完成了整个算法。


- 问题1 数组的越界问题

```
		while (arr[i] <= pivot)
		{
			i++;
		}

		while (arr[j] >= pivot)
		{
			j--;
		}
```
这两个循环有可能会造成数组越界，外层的 while(i<=j) 是在一开始进行的判断，无法约束i和j一直递增或者递减的情况。假设整个数组arr[0]是最小值，那么在第一次选pivot的时候是arr[0]，j这个变量就会越界，因为所有的元素都比pivot大（或者等），于是j就直接变成-1了，很显然是错误的。所以我们要做的第一处修改就是约束 i 和 j 变量的取值

我们直到如果i和j相等实际上就说明已经通过i和j遍历完了整个数组了，于是判断条件是 `i < j`

于是代码变为：

```
	while (i <= j)
	{
		while (i < j && arr[i] <= pivot)
		{
			i++;
		}

		while (i < j && arr[j] >= pivot)
		{
			j--;
		}

		std::swap(arr[i], arr[j]);
	}
```
这样还是有一个问题，当i和j相等之后，两个内层循环进不去，一个swap语句在交换两个相等的数，并且外层的while循环一直是true，这样就进入死循环了，于是while进入的条件应该不能包括i和j相等的情况，所以外层的while循环也应该修改为while(i < j)

```
	while (i < j)
	{
		while (i < j && arr[i] <= pivot)
		{
			i++;
		}

		while (i < j && arr[j] >= pivot)
		{
			j--;
		}

		std::swap(arr[i], arr[j]);
	}
```

- 问题2  数组pivot分界的问题

在调试之后我发现数组根本每一次都没有在pivot的位置进行分界，也就是说我们期望数组在每一次找到pivot位置的时候，分成左右两侧，左侧小于pivot，右侧大约pivot（也就是数组的值如果小于pivot，那么它们的索引号一定也小于pivot），由于我们判断中arr[i] <= pivot的缘故，导致我们会越过pivot，也就是说最后循环退出的时候，i的位置上元素根本就不是pivot值，这样就起不到分块的作用了，于是代码修改为： arr[i] < pivot，把相等的条件去掉，于是代码修改为：
	
```
while(i < j && arr[i] < pivot)
```

- 问题3 递归调用的范围问题

当我们把条件修改成为 arr[i] < pivot 之后，伴随而来又有一个问题，如果数组的第一个元素是最小的，在第一次运行QuickSort_Impl时，我们的i就不会移动，导致退出while循环的时候i的取值是0，这样就没有办法调用 `QuickSort_Impl(arr, beg, i - 1);` 因为会产生一个-1的索引

另外如果数组的第一个元素是最大的，会导致j不会移动，当进行一次swap之后，最大的元素被移动到了数组最后的位置，这时候i会一直向后移动，直到i移动到j的位置（i==j）并且是移动到数组的最后一个位置，此时 `while (i < j && arr[j] >= pivot)` 这个循环不会被执行，最后进行一次swap(pivot,pivot)的操作，退出循环时候 i的取值是最大的（数组的最大索引号），这样就造成一个问题，最后一个分片的起始位置就越界了 `QuickSort_Impl(arr, i + 1, end);` 因为产生一个数组元素个数的索引（数组最大索引应该是size-1)

我们直到出现这两种情况的时候，数组其实只有一个元素，也就是说就是排好序的了，于是我们在递归中加上一个判断

```
	if (i > beg)
		QuickSort_Impl(arr, beg, i - 1);

	if (i < end)
		QuickSort_Impl(arr, i + 1, end);
```

- 问题4 pivot值的处理

尽管已经按照上面的方式改动了大量的代码，但还是有问题，比如下面这个测试用例： [6,8,7,5]

![quicksort-e](/img/quicksort_error2.png)

这种情况就比较麻烦了，也就是说这个pivot值要十分小心的进行处理，可以想到的处理方式如下：

1. 选定最开始的元素作为pivot之后，执行一次swap，把最开始的元素和结尾的元素进行一次交换
2. j的位置从倒数第二个元素开始（因为最后一个元素是交换过去的pivot）
3. 最终确定好pivot应有的位置之后，把i所在的位置和数组最末尾的元素交换回来

按照这种想法改进如下：

```
template<typename T>
void QuickSort_Impl(T*& arr, int beg, int end)
{
	if (beg >= end)
		return;

	T pivot = arr[beg];

	std::swap(arr[beg], arr[end]);
	int i = beg;
	int j = end - 1;
	
	while (i < j)
	{
		while (i < j && arr[i] < pivot)
		{
			i++;
		}

		while (i < j && arr[j] >= pivot)
		{
			j--;
		}

		std::swap(arr[i], arr[j]);
	}

	std::swap(arr[i], arr[end]);


	if (i > beg)
		QuickSort_Impl(arr, beg, i - 1);

	if (i < end)
		QuickSort_Impl(arr, i + 1, end);
}
```
经过测试发现在只有两个元素时，这种情况下会出错。比如数组是 [2,1]时，排序结果是 [2,1]，为了处理这种情况，只需要对最后 pivot和arr[i]做一个判断即可，加上条件 if (arr[i] > arr[end])

最终的快速排序的完整代码如下：

```
#include <algorithm>

template<typename T>
void QuickSort_Impl(T*& arr, int beg, int end)
{
	if (beg >= end)
		return;

	T pivot = arr[beg];

	std::swap(arr[beg], arr[end]);
	int i = beg;
	int j = end - 1;
	
	while (i < j)
	{
		while (i < j && arr[i] < pivot)
		{
			i++;
		}

		while (i < j && arr[j] >= pivot)
		{
			j--;
		}

		std::swap(arr[i], arr[j]);
	}

	if (arr[i] > arr[end])
		std::swap(arr[i], arr[end]);


	if (i > beg)
		QuickSort_Impl(arr, beg, i - 1);

	if (i < end)
		QuickSort_Impl(arr, i + 1, end);
}

template<typename T>
void quickSort(T arr[], int n)
{
	QuickSort_Impl(arr, 0, n - 1);
}
```

这个快速排序从我看完算法描述到写出完整代码调试边界情况，整整花费了一个下午的时间，这也充分说明了理解和最终实作落到代码层面是两个完全不同的事情，平常还是需要多写。数据结构不能仅仅停留在理论上理解了，一定要落在纸上，真正的写一个可以运行的实现。



## 3.2 网络上另一种实现思路(参考资料2)



# 4. 更好的快速排序实现(wiss参考资料1)




# 参考资料

1. 书籍：Data Structures and Algorithm Analysis in C++ (Fourth Edition), by Mark Allen Weiss 7.7节-快速排序
[2. 2.8.1 QuickSort Algorithm](https://www.youtube.com/watch?v=7h1s2SojIRw) 
[3.Quicksort](https://en.wikipedia.org/wiki/Quicksort)