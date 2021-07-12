---
title: JAVA 排序算法
date: 2021/3/8
description: JAVA 排序算法
top_img: >-
  https://fabian.oss-cn-hangzhou.aliyuncs.com/img/v2-e2ae4a8bbb1668c750bc9e64274f421f_hd.jpg
cover: >-
  https://fabian.oss-cn-hangzhou.aliyuncs.com/img/v2-e2ae4a8bbb1668c750bc9e64274f421f_hd.jpg
categories:
  - 算法
tags:
  - Java
  - 算法
abbrlink: 58938
---

# JAVA 排序算法

## 一、冒泡排序（Bubble Sort）

### 1. 算法简介

冒泡排序是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果它们的顺序错误就把它们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。 

### 2. 算法描述

- 比较相邻的元素。如果第一个比第二个大，就交换它们两个；
- 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对，这样在最后的元素应该会是最大的数；
- 针对所有的元素重复以上的步骤，除了最后一个；
- 重复步骤1~3，直到排序完成。

### 3. 动态演示

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015223238449-2146169197.gif)

### 4. 代码实现

~~~java
public static int[] bubbleSort(int[] array){
    if(array.length > 0){
        for(int i = 0;i<array.length;i++){
            for(int j = 0;j<array.length - 1 - i;j++){
                if(array[j] > array[j+1]){
                    int temp = array[j];
                    array[j] = array[j+1];
                    array[j+1] = temp;
                }
            }
        }
    }
    return array;
}
~~~

### 5. 算法分析

- **最佳情况：T(n) = O(n)**
- **最差情况：T(n) = O(n2)**
- **平均情况：T(n) = O(n2)**



## 二、选择排序（Selection Sort）

### 1. 算法简介

表现最稳定的排序算法之一，因为无论什么数据进去都是O(n2)的时间复杂度，所以用到它的时候，数据规模越小越好。

唯一的好处可能就是不占用额外的内存空间了吧。理论上讲，选择排序可能也是平时排序一般人想到的最多的排序方法了吧。

选择排序(Selection-sort)是一种简单直观的排序算法。

它的工作原理：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

### 2. 算法描述

n个记录的直接选择排序可经过n-1趟直接选择排序得到有序结果。具体算法描述如下：

- 初始状态：无序区为R[1..n]，有序区为空；
- 第i趟排序(i=1,2,3…n-1)开始时，当前有序区和无序区分别为R[1..i-1]和R(i..n）。该趟排序从当前无序区中-选出关键字最小的记录 R[k]，将它与无序区的第1个记录R交换，使R[1..i]和R[i+1..n)分别变为记录个数增加1个的新有序区和记录个数减少1个的新无序区；
- n-1趟结束，数组有序化了。

### 3. 动态演示

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015224719590-1433219824.gif)

### 4. 代码实现

```java
public static int[] selectionSort(int[] array){
	if(array.length > 0){
		for(int i = 0;i<array.length;i++){
			int minIndex = i;
			//遍历未剩余未排序元素中继续寻找最小元素
			for(int j = i;j<array.length;j++){
				if(array[j] < array[minIndex]){
					minIndex = j;
				}
			}
			if(minIndex != i){
				int temp = array[minIndex];
				array[minIndex] = array[i];
				array[i] = temp;
			}
		}
	}
	return array;
}
```
### 5. 算法分析

- **最佳情况：T(n) = O(n2)**
- **最差情况：T(n) = O(n2)**
- **平均情况：T(n) = O(n2)**



## 三、插入排序（Insertion Sort）

### 1. 算法简介

插入排序（Insertion-Sort）的算法描述是一种简单直观的排序算法。

它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

插入排序在实现上，通常采用in-place排序（即只需用到O(1)的额外空间的排序），因而在从后向前扫描过程中，需要反复把已排序元素逐步向后挪位，为最新元素提供插入空间。

### 2. 算法描述

一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：

- 从第一个元素开始，该元素可以认为已经被排序；
- 取出下一个元素，在已经排序的元素序列中从后向前扫描；
- 如果该元素（已排序）大于新元素，将该元素移到下一位置；
- 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置；
- 将新元素插入到该位置后；
- 重复步骤2~5。

### 3. 动态演示

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015225645277-1151100000.gif)

### 4. 代码实现

```java
public static int[] insertSort(int[] array){
	if(array.length > 0){			
		for(int i = 0 ;i<array.length - 1;i++){
			int current = array[i+1];
			int index = i;
			while(index >= 0 && current < array[index]){
				array[index + 1] = array[index]; 
				index--;
			}
			array[index+1] = current;
		}
	}
	return array;
}
```
### 5. 算法分析

- 最佳情况：**T(n) = O(n)**
- 最坏情况：**T(n) = O(n2)**  
- 平均情况：**T(n) = O(n2)**



## 四、希尔排序（Shell Sort）

### 1. 算法简介

希尔排序是希尔（Donald Shell）于1959年提出的一种排序算法。

希尔排序也是一种插入排序，它是简单插入排序经过改进之后的一个更高效的版本，也称为缩小增量排序，同时该算法是冲破O(n2）的第一批算法之一。

它与插入排序的不同之处在于，它会优先比较距离较远的元素。

希尔排序又叫缩小增量排序。

希尔排序是把记录按下表的一定增量分组，对每组使用直接插入排序算法排序；随着增量逐渐减少，每组包含的关键词越来越多，当增量减至1时，整个文件恰被分成一组，算法便终止。

### 2. 算法描述

我们来看下希尔排序的基本步骤，在此我们选择增量gap=length/2，缩小增量继续以gap = gap/2的方式，这种增量选择我们可以用一个序列来表示，{n/2,(n/2)/2...1}，称为增量序列。希尔排序的增量序列的选择与证明是个数学难题，我们选择的这个增量序列是比较常用的，也是希尔建议的增量，称为希尔增量，但其实这个增量序列不是最优的。此处我们做示例使用希尔增量。

先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，具体算法描述：

- 选择一个增量序列t1，t2，…，tk，其中ti>tj，tk=1；
- 按增量序列个数k，对序列进行k 趟排序；
- 每趟排序，根据对应的增量ti，将待排序列分割成若干长度为m 的子序列，分别对各子表进行直接插入排序。仅增量因子为1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。

### 3. 动态演示

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/1192699-20180319094116040-1638766271.png)

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/665FDA705E0082F9A8D33BB1C607BEEF6A2866AA_size752_w954_h537.gif)

### 4. 代码实现

```java
public static int[] shellSort(int[] array){

    if(array.length > 0){    
        int len = array.length;
        int gap = len / 2;
        while(gap > 0){
            for(int i = gap;i < len;i++){
                int temp = array[i];
                int index = i - gap;
                while(index >= 0 && array[index] > temp){
                    array[index + gap] = array[index];
                    index -= gap;
                }
                array[index + gap] = temp;
            }            
            gap /= 2;
        }

    }
    return array;
}    
```

### 5. 算法分析

- **最佳情况：T(n) = O(nlog2 n)**
- **最坏情况：T(n) = O(nlog2 n)**
- **平均情况：T(n) =O(nlog2n)**



## 五、归并排序（Merge Sort）

### 1. 算法简介

和选择排序一样，归并排序的性能不受输入数据的影响，但表现比选择排序好的多，因为始终都是O(n log n）的时间复杂度。代价是需要额外的内存空间。

归并排序是建立在归并操作上的一种有效的排序算法。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。

归并排序是一种稳定的排序方法。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为2-路归并。 

### 2. 算法描述

- 把长度为n的输入序列分成两个长度为n/2的子序列；
- 对这两个子序列分别采用归并排序；
- 将两个排序好的子序列合并成一个最终的排序序列。

### 3. 动态演示

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015230557043-37375010.gif)

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/60585793BABAD3B1DD1E71DBE6CB449818610E2A_size838_w954_h537.gif)

### 4. 代码实现

```java
public static int[] MergeSort(int[] array){
	if(array.length < 2){
		return array;
	}
	int mid = array.length /2;
	int[] left = Arrays.copyOfRange(array, 0, mid);
	int[] right = Arrays.copyOfRange(array, mid, array.length);
	return merge(MergeSort(left),MergeSort(right));	
}

public static int[] merge(int[] left,int[] right){
	int[] result = new int[left.length + right.length];
	for(int index = 0,i = 0, j = 0;index < result.length;index++){
		if(i >= left.length){
			result[index] = right[j++];
		}else if(j >= right.length){
			result[index] = left[i++];
		}else if(left[i] > right[j]){
			result[index] = right[j++];
		}else{
			result[index] = left[i++];
		}
	}
	return result;
	
}
```

### 5. 算法分析

- **最佳情况：T(n) = O(n)**
- 最差情况：**T(n) = O(nlogn)**
- 平均情况：**T(n) = O(nlogn)**



## 六、快速排序（Quick Sort）

### 1. 算法简介

快速排序的基本思想：通过一趟排序将待排记录分隔成独立的两部分，其中一部分记录的关键字均比另一部分的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序。

### 2. 算法描述

快速排序使用分治法来把一个串（list）分为两个子串（sub-lists）。具体算法描述如下：

- 从数列中挑出一个元素，称为 “基准”（pivot）；
- 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；
- 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

### 3. 动态演示

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015230936371-1413523412.gif)

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/2C853E23A97618AAD932A1DA89103646303147A9_size727_w954_h537.gif)

### 4. 代码实现

```java
public static void QuickSort(int[] array,int low,int hight){
	//if (array.length < 1 || low < 0 || hight >= array.length || low > hight) return null;
	if(low < hight){
		int privotpos = partition(array,low,hight);
		QuickSort(array,low,privotpos - 1);
		QuickSort(array,privotpos + 1,hight);			
	}

}

public static int partition(int[] array,int low,int hight){
	int privot = array[low];
	while(low < hight){
		while(low < hight && array[hight] >= privot) --hight;
		array[low] = array[hight];
		while(low < hight && array[low] <= privot) ++low;
		array[hight] = array[low];
	}
	array[low] = privot;
	return low;			
}
```
### 5. 算法分析

- **最佳情况：T(n) = O(nlogn)**
- **最差情况：T(n) = O(n2)**
- **平均情况：T(n) = O(nlogn)**



## 七、堆排序（Heap Sort）

### 1. 算法简介

堆的定义如下: n个元素的序列{k1, k2, ... , kn}当且仅当满足一下条件时，称之为堆。

​                ![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/20180610110041162)

可以将堆看做是一个完全二叉树。并且，每个结点的值都大于等于其左右孩子结点的值，称为大顶堆；或者每个结点的值都小于等于其左右孩子结点的值，称为小顶堆。

堆排序(Heap Sort)是利用堆进行排序的方法。其基本思想为：将待排序列构造成一个大顶堆(或小顶堆)，整个序列的最大值(或最小值)就是堆顶的根结点，将根节点的值和堆数组的末尾元素交换，此时末尾元素就是最大值(或最小值)，然后将剩余的n-1个序列重新构造成一个堆，这样就会得到n个元素中的次大值(或次小值)，如此反复执行，最终得到一个有序序列。

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/70)

### 2. 算法描述

- 将初始待排序关键字序列(R1,R2….Rn)构建成大顶堆，此堆为初始的无序区；
- 将堆顶元素R[1]与最后一个元素R[n]交换，此时得到新的无序区(R1,R2,……Rn-1)和新的有序区(Rn),且满足R[1,2…n-1]<=R[n]；
- 由于交换后新的堆顶R[1]可能违反堆的性质，因此需要对当前无序区(R1,R2,……Rn-1)调整为新堆，然后再次将R[1]与无序区最后一个元素交换，得到新的无序区(R1,R2….Rn-2)和新的有序区(Rn-1,Rn)。不断重复此过程直到有序区的元素个数为n-1，则整个排序过程完成。

### 3. 动态演示

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015231308699-356134237.gif)

### 4. 代码实现

```java
public static void heapAdjust(int[] array,int index,int length){
	//保存当前结点的下标
	int max = index;
	//当前节点左子节点的下标
	int lchild = 2*index;
	//当前节点右子节点的下标
	int rchild = 2*index + 1;
	if(length > lchild && array[max] < array[lchild]){
		max = lchild;
	}
	if(length > rchild && array[max] < array[rchild]){
		max = rchild;
	}
	//若此节点比其左右孩子的值小，就将其和最大值交换，并调整堆
	if(max != index){
		int temp = array[index];
		array[index] = array[max];
		array[max] = temp;
		heapAdjust(array,max,length);
	}
	
}

public static int[] heapSort(int[] array){
	int len = array.length;
	//初始化堆，构造一个最大堆
	for(int i = (len/2 - 1);i >= 0;i--){
		heapAdjust(array,i,len);
	}
	//将堆顶的元素和最后一个元素交换，并重新调整堆
	for(int i = len - 1;i > 0;i--){
		int temp = array[i];
		array[i] = array[0];
		array[0] = temp;
		
		heapAdjust(array,0,i);
	}
	return array;
}
```
### 5. 算法分析

- **最佳情况：T(n) = O(nlogn)**
- **最差情况：T(n) = O(nlogn)**
- **平均情况：T(n) = O(nlogn)**



## 八、计数排序（Counting Sort）

### 1. 算法简介

计数排序的核心在于将输入的数据值转化为键存储在额外开辟的数组空间中。 作为一种线性时间复杂度的排序，计数排序要求输入的数据必须是有确定范围的整数。

计数排序(Counting sort)是一种稳定的排序算法。计数排序使用一个额外的数组C，其中第i个元素是待排序数组A中值等于i的元素的个数。然后根据数组C来将A中的元素排到正确的位置。它只能对整数进行排序。

### 2. 算法描述

- 找出待排序的数组中最大和最小的元素；
- 统计数组中每个值为i的元素出现的次数，存入数组C的第i项；
- 对所有的计数累加（从C中的第一个元素开始，每一项和前一项相加）；
- 反向填充目标数组：将每个元素i放在新数组的第C(i)项，每放一个元素就将C(i)减去1。

### 3. 动态演示

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015231740840-6968181.gif)

### 4. 代码实现

```java
public static int[] countingSort(int[] array){
	if(array.length == 0){
		return array;
	}
	int bias ,min = array[0],max = array[0];
	//找出最小值和最大值
	for(int i = 0;i < array.length;i++){
		if(array[i] < min){
			min = array[i];
		}
		if(array[i] > max){
			max = array[i];
		}
	}
	//偏差
	bias = 0 - min;
	//新开辟一个数组
	int[] bucket = new int[max - min +1];
	//数据初始化为0
	Arrays.fill(bucket, 0);
	for(int i = 0;i < array.length;i++){
		bucket[array[i] + bias] += 1;
	}
 
	int index = 0;
	for(int i = 0;i < bucket.length;i++){
		int len = bucket[i];
		while(len > 0){
			array[index++] = i - bias;
			len --;
		}		
	}
	return array;
}
```
### 5. 算法分析

当输入的元素是n 个0到k之间的整数时，它的运行时间是 O(n + k)。计数排序不是比较排序，排序的速度快于任何比较排序算法。由于用来计数的数组C的长度取决于待排序数组中数据的范围（等于待排序数组的最大值与最小值的差加上1），这使得计数排序对于数据范围很大的数组，需要大量时间和内存。

- **最佳情况：T(n) = O(n+k)**
- **最差情况：T(n) = O(n+k)**
- **平均情况：T(n) = O(n+k)**

## 九、桶排序（Bucket Sort）

### 1. 算法简介

桶排序是计数排序的升级版。它利用了函数的映射关系，高效与否的关键就在于这个映射函数的确定。

桶排序 (Bucket sort)的工作的原理：**假设输入数据服从均匀分布，将数据分到有限数量的桶里，每个桶再分别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排**

### 2. 算法描述

- 人为设置一个BucketSize，作为每个桶所能放置多少个不同数值（例如当BucketSize==5时，该桶可以存放｛1,2,3,4,5｝这几种数字，但是容量不限，即可以存放100个3）；
- 遍历输入数据，并且把数据一个一个放到对应的桶里去；
- 对每个不是空的桶进行排序，可以使用其它排序方法，也可以递归使用桶排序；
- 从不是空的桶里把排好序的数据拼接起来。 

**注意，如果递归使用桶排序为各个桶排序，则当桶数量为1时要手动减小BucketSize增加下一循环桶的数量，否则会陷入死循环，导致内存溢出。**

### 3. 动态演示

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015232107090-1920702011.png)

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/DD6AB96E7E353C0B72A1068DE59E7B6A297C4EDD_size1514_w954_h537.gif)

### 4. 代码实现

```java
public static ArrayList<Integer> BucketSort(ArrayList<Integer> array, int bucketSize) {
    if (array == null || array.size() < 2)
        return array;
    int max = array.get(0), min = array.get(0);
    // 找到最大值最小值
    for (int i = 0; i < array.size(); i++) {
        if (array.get(i) > max)
            max = array.get(i);
        if (array.get(i) < min)
            min = array.get(i);
    }
    int bucketCount = (max - min) / bucketSize + 1;
    ArrayList<ArrayList<Integer>> bucketArr = new ArrayList<>(bucketCount);
    ArrayList<Integer> resultArr = new ArrayList<>();
    //构造桶
    for (int i = 0; i < bucketCount; i++) {
        bucketArr.add(new ArrayList<Integer>());
    }
    //往桶里塞元素
    for (int i = 0; i < array.size(); i++) {
        bucketArr.get((array.get(i) - min) / bucketSize).add(array.get(i));
    }
    for (int i = 0; i < bucketCount; i++) {
        if (bucketSize == 1) { 
            for (int j = 0; j < bucketArr.get(i).size(); j++)
                resultArr.add(bucketArr.get(i).get(j));
        } else {
            if (bucketCount == 1)
                bucketSize--;
            ArrayList<Integer> temp = BucketSort(bucketArr.get(i), bucketSize);
            for (int j = 0; j < temp.size(); j++)
                resultArr.add(temp.get(j));
        }
    }
    return resultArr;
}
```
### 5. 算法分析

桶排序最好情况下使用线性时间O(n)，桶排序的时间复杂度，取决与对各个桶之间数据进行排序的时间复杂度，因为其它部分的时间复杂度都为O(n)。很显然，桶划分的越小，各个桶之间的数据越少，排序所用的时间也会越少。但相应的空间消耗就会增大。 

- **最佳情况：T(n) = O(n+k)**
- **最差情况：T(n) = O(n+k)**
- **平均情况：T(n) = O(n2)**

## 十、基数排序（Radix Sort）

### 1. 算法简介

基数排序也是非比较的排序算法，对每一位进行排序，从最低位开始排序，复杂度为O(kn),为数组长度，k为数组中的数的最大的位数；

基数排序是按照低位先排序，然后收集；再按照高位排序，然后再收集；依次类推，直到最高位。有时候有些属性是有优先级顺序的，先按低优先级排序，再按高优先级排序。最后的次序就是高优先级高的在前，高优先级相同的低优先级高的在前。基数排序基于分别排序，分别收集，所以是稳定的。

### 2. 算法描述

- 取得数组中的最大数，并取得位数；
- arr为原始数组，从最低位开始取每个位组成radix数组；
- 对radix进行计数排序（利用计数排序适用于小范围数的特点）；

### 3. 动态演示

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015232453668-1397662527.gif)

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/0630D517B5FAABFC6609A0EA9BF55F7E89E08215_size1761_w954_h537.gif)

### 4. 代码实现

```java
public static int[] RadixSort(int[] array) {
    if (array == null || array.length < 2)
        return array;
    // 1.先算出最大数的位数；
    int max = array[0];
    for (int i = 1; i < array.length; i++) {
        max = Math.max(max, array[i]);
    }
    int maxDigit = 0;
    while (max != 0) {
        max /= 10;
        maxDigit++;
    }
    int mod = 10, div = 1;
    ArrayList<ArrayList<Integer>> bucketList = new ArrayList<ArrayList<Integer>>();
    for(int i = 0; i < 10;i++){
    	bucketList.add(new ArrayList<Integer>());
    }
    for(int i = 0;i < maxDigit;i++,mod *= 10 ,div *= 10){
    	for(int j = 0;j < array.length;j++){
    		int num = (array[j] % mod) / div;
    		bucketList.get(num).add(array[j]);
    	}
    	int index = 0;
    	for(int j = 0;j < bucketList.size();j++){
    		for(int k = 0;k < bucketList.get(j).size();k++){
    			array[index++] = bucketList.get(j).get(k);
    		}
			bucketList.get(j).clear();
    	}	
    }
    return array;
}
```
### 5. 算法分析

- **最佳情况：T(n) = O(n * k)**
- **最差情况：T(n) = O(n * k)**
- **平均情况：T(n) = O(n * k)**

> 基数排序有两种方法：
>
> - MSD 从高位开始进行排序 LSD 从低位开始进行排序 

 

> **基数排序 vs 计数排序 vs 桶排序**
>
> 这三种排序算法都利用了桶的概念，但对桶的使用方法上有明显差异：
>
> - 基数排序：根据键值的每位数字来分配桶
> - 计数排序：每个桶只存储单一键值
> - 桶排序：每个桶存储一定范围的数值



## 十一、排序算法总结

![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015233043168-1867817869.png)

- 稳定：如果a原本在b前面，而a=b,排序之后a仍然在b的前面；
- 不稳定：如果a原本在b前面，而a=b,排序之后a有可能会出现在b的后面；
- 内排序：所有排序操作都在内存中完成；
- 外排序：由于数据太大，因此把数据放在磁盘中，而排序通过磁盘和内存的数据传输才能进行；
- 时间复杂度：描述算法运行时间的函数，用大O符号表述；
- 空间复杂度：描述算法所需要的内存空间大小。
- n：数据规模
- k："桶"的个数
- In-place：占用常数内存，不占用额外内存
- Out-place：占用额外内存



![img](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/849589-20171015233220637-1055088118.png)

### **比较和非比较排序的区别**

常见的快速排序、归并排序、堆排序、冒泡排序等属于比较排序。在排序的最终结果里，元素之间的次序依赖于它们之间的比较。每个数都必须和其他数进行比较，才能确定自己的位置。

在冒泡排序之类的排序中，问题规模为n，又因为需要比较n次，所以平均时间复杂度为O(n²)。在归并排序、快速排序之类的排序中，问题规模通过分治法消减为logN次，所以平均时间复杂度为O(nlogn)。

**比较排序的优势是，适用于各种规模的数据，也不在乎数据的分布，都能进行排序。可以说，比较排序适用于一切需要排序的情况。**

计数排序、基数排序、桶排序则属于非比较排序。非比较排序是通过确定每个元素之前，应该有多少个元素来排序。针对数组arr,计算arr[i]之前有多少个元素，则唯一确定了arr[i]在排序后数组中的位置。

**非比较排序只要确定每个元素之前的已有的元素个数即可，所有一次遍历即可解决。算法时间复杂度O(n)。**

非比较排序的时间复杂度低，但由于非比较排序需要占用空间来确定唯一的位置。所以对数据规模和数据分布有一定的要求。