---
layout: post
title: "排序类算法的重点串联（C语言实现）"
tagline: ""
description: "排序类算法的重点串联"
category: algorithm
tags: [C,sorting,algorithm]
last_updated: 2017-12-04
---

排序算法是经典算法的重要分支之一，里面包含和诸多计算机算法实现的经典思想。基于比较的排序算法中的三种（归并排序、快速排序、堆排序）更是被列入了支配世界的10个算法之一。下面按照个人认为的理解难易程度以及使用到的思想分类逐一介绍（由易到难）。

## 穷举法（又称蛮力法、暴力求解法）
计算机尽管目前没有那么的智能，但是其远超人类的计算速度可以让其运行一些看似很笨的办法：就是在一个可能的范围内逐个试验，直到找到正确的结果为止。这种穷举的思想看似笨重，却在解决一些有限区间问题时有很好的效果。排序算法方面使用穷举思想的算法便是大家入门时候都学过的冒泡排序和选择排序算法。

### 冒泡排序
冒泡排序的思想比较简单，既然是基于比较的排序，那就可以相邻的两个数比较，如果逆序的话就交换位置，这样第一趟就能把最大值（抑或最小值，取决于排序需求）排到最后一个数。下一轮重复该动作，直到n-1遍之后，就能完成排序。该方法的代码如下：
```C++
//===冒泡算法===
int bubble_sort(ELETYPE *pArray,int ArraySize)
{
	int i,j;
	ELETYPE temp;
	BOOL exchange;

	if (pArray == NULL)//===对于排序算法输入参数的判断===
		return -1;
		
	for (i = 1; i < ArraySize ; i++)
	{
		for (j = 0; j < ArraySize - i ; j++)
		{
			exchange = FALSE;//===交换标记===
			if (pArray[j] > pArray[j+1])
			{
				temp = pArray[j+1];
				pArray[j+1] = pArray[j];
				pArray[j] = temp;
				exchange = TRUE;
			}
		}
		if (!exchange)
			return 0;
	}
	return 0;
}
```
该段代码有一个尚未提及的优化，即设置了一个交换标志，若在一轮比较后没有任何交换，则说明已排序完成，可以直接终止程序。需要指出的是，该优化并不影响该算法的复杂度，其复杂度仍为$$O(N^2)$$。

### 选择排序
以从小到大排序为例，选择排序的思想比冒泡排序更为直接：首先扫描整个待排序数组，找到最小的元素，将其交换到第一个位置上，随后从第二个位置开始，继续之前过程，经过n-1次循环后完成排序。具体实现代码如下：
```C++
//===选择排序===
int select_sort(ELETYPE *pArray,int ArraySize)
{
	int i,j,min;
	ELETYPE temp;
	if (pArray == NULL)//===对于输入参数的判断===
	{
		return -1;
	}
	for (i = 0; i < ArraySize - 1; i++)
	{
		min = i;
		for (j = i+1; j < ArraySize; j++)
		{
			if (pArray[min] > pArray[j])
			{
				min = j;
			}
		}
		temp = pArray[min];
		pArray[min] = pArray[i];
		pArray[i] = temp;

	}
	return 0;
}
```
这段代码中包含了经常会用到的两个代码片段：找最值和交换数组元素。这两个片段的实用性超越了选择排序本身的应用，由于选择排序本身较高的复杂度($$O(N^2)$$)，其实际应用很少，事实上工程上的代码鄙人目前还没见过用选择排序的。

## 减治法
减治是一种重要的解决问题的方法。我们要解决N个数的排序问题，减治的思想就是那么我们是不是可以先解决前面N-D个数的排序，如果解决了的话，第N个数的排序就是理所当然的事了。于是N的问题就变成了N-D的同类问题，同理可以先解决N-2D的同类问题，最终推导至解决极少数数据的问题，大大减少了问题难度。这里的D称为增量，可以为任意小于N的整数，增量为1时为一般的减治思想，增量为N/2时是减治的一种特殊情况，被称为分治法，在一些问题中，增量也可为固定甚至可变的数字。特殊问题中，如果能适当的选择增量，能极大程度上减小算法的复杂度。

### 插入排序

插入排序是减治法的典型应用，就是先解决最初始的问题，然后逐步递推。在已经解决N-i问题的前提下（即前面N-i个元素已经排序完成），将第i个元素插入前面已排序序列的适当位置。具体代码实现如下：
```C++
//===插入排序===
int insertion_sort(ELETYPE *pArray, int ArraySize)
{
	int i,j;
	ELETYPE temp;
	if (pArray == NULL)//===对于输入参数的判断===
	{
		return -1;
	}
	for (i = 1; i < ArraySize; i++)
	{
		temp = pArray[i];
		for (j = i; j > 0 && pArray[j-1] > temp; j-- )
			pArray[j] = pArray[j-1];
		pArray[j] = temp;

	}
	return 0;
}
```
这里为了避免频繁的使用交换用了一个小技巧，基于待插入参数必然导致已排序序列长度+1且所有大于带插入参数的数值都会后移一位的事实，避免了不断地使用交换。

### 希尔排序

希尔排序的名称源于其发明者donald shell，这种排序方法某种意义上是插入排序的特例，也是减治法中变化增量的代表案例。希尔排序通过比较相距一定间隔（增量D）的元素，这个间隔会随着比较的进行逐渐变小，一直到1之后完成最后一轮排序，最终得到完全排序的序列，因而希尔排序也被称为缩小增量排序。这个缩小增量组的选择可以是任意组合，只要其最小增量为1就行。然而，这个增量组的选择好坏会对算法的表现有巨大的影响。增量组的一种流行（但是效果一般)的选择是使用希尔建议的序列：初始值$$h_0 = \lfloor N / 2 \rfloor$$，递推公式：$$h_k = \lfloor h_{k+1} / 2 \rfloor$$。具体的代码实现如下：
```C++
//===希尔排序(希尔增量)===
int Shell_sort(ELETYPE A[],int ArraySize)
{
	int i,j,increment;
	ELETYPE temp;
	for (increment = ArraySize/2; increment > 0; increment /=2)
	{
		for (i = increment; i < ArraySize; i++)
		{
			temp = A[i];
			for (j = i; j >= increment; j-=increment)
			{
				if (temp < A[j-increment])
				{
					A[j] = A[j-increment];
				}
				else
					break;
			}
			A[j] = temp;
		}
	}
	return 0;
}
```
实际实现中用了与插入排序类似的方法避免元素的频繁交换。在增量的选择方面，Hibbard增量被认为是一个由于希尔增量的选择，其公式为$$h_n = 2^n - 1, (n = 1...n)$$；另一方面，Sedgewick提出的增量序列{1,5,19,41,109，...}，公式：$$h_n  = (9*4^n - 9*2^n + 1) or (4^n - 3*2^n+1)$$两个公式按结果顺序取值，这个增量序列在实践中效果良好，对于一兆以下的数据排序也有很好的效果。当然并不排除针对特殊问题有更好的增量序列。

## 分治法
分治法某种意义上是减治法的特例，其核心思想就是将大问题不断分解成两个子问题来解决，随后将两个子问题的结果融合。由于其本身思想的重要性以及在时间复杂度上的明显优化效果，分治法经常被单独提出，作为算法设计中的一个重要的思想。
### 归并排序
分治的思想在算法里面非常重要，其在排序算法方面也有很多及其有效的应用。归并排序就是分治法应用的典型例子。对于一个需要排序的数组，将其一分为二分别排序，然后将两个有序的子数组合并为一个有序数组。实际可以用递归优雅的实现，具体代码如下：
```C++
//===归并排序===
void merge(ELETYPE A[], ELETYPE tmpArray[],int Lpos, int Rpos, int RightEnd)
{
	int i,LeftEnd,NumElements,TmpPos,Lbackup;

	LeftEnd = Rpos - 1;
	Lbackup = TmpPos = Lpos;
	NumElements = RightEnd - Lpos + 1;

	while(Lpos <= LeftEnd && Rpos <= RightEnd)
		if(A[Lpos] <= A[Rpos])
			tmpArray[TmpPos++] = A[Lpos++];
		else
			tmpArray[TmpPos++] = A[Rpos++];
	while(Lpos <= LeftEnd)
		tmpArray[TmpPos++] = A[Lpos++];
	while(Rpos <= RightEnd)
		tmpArray[TmpPos++] = A[Rpos++];
   /* for(i = 0; i < NumElements; i++,RightEnd--)*/
		/*A[RightEnd] = tmpArray[RightEnd];*/
	for(i = Lbackup; i <= RightEnd; i++)
		A[i] = tmpArray[i];

	
}

void divide(ELETYPE A[], ELETYPE tmpArray[], int nLeft, int nRight)
{
	int nCenter;
	
	if(nLeft < nRight){
		nCenter = (nLeft + nRight)/2;
		divide(A,tmpArray,nLeft,nCenter);
		divide(A,tmpArray,nCenter+1,nRight);
		merge(A,tmpArray,nLeft,nCenter+1,nRight);
	}
}

void merge_sort(ELETYPE A[], int ArraySize)
{
	ELETYPE *tmpArray;

	tmpArray =(ELETYPE*) malloc(ArraySize*sizeof(ELETYPE));
	if(tmpArray == NULL)
		return;
	divide(A,tmpArray,0,ArraySize-1);
	free(tmpArray);
}
```
归并排序是及其稳定的，但是其主要缺点是需要线性的额外空间。当然理论上合并排序也能做到“在位”，但是会使算法变得及其复杂，若有这种需求，何不选择其他算法呢（滑稽~）。归并排序是亚二次的时间复杂度，但是就随机值的测试来看表现不如接下来要介绍的堆排序和快速排序。但是归并排序的merge思想是外部排序的核心思想。

### 快速排序
快速排序也是分治法的典型应用，和归并排序不同的是，归并排序按照元素位置进行划分，而快速排序是按元素的值进行划分。这个值称为枢纽元，快速排序在选定枢纽元后，对给定数组的元素进行划分，使得枢纽元左边的元素都小于枢纽元，右边的元素都大于枢纽元。随后递归处理即可。快速排序的核心问题在于枢纽元的选取，最简单的方法是选取子数组的第一个元素。这也是实践中证明非常糟糕的一种选法。更好的策略是在子数组中随机选取，然而如果待排序数组规模极大的话，随机数的开销也会变得很大。当然，理想的状态是每次取到子数组的中值，然而这又是一笔开销。于是，目前一种广泛使用的策略是选取子数组左端、右端和中间三个元素的中值作为枢纽元。这种选取枢纽元的方法称为三数中值分割法，实践证明，该方法减少了快速排序大约5%的平均时间。具体的代码如下。
```C++
//===快速排序===
void swap(ELETYPE *a, ELETYPE *b)
{
	ELETYPE tmp;
	tmp = *a;
	*a = *b;
	*b = tmp;
}
ELETYPE choose_pivot_using_median3(ELETYPE A[], int left, int right)
{
	int center = (left + right) / 2;
	if (A[left] > A[center])
		swap(&A[left], &A[center]);
	if (A[left] > A[right])
		swap(&A[left], &A[right]);
	if (A[center] > A[right])
		swap(&A[center], &A[right]);

	swap(&A[center], &A[right - 1]);
	return A[right - 1];

}

#define CUTOFF (3)

void q_sort(ELETYPE A[], int left, int right)
{
	int i, j;
	ELETYPE pivot;

	if (left + CUTOFF <= right)
	{
		pivot = choose_pivot_using_median3(A, left, right);
		i = left;
		j = right - 1;
		for (;;)
		{
			while (A[++i] < pivot) {}
			while (A[--j] > pivot) {}
			if (i < j)
				swap(&A[i], &A[j]);
			else
				break;
		}
		swap(&A[i], &A[right - 1]);
		q_sort(A, left, i - 1);
		q_sort(A, i + 1, right);
	}
	else
		insertion_sort(A + left, right - left + 1); //子数组元素小于CUTOFF值时改用插入排序

}

void quick_sort(ELETYPE A[], int ArraySize) 
{
	q_sort(A, 0, ArraySize - 1);
}
```
代码中需要特别注意的是在选取pivot的时候将pivot移到了倒数第二的位置，这是为了后续比较的方便以及防止游标越界。子数组分好后需再将pivot移回正确的位置，在例程中就是游标i的位置。快速排序是目前已知的平均运行时间最短的基于比较的排序方式，然而它也有自己的缺点：不稳定。另外，在快速排序中，当子数组足够小时（<20），若改用插入排序，即可节省多余的递归调用，优化运行时间。

## 变治法
变治法的核心思想是先“变”在“治”，“变”一定是为了将原来的问题转化成一个更简单的问题，而“治”就是对转化后的问题进行求解。事实上，在实际工程问题的解决中经常用到这一类的方法，只是没有意识到而已。
### 堆排序
排序算法中变治法的典型应用就是堆排序。这种变属于“变换为另一个问题的实例，而该问题的算法是已知的”。我们首先利用二叉堆的数组实现，将待排序序列建立为一个MAX堆（即根节点大于子节点的二叉堆）。然后使用delete max的方法，将最大值（即根节点值）与数组末尾值互换，随后堆大小-1，下滤末尾值。依次循环N-1次后，剩余元素即已完成排序。代码如下：
```C++
//===堆排序（max堆）☆☆☆===
#define LEFTCHILD(i) (2*i+1)
void perc_down(ELETYPE A[],int i,int ArraySize)//i:需下滤元素的下标
{
	int child = 0;
	ELETYPE tmp;
	for (tmp = A[i]; LEFTCHILD(i) < ArraySize; i = child)
	{
		child = LEFTCHILD(i);
		if(child != ArraySize-1 && A[child + 1] > A[child] )
			child++;
		if(A[child] > tmp)
			A[i] = A[child];
		else
			break;
	}
	A[i] = tmp;
}

void heap_sort(ELETYPE A[],int ArraySize)
{
	int i;
	ELETYPE tmp;
	for(i = ArraySize/2; i >= 0; i--)
		perc_down(A,i,ArraySize); //以现有数组建立max堆
	for(i = ArraySize-1; i > 0; i-- )
	{
		tmp = A[i];
		A[i] = A[0];
		A[0] = tmp;
		perc_down(A,0,i); //delete max
	}
		
}
```
堆排序事实上展示了从另一个方面看问题，问题会被简化不少。也展示了数据结构的威力，正确的选择数据结构能够大幅简化问题的难度。包括当前流行的神经网络，也是一种数据结构。掌握各种基础的数据结构可以说是算法设计的基础。