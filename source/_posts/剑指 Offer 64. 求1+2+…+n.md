---

title: 剑指 Offer 64. 求1+2+…+n

date: 2021-12-09 10:12:12

categories: 2021年12月

tags: [Leetcode, Bit Manipulation, Recursion, Brainteaser]

---

求 1+2+...+n ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

<!-- more -->


[TOC]





示例 1：
    
    输入: n = 3
    输出: 6
示例 2：
    
    输入: n = 9
    输出: 45


限制：
    
    1 <= n <= 10000

# 前言

首先我们梳理题目要求。题目要求我们不能使用乘除法、`for`、`while`、`if`、`else`、`switch`、`case` 等关键字及条件判断语句，因此我们手里能用的工具很少，列举出来发现只有加减法、赋值、位运算符以及逻辑运算符。

# 方法一：递归

**思路和算法**

试想一下如果不加限制地使用递归的方法来实现这道题，相信大家都能很容易地给出下面的实现（以 C++ 为例）：

```C++ [sol0-C++]
class Solution {
public:
    int sumNums(int n) {
        return n == 0 ? 0 : n + sumNums(n - 1);
    }
};
```

通常实现递归的时候我们都会利用条件判断语句来决定递归的出口，但由于题目的限制我们不能使用条件判断语句，那么我们是否能使用别的办法来确定递归出口呢？答案就是逻辑运算符的短路性质。

以逻辑运算符 `&&` 为例，对于 `A && B` 这个表达式，如果 `A` 表达式返回False  ，那么 `A && B` 已经确定为 False ，此时不会去执行表达式 `B`。同理，对于逻辑运算符 `||`， 对于 `A || B` 这个表达式，如果 `A` 表达式返回 True ，那么 `A || B` 已经确定为 True ，此时不会去执行表达式 `B`。

利用这一特性，我们可以将判断是否为递归的出口看作 `A && B` 表达式中的 `A` 部分，递归的主体函数看作 `B` 部分。如果不是递归出口，则返回 True，并继续执行表达式 `B` 的部分，否则递归结束。当然，你也可以用逻辑运算符 `||` 给出类似的实现，这里我们只提供结合逻辑运算符 `&&` 的递归实现。
<details>
    <summary>C++</summary>
    
```C++ [sol1-C++]
class Solution {
public:
    int sumNums(int n) {
        n && (n += sumNums(n-1));
        return n;
    }
};
```
</details>
<details>
    <summary>Java</summary>
    
```Java [sol1-Java]
class Solution {
    public int sumNums(int n) {
        boolean flag = n > 0 && (n += sumNums(n - 1)) > 0;
        return n;
    }
}
```
</details>
<details>
    <summary>TypeScript</summary>
 
```TypeScript [sol1-TypeScript]
var sumNums = function(n: number): number {
    n && (n += sumNums(n - 1));
    return n;
};
```
</details>
<details>
    <summary>Golang</summary>
 
```golang [sol1-Golang]
func sumNums(n int) int {
    ans := 0
    var sum func(int) bool
    sum = func(n int) bool {
        ans += n
        return n > 0 && sum(n-1)
    }
    sum(n)
    return ans
}
```
</details>

**复杂度分析**

- 时间复杂度：*O(n)*。递归函数递归 *n* 次，每次递归中计算时间复杂度为 *O(1)*，因此总时间复杂度为 *O(n)*。
- 空间复杂度：*O(n)*。递归函数的空间复杂度取决于递归调用栈的深度，这里递归函数调用栈深度为 *O(n)*，因此空间复杂度为 *O(n)*。

# 方法二：快速乘

**思路和算法**

考虑 `A` 和 `B` 两数相乘的时候我们如何利用加法和位运算来模拟，其实就是将 `B` 二进制展开，如果 `B` 的二进制表示下第 *i* 位为 1，那么这一位对最后结果的贡献就是 *A*(1<<i)* ，即 *A<<i*。我们遍历 `B` 二进制展开下的每一位，将所有贡献累加起来就是最后的答案，这个方法也被称作「俄罗斯农民乘法」，感兴趣的读者可以自行网上搜索相关资料。这个方法经常被用于两数相乘取模的场景，如果两数相乘已经超过数据范围，但取模后不会超过，我们就可以利用这个方法来拆位取模计算贡献，保证每次运算都在数据范围内。

下面给出这个算法的 C++ 实现：

<details>
    <summary>C++</summary>
 
```C++ [sol20-C++]
int quickMulti(int A, int B) {
    int ans = 0;
    for ( ; B; B >>= 1) {
        if (B & 1) {
            ans += A;
        }
        A <<= 1;
    }
    return ans;
}
```
</details>


回到本题，由等差数列求和公式我们可以知道 1+2+...+n 等价于 n(n+1)/2 ，对于除以 *2* 我们可以用右移操作符来模拟，那么等式变成了 *n(n+1)>>1*，剩下不符合题目要求的部分即为 *n(n+1)*，根据上文提及的快速乘，我们可以将两个数相乘用加法和位运算来模拟，但是可以看到上面的 C++ 实现里我们还是需要循环语句，有没有办法去掉这个循环语句呢？答案是有的，那就是自己手动展开，因为题目数据范围 *n* 为 *[1,10000]*，所以 *n* 二进制展开最多不会超过 *14* 位，我们手动展开 *14* 层代替循环即可，至此满足了题目的要求，具体实现可以参考下面给出的代码。


<details>
    <summary>C++</summary>
 
```C++ [sol2-C++]
class Solution {
public:
    int sumNums(int n) {
        int ans = 0, A = n, B = n + 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        (B & 1) && (ans += A);
        A <<= 1;
        B >>= 1;

        return ans >> 1;
    }
};
```
</details>
<details>
    <summary>Java</summary>
 
```Java [sol2-Java]
class Solution {
    public int sumNums(int n) {
        int ans = 0, A = n, B = n + 1;
        boolean flag;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        flag = ((B & 1) > 0) && (ans += A) > 0;
        A <<= 1;
        B >>= 1;

        return ans >> 1;
    }
}
```
</details>
<details>
    <summary>TypeScript</summary>
 

```TypeScript [sol2-TypeScript]
var sumNums = function(n: number): number {
    let ans: number = 0, A: number = n, B: number = n + 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    (B & 1) && (ans += A);
    A <<= 1;
    B >>= 1;

    return ans >> 1;
};
```
</details>
<details>
    <summary>Golang</summary>
 
```golang [sol2-Golang]
func sumNums(n int) int {
    ans, A, B := 0, n, n + 1
    addGreatZero := func() bool {
        ans += A
        return ans > 0
    }

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1

    _ = ((B & 1) > 0) && addGreatZero()
    A <<= 1
    B >>= 1
    return ans >> 1
}
```
</details>
