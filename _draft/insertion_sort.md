+++
author = "FanHe"
title = "排序算法之插入排序"
date = "2021-05-31T15:52:20+08:00"
description = ""
categories = [
    "排序算法"
]
+++

# 1. 概述

插入排序是一种相对简单的排序算法，插入排序算法在处理过程中每次处理一个待插入的元素，将它和已经排好序的子序列进行合并成新的已排好序的部分，逐渐增长直到整个数组排序完成。

插入排序的主要优点包括：
1. 实现相对简单（相比较快速排序，堆排序，归并排序来说它的代码相对较少）
2. 对元素不太多的序列有比较好的性能
3. 相比较其他 $ O(n^2) $ 的算法来说更加高效
4. 算法是排序稳定的（值相等的元素位置顺序保持不变）
5. In-place算法，只需要固定的内存空间占用（基本上只需要原先数组的空间即可）
6. Online算法，可以来一个元素处理一个


# 2. 实现方式

插入排序的算法虽然简单，但是还是有一些细节需要注意，以下列举一些实现

## 2.1 我最初的实现

以下是我自己在阅读插入排序描述之后给出的代码：(C++代码)

- 版本1

```cplusplus
template<typename T>
void insertionSort(T array[], int n)
{
	for (int i = 1; i < n; i++)
	{
		int insertPos = 0;
		T tmp = array[i];

		for (int j = 0; j < i; j++)
		{
			if (array[j] < tmp)
			{
				insertPos++;
			}
		}
			
		for (int k = i; k > insertPos; --k)
		{
			array[k] = array[k - 1];
		}

		array[insertPos] = tmp;
	}
}
```

- 版本2

```cplusplus
template<typename T>
void insertionSort(T array[], int n)
{
	for (int i = 1; i < n; i++)
	{
		int insertPos = 0;
		T tmp = array[i];

		for (int j = 0; j <= i; j++)
		{
			if (array[j] < tmp)
			{
				insertPos++;
			}
		}
			
		for (int k = i; k > insertPos; --k)
		{
			array[k] = array[k - 1];
		}

		array[insertPos] = tmp;
	}
}
```

以上给出的两个版本的代码，我自己通过测试程序进行测试，测试程序如下：

```cplusplus
int main()
{
	int arr[] = { 1,9,2,6,4,3,8,7,5 };

	insertionSort(arr, sizeof(arr) / sizeof(arr[0]));

	for (auto i : arr)
	{
		std::cout << i << std::endl;
	}
}
```

分析我个人写的代码，发现有一些需要改进的点：

1. 算法2实际上是有问题的
使用测试程序是看不出来的，最后给出的结果都是1-9的顺序输出，但是算法2会造成算法的不稳定（也就是会把相等的元素的先后顺序改变）

2. 算法1是正确的，但是先找位置，再移动元素可以改进一下，可以从排好序的子序列的最后位置开始往前找，而不是像我写的代码这样从前往后找。因为从后往前有一个好处是交换操作可以边找边做，而从前往后找，再找到位置之后也需要移动元素，何不一边找一边移动呢？其实这也是标准的插入排序的算法实现

## 2.2 常见的插入排序算法实现

以下是一种插入排序的算法伪码实现

``` 位置以0作为第一个位置
i ← 1
while i < length(A)
    j ← i
    while j > 0 and A[j-1] > A[j]
        swap A[j] and A[j-1]
        j ← j - 1
    end while
    i ← i + 1
end while
```
这个算法有几个细节需要注意：

1. while j > 0 and A[j-1] > A[j] 的And 操作必须是一个短路的and操作（类似于C/C++中的 &&，当第一个条件失效时，不会计算第二个条件）

2. 这个算法插入的时候，从插入点开始每次都进行了元素的交换

进行元素的交换有一个好处是完全不需要额外的空间排序，也就是说算法整个运行过程中仅仅需要算法原始元素的存储空间。不过过多的交换或许并不是很合适，可以通过记录我们当前即将要插入的元素，通过额外的一个元素的空间，来换取一直进行的交换操作。

以上伪代码的C++实现如下

```
#include <algorithm>

template<typename T>
void insertionSort(T arr[], int n)
{
	for (int i = 1; i < n; i++)
	{
		for (int j = i; j > 0 && arr[j - 1] > arr[j]; j--)
		{
			std::swap(arr[j - 1], arr[j]);
		}
	}
}
```

## 2.3 略微改进的插入排序算法

上面在2.2节中讨论到可以通过额外的一个临时变量存储当前待插入的元素，从而减少一直进行的交换操作，伪码如下：

```
i ← 1
while i < length(A)
    x ← A[i]
    j ← i - 1
    while j >= 0 and A[j] > x
        A[j+1] ← A[j]
        j ← j - 1
    end while
    A[j+1] ← x
    i ← i + 1
end while
```
注意算法的几个细节：

1. 使用临时变量保存了即将插入的元素A[i]的值，在内层循环中一直向后移动元素，直到找到A[i]归属的那个位置

2. 内层代码中赋值使用的是A[j+1] = A[j]，和2.2节中略有不同，这些是实现中边界条件的一些细节，在实际编码中需要格外注意（最好是手写一个简单的数组来模拟）

以上伪码的C++实现如下：

```
template<typename T>
void insertionSort(T arr[], int n)
{
	for (int i = 1; i < n; i++)
	{
		T tmp = std::move(arr[i]);

		int j = i-1;
		for (; j >= 0 && arr[j] > tmp; j--)
		{
			arr[j + 1] = std::move(arr[j]);
		}
		arr[j+1] = std::move(tmp);
	}
}
```
以上代码在设置 int j = i - 1，写起来特别别扭，建议还是用 int j = i的方式，赋值使用 arr[j] = arr[j-1]的方式，更加自然一点。

另外代码中使用了C++的移动函数，如果数组的元素是特别大的对象，那么这样处理减少了拷贝对象的过程


## 2.4 递归的实现

插入排序可以采用递归的方式来实现，插入排序采用递归的方式没有任何优势可言，不过可以考察对于递归和插入排序的理解，递归的思路如下：

1. 基本情形(Base): 如果数组长度为1，排序完成
2. 递归的处理最开始的n-1个元素
3. 插入最后一个元素到已经排好序的子数列中

实现代码如下：

```
template<typename T>
void insertionSort(T arr[], int n)
{
	if (n <= 1)
		return;
	
	insertionSort(arr, n - 1);

	T last = arr[n - 1];
	int j = n - 1;

	while (j > 0 && arr[j-1] > last)
	{
		arr[j] = arr[j-1];
		j--;
	}
	arr[j] = last;
}

```
也就是在已经排好序的数组中再新增一个元素


# 3. 算法的复杂度分析

最好情况：

当数组已经是排好序的情况下，内层的循环只需要进行一次，时间复杂度是O(n)

最坏的情况：

当数组完全是逆序的时候，整个循环需要依次比较 1+2+3+...+n-1次，于是时间复杂度是 $ O(n^2) $

平均情况：
平均的时间复杂度也是 $ O(n^2) $

# 4. 参考资料

[1.Insertion sort](https://en.wikipedia.org/wiki/Insertion_sort)
[2.Insertion Sort](https://www.geeksforgeeks.org/insertion-sort/)
[3.Recursive Insertion Sort]()
[4.Insertion Sort by swapping???](https://stackoverflow.com/questions/40051083/insertion-sort-by-swapping)
5.Introduction To Algorithms(chapter 2.1)