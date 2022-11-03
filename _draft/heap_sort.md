+++
author = "FanHe"
title = "排序算法之堆排序"
date = 2021-06-01T10:22:41+08:00
description = ""
categories = [
    ""
]
draft = false
+++

# 1. 简介

堆排序使用到了堆（优先队列）的性质，假设一个堆是小顶堆，那么最大的元素在堆顶的位置，堆排序的步骤包括两个：

1. 创建堆结构
2. 依次移除堆顶的元素，移除的元素存放在数组中

从上面的描述我们可以大概的直到，堆排序需要一个额外的数组来存储排好序的队列，接着再把排好序的数组复制回原来的数组

# 2. 预备知识

首先了解以下堆的特性，堆和二叉排序树不同，它并没有左右子树的约束关系，而仅仅是有一个树根和左右子树的约束关系（树根的元素一定的最小的，假设针对的是小顶堆），并且使用数组实现的堆有一些特别的特性。

在讨论特性时，我们需要了解一下堆这种数据结构的实现（假设底层采用的是数组的实现），在我遇到的代码中有基于索引1开始的实现（也就是把数组索引为0的位置空出来，另作他用），也有使用索引位置为0开始的实现，它们的区别如下：

```
                    根节点索引为0              根节点索引为1
左孩子(数组下标)      (index * 2 + 1)          (index * 2)
右孩子(数组下标)      (index * 2 + 2)          (index*2 + 1)
父节点(数组下标)      (index - 1) / 2          (index/2)

```
也就是说采用索引0或者1开始其实无关紧要（但是也有人争议说采用*2 /2可以通过移位快速实现，也有人反驳提到 *2 /2 的移位实现在当今的硬件上和 +1 +2几乎误差，具体见参考资料2）


# 3. 实现

在堆排序实现时，需要注意2点：

1. 按照之前的讨论，感觉上我们应该需要额外的一个数组空间来存储排好序的结果，但是事实上我们可以复用之前的数组，因为当数组deleteRoot之后，整个堆实际上少一个元素，我们可以把删除的元素存储在空出的数组里面，但是这样做会带来一个问题，最后得到的结果时逆序的，因此为了可以正常排序，我们需要建立的堆和我们排序的情况相反的堆，也就是当要从小到大排列的时候，我们需要创建大顶堆；当需要从大到小的情况排列时，我们需要创建小顶堆。

2. 堆排序不要尝试去创建一个堆数据结构，也就是说当使用C++去实现的时候，不要创建一个堆的类，这样代码太复杂了，而仅仅是创建堆序就好了，代码比较轻量级一点。


## 3.1 自底向上的构建堆的方法

实现的伪代码如下：

堆排序包括两个步骤（1）创建堆（2）移除堆顶元素  直至整个堆元素处理完

- HeapSort算法

```
heapsort(a, count)

input: 长度为count乱序的数组

heapify(a, count) //首先创建堆

//在处理过程中使用变量end记录堆结束的位置（a[0:end]标识堆区域）
// a[end:count-1]是已经拍好的排好序的元素数组
end = count - 1  //用 end 记录堆中排好序和属于堆的元素

while end > 0 do
    swap(a[end],a[0])  //交换end和a[0]，把最大的元素移除
    end = end - 1      //end前移1，标识最大的元素被删除出堆，进入排好序的子列
    siftDown(a,0,end) //a[0]根节点位置被移除，堆性质被破坏，重新整理堆
```

可以从上面的伪代码看出，堆排序需要两个子函数 heapify（创建堆）和 siftDown（在堆的性质破坏之后恢复堆的性质）


- 子函数 heapify

procedure heapify(a, count)
    start = iParent(count-1)    //找到最后一个元素的父节点索引号

    while start >=0 do
        siftDown(a, start, count-1)
        start = start - 1


- 子函数 siftDown(a, start, end)

procedure siftDown(a, start, end)

    root = start

    while iLeftChild(root) <= end do
        child = iLeftChild(root)
        nextPos = root

        if a[nextPos] < a[child] then
            nextPos = child
        if child+1 <= end and a[nextPos] < a[child+1] then
            nextPos = child + 1

        if nextPos = root then
            return
        else
            swap(a[root], a[nextPos])
            root = nextPos;


以下是堆排序的C++实现代码(我个人手写的，可能还需要优化)

```
int parent(int childIndex)
{
	return (childIndex - 1) / 2;
}

int leftChild(int parentIndex)
{
	return (2 * parentIndex + 1);
}


//从pos位置节点开始下沉直到处理到叶节点
//arr:数组指针
//pos:当前调整的节点索引号（是一个子树的根节点的索引号）
//n：当前堆的元素总数
template<typename T>
void siftDown(T arr[], int pos, int n)
{
	int leftChildIndex = leftChild(pos);

	while (leftChildIndex < n)
	{
		//记录我们把根节点元素移动到的下一个位置
		//下一个位置有可能是左孩子或者右孩子（如果存在右孩子的话）
		//如果下一个节点位置通过比较根节点和左右孩子节点的值发现不需要
		//移动的话，说明我们根节点的值就应该保持原地不动，也就是说已经完成了
		int nextTravsalIndex = pos; 
		

		if (arr[nextTravsalIndex] < arr[leftChildIndex])
		{
			nextTravsalIndex = leftChildIndex;
		}

		//如果节点有右节点,并且右节点的值大于（左节点和根节点值中的较大者）
		if (leftChildIndex + 1 < n)
		{
			if (arr[nextTravsalIndex] < arr[leftChildIndex + 1])
			{
				nextTravsalIndex = leftChildIndex + 1;
			}
		}

		//我们把根节点上的元素往下沉，但是判断左右孩子都不大于这个根节点的数，
		//说明当前根节点的元素位置是符合堆性质的
		if (nextTravsalIndex == pos)
			return;

		std::swap(arr[nextTravsalIndex], arr[pos]);
		pos = nextTravsalIndex;
		leftChildIndex = leftChild(nextTravsalIndex);
	}
}

template<typename T>
void heapify(T arr[], int n)
{
	if (n <= 1)
		return;
	
	//从最后一个节点的父节点开始，依次减少到根节点，将每一棵子树都调整为大顶堆
	for (int i = parent(n-1); i >= 0; i--)
	{
		siftDown(arr, i, n - 1);
	}
}


template<typename T>
void heapSort(T arr[], int n)
{
	if (n <= 0)
		return;

	heapify(arr, n);
	
	while (n > 0)
	{
		std::swap(arr[0], arr[n - 1]);
		n--;
		siftDown(arr, 0, n);
	}
}
```

### 3.1.2 补充：递归构造下沉

在下沉的时候，可以使用迭代亦可使用递归的做法，上文中给出的是迭代的代码，此处补充一下递归下沉的代码：

```
/*
recursive version of siftDown
具体见参考资料4
*/

template<typename T>
void siftDown_recursive(T arr[], int pos, int n)
{
    //递归退出的出口
	if (pos > n)
		return;

	int leftChildIndex = leftChild(pos);
	int rightChildIndex = leftChildIndex + 1;
	
	T leftValue = arr[leftChildIndex];
	int maxValueIndex = pos;
	
	if (leftChildIndex < n && arr[pos] < arr[leftChildIndex])
		maxValueIndex = leftChildIndex;

	if (rightChildIndex < n && arr[maxValueIndex] < arr[rightChildIndex])
		maxValueIndex = rightChildIndex;

	if (maxValueIndex == pos)
	{
		return;
	}
	else
	{
		std::swap(arr[pos], arr[maxValueIndex]);
		siftDown(arr,maxValueIndex,n);
	}
}
```


### 3.1.3 补充：延伸阅读：见参考资料1，里面有提及一种叫”heapsort with bounce“的优化方式，我暂时没看明白，不理解它的优化点在何处，后续继续研究



## 3.2 自顶向下构造堆的方法

在上述的3.1中介绍的方法，创建堆的方法是从最后一个节点的父节点开始，依次往上去构造堆的方法进行的，这种做法有些tricky，可能第一时间不太容易想到，比较直观的一种做法是直接从乱序的数组第一个元素开始构造堆，这种方式应该最容易想到的做法，我们来尝试一下。

伪代码如下：

```
procedure heapify(a, count)
    end = 1    //end从第一个左孩子开始（根的左孩子，根的索引是0）
    while end < count
        siftUp(a,0,end)
        end = end + 1
```

```
procedure siftUp(a, start, end)
    child = end
    while child > start
        parent = iParent(child)  //计算parent的索引号
        if a[parent] < a[child] then
            swap(a[parent], a[child])
            child = parent
        else
            return
```
siftUp实现的堆不能像siftDown那样，在交换元素之后可以进行“堆的修复”，** siftUp这种方式实现的堆必须每次移除元素后都重建整个堆 **

```
procedure heapsort(a, count)
    heapify(a,count)

    end = count - 1
    while end > 0 do
        swap(a[end],a[0])
        heapify(a,end)
        end = end - 1

```

这种方式实现的C++代码如下所示：（我手写的代码，大量使用了递归）

```
template<typename T>
void siftUp(T arr[], int index, int n)
{
	int parentIndex = parent(index);

	if (parentIndex >= 0 && arr[index] > arr[parentIndex])
	{
		std::swap(arr[index], arr[parentIndex]);
		siftUp(arr,parentIndex,n);
	}
}


template<typename T>
void heapify(T arr[], int n)
{
	for (int i = 1; i < n; i++)
	{
		siftUp(arr, i, n);
	}
}


template<typename T>
void heapSort(T arr[], int n)
{
	if (n <= 1)
		return;

	heapify(arr, n);
	std::swap(arr[0], arr[n - 1]);
	heapSort(arr, n-1);
}
```


## 3.3 两种思路的比较和算法复杂度分析

3.1 的思路有点类似于从最底下的非叶节点开始，逐步向根移动，并且每次都“修补”好堆的结构
3.2 的思路是从空的堆开始，逐步构建一个完整的堆，它是从最上面的根的子节点开始的

需要根据算法中描述的具体操作步骤才能分析清楚这两者的特点：
3.1 具体来说是一种“ bottom up scan , and perform a sift-down” 也就是从整体来说是 “从下往上”，但是局部确是“从上往下”
3.2 具体来说是一种“ top down scan, and perform a sift-up"，从整体来说是”从上往下“，但是局部确实”从下往上“

Heapify的时间复杂度：
3.1 这种方式时间复杂度是O(N)
3.2 这种方式时间复杂度是O(NlogN)

但是两者实现的HeapSort的时间复杂度都是O(NlogN)，一般来说大家普遍实现的堆排序的算法是3.1中这种方式

在通过数组构建堆的时候，从数组的第一个位置开始依次向后，a[0]-->a[n-1] 这种方式称之为 sift-up的方式（从外在形式上看其实是Top-Down的方式，但是这里的sift-up指的是插入的每一步操作的方式，是将节点依次往上浮），从0开始创建一个堆；
在通过数组构建堆的时候，从数组的最后一个位置开始依次向前，a[n-1]-->a[0] 这种方式称之为 sift-down的方式（从外在形式上看其实是Bottom-Up的方式，但是sift-bottom指的是每一次操作一个元素采用的方式，是把节点往下沉），类似于修补一个损坏的堆

可以通过下面的事实了解到一些复杂度区别的本质：

- 往上浮的代价是节点距离根节点的距离
- 往下沉的代价是节点距离叶子节点的距离

很明显由于堆是完全二叉树，叶节点的节点数量明显比非叶节点多不少，于是从感性上就可以认识到往上浮肯定没有往下沉好。因为往上浮叶节点的操作代价最高，而有大量的叶节点，而往下沉根节点的代价最高，但是根节点只有一个而已。

另外这两种思路最后获得的堆结构可能是不同的，但是最终都能得到正确的排序结果。

更多的内容见参考资料部分



# 5. 参考资料

[1. Wiki: Heapsort](https://en.wikipedia.org/wiki/Heapsort)
[2. Why in a heap implemented by array the index 0 is left unused?](https://stackoverflow.com/questions/22900388/why-in-a-heap-implemented-by-array-the-index-0-is-left-unused)
3. Weiss :Data Structures and Algorithm Analysis in C++ (Fourth Edition) Chapter 7.5 Heap Sort
[4. 堆排序(Heapsort)](https://www.youtube.com/watch?v=j-DqQcNPGbE&t=816s)
[5.siftUp and siftDown operation in heap for heapifying an array](https://stackoverflow.com/questions/34329942/siftup-and-siftdown-operation-in-heap-for-heapifying-an-array)
[6.How can building a heap be O(n) time complexity?](https://stackoverflow.com/questions/9755721/how-can-building-a-heap-be-on-time-complexity)