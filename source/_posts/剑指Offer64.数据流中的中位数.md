---
title: 64.StreamMedian数据流中的中位数(Coding Interviews )
date: 2018-08-22 09:58:44
categories: 2018年8月
tags: [Coding Interviews,Max Heap,Template]


---
# 64.StreamMedian数据流中的中位数(Coding Interviews )
如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。
 

<!-- more -->

将数据在容器中排序，将整个数据容器分隔成两部分。位于容器左边部分的数据比右边的数据小。中位数可以由两个指针指向的数据得到。如果容器中数据的数目是奇数，那么两个指针指向同一个数据。指针P1指向左边部分最大的数，指针P2指向右边部分最小的数。

如果能保证数据容器左边的数据都小于右边的数据，即使左右两边内部的数据没有排序，也可以根据左边最大的数及右边最小的数得到中位数。

使用一个最大堆来实现左边的数据容器，使用最小堆实现右边的数据容器。往堆中插入一个数据的时间效率是O(logn).由于只需要O(1）时间就可以得到位于堆顶的数据，因此得到中位数的时间效率是O(1)。

为了保证数据平均分配到两个堆中，可以在数据的总数目是偶数时把新数据插入到最小堆中，否则插入到最大堆中。

还要保证最大堆中里的所有数据都要小于最小堆中的数据。当数据的总数目是偶数时，按照前面的分配规则会把新的数据插入到最小堆中。如果此时这个新的数据比最大堆中的一些数据要小，那么

首先可以把这个新的数据插入到最大堆中，

接着把最大堆中的最大的数字拿出来插入到最小堆中。

由于最终插入到最小堆的数字是原最大堆中最大的数字，这样就保证了最小堆中所有数字都大于最大堆中的数字。

### 代码如下：

	
	#include <stdio.h>
	#include <algorithm>
	#include <vector>
	#include <functional>
	
	using namespace std;
	
	template<typename T> class DynamicArray
	{
	public:
	    void Insert(T num)
	    {
	        if (((min.size() + max.size()) & 1) == 0) {// even length 
	            if (max.size() > 0 && num < max[0]) {//new num is lower than  some numbers in the maxheap
	                max.push_back(num);
	                push_heap(max.begin(), max.end(), less<T>());// make the max heap
	
	                num = max[0];
	
	                pop_heap(max.begin(), max.end(), less<T>());
	                max.pop_heap();
	            }
	            min.push_heap(num);
	            push_heap(min.begin(), min.end(), greater<T>());//make the min heap
	        }
	        else {//odd length
	            if (min.size() > 0 && min[0] < num) {
	                min.push_back(num);
	                push_heap(min, begin(), min.end(), greater<T>());
	
	                num = min[0];
	
	                pop_heap(min.begin(), min.end(), greater<T>());
	                min.pop_heap();
	            }
	
	            max.push_back(num);
	            push_heap(max.begin(), max.end(), less<T>());
	        }
	    }
	
	    T GetMedian()
	    { 
	        int size = min.size() + max.size();
	        if (size == 0) throw exception("No numbers are available");
	        T median = 0;
	        if ((size & 1) == 1) meidan = min[0];// only has one number
	        else median = (min[0] + max[0]) / 2;
	        return median;
	    }
	
	private:
	    vector<T> min;//min heap
	    vector<T> max;//max heap
	};
	
	// ==================== Test Code ====================
	void Test(char* testName, DynamicArray<double>& numbers, double expected)
	{
	    if(testName != NULL)
	        printf("%s begins: ", testName);
	
	    if(abs(numbers.GetMedian() - expected) < 0.0000001)
	        printf("Passed.\n");
	    else
	        printf("FAILED.\n");
	}
	
	int main(int argc, char* argv[])
	{
	    DynamicArray<double> numbers;
	
	    printf("Test1 begins: ");
	    try
	    {
	        numbers.GetMedian();
	        printf("FAILED.\n");
	    }
	    catch(exception e)
	    {
	        printf("Passed.\n");
	    }
	
	    numbers.Insert(5);
	    Test("Test2", numbers, 5);
	
	    numbers.Insert(2);
	    Test("Test3", numbers, 3.5);
	
	    numbers.Insert(3);
	    Test("Test4", numbers, 3);
	
	    numbers.Insert(4);
	    Test("Test6", numbers, 3.5);
	
	    numbers.Insert(1);
	    Test("Test5", numbers, 3);
	
	    numbers.Insert(6);
	    Test("Test7", numbers, 3.5);
	
	    numbers.Insert(7);
	    Test("Test8", numbers, 4);
	
	    numbers.Insert(0);
	    Test("Test9", numbers, 3.5);
	
	    numbers.Insert(8);
	    Test("Test10", numbers, 4);
	
		return 0;
	}
	
