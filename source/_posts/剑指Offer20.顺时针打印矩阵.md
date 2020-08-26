---
title: 20.PrintMatrix顺时针打印矩阵(CodingInterview)

date: 2018-08-24 09:35:00

categories: 2018年8月

tags: [Coding Interviews,Array]


---
 

与其类似的一道题是LeetCode上的54. Spiral Matrix

题目：输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

<!-- more -->

## 思路：

设矩阵的行数为row，列数为col，以一个5X5的矩阵为例，第一圈左上角坐标为（0，0），第二圈左上角坐标为（1，1），行列坐标相同，因此选取（start，start）作为起点。

最后一圈只有一个数字，对应坐标为（2，2）。注意到5>2*2。对于6X6的矩阵6>2*2仍然成立。因此可以得到让循环继续的条件是

	col>start*2 && row>start*2

可以解释为因为矩阵是对称的，所以为了能完整打印一圈startX和startY不会超出行列数的一半

考虑如何实现打印一圈的功能，可以分为向右->向下->向上->向左四步，值得注意的是最后一圈可能退化成只有一行，一列或一个数字，因此此时打印一圈就不需要四步，而缩减为三步、两步、一步。
所以要分析打印每一步的前提条件。

打印第一步时候不需要前提条件

打印第二步时至少需要有两行，endY>startY

打印第三步时除了至少需要有两行还至少需要有两列 endY>startY && endX>startX

打印第四步时至少需要三行两列 endY>startY+1 && endX>startX

与此同时还要注意每一步的起始位置不要与上一步重合


	//==================================================================
	// 《剑指Offer——名企面试官精讲典型编程题》代码

	// 面试题29：顺时针打印矩阵
	// 题目：输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

	#include <cstdio>

	void PrintMatrixInCircle(int** numbers, int columns, int rows, int start);
	void printNumber(int number);

	void PrintMatrixClockwisely(int** numbers, int columns, int rows)
	{
	    if (numbers == nullptr || columns <= 0 || rows <= 0)
	        return;

	    int start = 0;

	    while (columns > start * 2 && rows > start * 2)
	    {
	        PrintMatrixInCircle(numbers, columns, rows, start);

	        ++start;
	    }
	}

	void PrintMatrixInCircle(int** numbers, int columns, int rows, int start)
	{
	    int endX = columns - 1 - start;
	    int endY = rows - 1 - start;

	    // 从左到右打印一行
	    for (int i = start; i <= endX; ++i)
	    {
	        int number = numbers[start][i];
	        printNumber(number);
	    }

	    // 从上到下打印一列
	    if (start < endY)
	    {
	        for (int i = start + 1; i <= endY; ++i)
	        {
	            int number = numbers[i][endX];
	            printNumber(number);
	        }
	    }

	    // 从右到左打印一行
	    if (start < endX && start < endY)
	    {
	        for (int i = endX - 1; i >= start; --i)
	        {
	            int number = numbers[endY][i];
	            printNumber(number);
	        }
	    }

	    // 从下到上打印一行
	    if (start < endX && start < endY - 1)
	    {
	        for (int i = endY - 1; i >= start + 1; --i)
	        {
	            int number = numbers[i][start];
	            printNumber(number);
	        }
	    }
	}

	void printNumber(int number)
	{
	    printf("%d\t", number);
	}

	// ====================测试代码====================
	void Test(int columns, int rows)
	{
	    printf("Test Begin: %d columns, %d rows.\n", columns, rows);

	    if (columns < 1 || rows < 1)
	        return;

	    int** numbers = new int*[rows];
	    for (int i = 0; i < rows; ++i)
	    {
	        numbers[i] = new int[columns];
	        for (int j = 0; j < columns; ++j)
	        {
	            numbers[i][j] = i * columns + j + 1;
	        }
	    }

	    PrintMatrixClockwisely(numbers, columns, rows);
	    printf("\n");

	    for (int i = 0; i < rows; ++i)
	        delete[](int*)numbers[i];

	    delete[] numbers;
	}

	int main(int argc, char* argv[])
	{
	    /*
	    1
	    */
	    Test(1, 1);

	    /*
	    1    2
	    3    4
	    */
	    Test(2, 2);

	    /*
	    1    2    3    4
	    5    6    7    8
	    9    10   11   12
	    13   14   15   16
	    */
	    Test(4, 4);

	    /*
	    1    2    3    4    5
	    6    7    8    9    10
	    11   12   13   14   15
	    16   17   18   19   20
	    21   22   23   24   25
	    */
	    Test(5, 5);

	    /*
	    1
	    2
	    3
	    4
	    5
	    */
	    Test(1, 5);

	    /*
	    1    2
	    3    4
	    5    6
	    7    8
	    9    10
	    */
	    Test(2, 5);

	    /*
	    1    2    3
	    4    5    6
	    7    8    9
	    10   11   12
	    13   14   15
	    */
	    Test(3, 5);

	    /*
	    1    2    3    4
	    5    6    7    8
	    9    10   11   12
	    13   14   15   16
	    17   18   19   20
	    */
	    Test(4, 5);

	    /*
	    1    2    3    4    5
	    */
	    Test(5, 1);

	    /*
	    1    2    3    4    5
	    6    7    8    9    10
	    */
	    Test(5, 2);

	    /*
	    1    2    3    4    5
	    6    7    8    9    10
	    11   12   13   14    15
	    */
	    Test(5, 3);

	    /*
	    1    2    3    4    5
	    6    7    8    9    10
	    11   12   13   14   15
	    16   17   18   19   20
	    */
	    Test(5, 4);

	    return 0;
	}
