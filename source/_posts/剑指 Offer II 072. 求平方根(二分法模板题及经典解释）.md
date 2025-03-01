---

title: 剑指 Offer II072

date: 2021-12-13 15:23:12

categories: 2021年12月

tags: [LeetCode, Math, Binary Search]


---
 

实现 int sqrt(int x) 函数。

计算并返回 x 的平方根，其中 x 是非负整数。

由于返回类型是整数，结果只保留整数的部分，小数部分将被舍去。
 
<!-- more -->
[TOC]

示例 1:
    
    输入: 4
    输出: 2
示例 2:
    
    输入: 8
    输出: 2
    说明: 8 的平方根是 2.82842..., 
         由于返回类型是整数，小数部分将被舍去。
 注意：本题与[69.x的平方根](https://leetcode-cn.com/problems/sqrtx/)相同

# 📖 文字题解

## 前言

本题是一道常见的面试题，面试官一般会要求面试者在不使用sqrt 函数的情况下，得到 *x* 的平方根的整数部分。一般的思路会有以下几种：

- 通过其它的数学函数代替平方根函数得到精确结果，取整数部分作为答案；

- 通过数学方法得到近似结果，直接作为答案。

# 二分法典型解法(二分法模板题及经典解释）[^1]

## 题意分析

+ 这道题要求我们实现平方根函数，输入是一个非负整数，输出也是一个整数；
+ 但是题目当中说：结果只保留整数的部分，小数部分将被舍去。这是什么意思呢？我们分析一下示例。

示例 1：

```
输入: 4
输出: 2
```

这是显然的，*4* 本身是一个完全平方数，*2^2 = 4*。虽然在数学上一个数的平方根有正有负，但是这个题目只要求我们返回算术平方根。

示例 2 ：

```
输入: 8
输出: 2
```

因为 *8* 的平方根实际上是 *2.82842*，题目要求我们将小数部分舍去。因此输出 *2*。于是我们知道：由于输出结果的时候，需要将小数部分舍去，因此问题的答案，平方以后一定不会严格大于输入的整数。这里返回 *3* 就不对了，这是因为 *3^2 = 9 > 8*。

## 思路分析

从题目的要求和示例我们可以看出，这其实是一个查找整数的问题，并且这个整数是有范围的。

+ 如果这个整数的平方 **恰好等于** 输入整数，那么我们就找到了这个整数；
+ 如果这个整数的平方 **严格大于** 输入整数，那么这个整数肯定不是我们要找的那个数；
+ 如果这个整数的平方 **严格小于** 输入整数，那么这个整数 **可能** 是我们要找的那个数（重点理解这句话）。

因此我们可以使用「二分查找」来查找这个整数，不断缩小范围去猜。

+ 猜的数平方以后大了就往小了猜；
+ 猜的数平方以后恰恰好等于输入的数就找到了；
+ 猜的数平方以后小了，可能猜的数就是，也可能不是。


很容易知道，题目要我们返回的整数是有范围的，直觉上一个整数的平方根肯定不会超过它自己的一半，但是 *0* 和 *1* 除外，因此我们可以在 *1* 到输入整数除以 *2* 这个范围里查找我们要找的平方根整数。*0* 单独判断一下就好。

**参考代码**：

<details>
    <summary>二分查找Java</summary>
    
```java
public class Solution {

    public int mySqrt(int x) {
        // 特殊值判断
        if (x == 0) {
            return 0;
        }
        if (x == 1) {
            return 1;
        }

        int left = 1;
        int right = x / 2;
        // 在区间 [left..right] 查找目标元素
        while (left < right) {
            int mid = left + (right - left + 1) / 2;
            // 注意：这里为了避免乘法溢出，改用除法
            if (mid > x / mid) {
                // 下一轮搜索区间是 [left..mid - 1]
                right = mid - 1;
            } else {
                // 下一轮搜索区间是 [mid..right]
                left = mid;
            }
        }
        return left;
    }
}
```

</details>

**复杂度分析**：

+ 时间复杂度：O(logx) ，每一次搜索的区间大小约为原来的1/2 ，时间复杂度为 O(log_2x)=O(logx) ；
+ 空间复杂度：*O(1)*。


**对代码编写逻辑的解释**：

一、写 `if` 和 `else` 的原因：

+ 猜的数是 `mid` ，根据上面的分析，如果 `mid` 的平方 **严格大于** `x`，`mid` 肯定不是解，比 `mid` 大的整数也肯定不是解，因此问题的答案只可能存在区间 `[left..mid - 1]`，此时设置 `right = mid - 1`；
+ `else` 的情况就是 `if` 的反面，只要 `if` 的分支和对应的区间分析对了，`else` 的区间是  `[left..mid - 1]` 的反面区间，即 ` [mid..right]` ，此时设置 `left = mid`。

二、为什么最后返回 `left`。

+ 退出 `while (left < right)` 循环的时候，由于边界搜索是 `left = mid` 与 `right = mid - 1`，因此退出循环的时候一定有 `left` 与 `right` 重合，返回 `right` 也可以。

---

## 问题：`mid` 为什么要加 `1`？

对着错误的测试用例打印出变量 `left` 、`right` 和 `mid` 的值看一下就很清楚了。

**`mid` 不加 `1` 会造成死循环的代码**：

<details>
    <summary>二分查找Java</summary>
    
```java
public class Solution {

    public int mySqrt(int x) {
        if (x == 0) {
            return 0;
        }
        if (x == 1) {
            return 1;
        }

        int left = 1;
        int right = x / 2;
        // 在区间 [left..right] 查找目标元素
        while (left < right) {
            // 取中间数 mid 下取整时
            int mid = left + (right - left) / 2;

            // 调试语句开始
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("left = " + left + ", right = " + right + ", mid = " + mid);
            // 调试语句结束

            // 注意：这里为了避免乘法溢出，改用除法
            if (mid > x / mid) {
                // 下一轮搜索区间是 [left..mid - 1]
                right = mid - 1;
            } else {
                // 下一轮搜索区间是 [mid..right]
                left = mid;
            }
        }
        return left;
    }

    public static void main(String[] args) {
        Solution solution = new Solution();
        int x = 9;
        int res = solution.mySqrt(x);
        System.out.println(res);
    }
}
```

</details>

控制台输出：

```
left = 1, right = 4, mid = 2
left = 2, right = 4, mid = 3
left = 3, right = 4, mid = 3
left = 3, right = 4, mid = 3
left = 3, right = 4, mid = 3
left = 3, right = 4, mid = 3
```

**在区间只有 *2* 个数的时候**，本题 `if`、`else` 的逻辑区间的划分方式是：`[left..mid - 1]` 与 `[mid..right]`。如果 `mid` 下取整，在区间只有 *2* 个数的时候有 `mid` 的值等于 `left`，一旦进入分支 `[mid..right]` 区间不会再缩小，出现死循环。

[image.png](https://pic.leetcode-cn.com/1639366986-BFgeWx-image.png)


**解决办法**：把取中间数的方式改成上取整。



# 方法一：袖珍计算器算法

「袖珍计算器算法」是一种用指数函数 exp  和对数函数 ln 代替平方根函数的方法。我们通过有限的可以使用的数学函数，得到我们想要计算的结果。

我们将sqrt(x) 写成幂的形式 *x^{1/2}*，再使用自然对数 *e* 进行换底，即可得到

    sqrt(x)=x^{1/2} = (e^lnx)^{1/2}=e^{lnx/2}

这样我们就可以得到sqrt(x)的值了。

**注意：** 由于计算机无法存储浮点数的精确值（浮点数的存储方法可以参考 [IEEE 754](https://baike.baidu.com/item/IEEE%20754)，这里不再赘述），而指数函数和对数函数的参数和返回值均为浮点数，因此运算过程中会存在误差。例如当 *x = 2147395600* 时，(e^lnx/2) 的计算结果与正确值 *46340* 相差 *10^{-11}*，这样在对结果取整数部分时，会得到 *46339* 这个错误的结果。

因此在得到结果的整数部分 ans 后，我们应当找出 ans 与 ans+1  中哪一个是真正的答案。
<details>
    <summary>袖珍计算器算法C++</summary>


```C++ [sol1-C++]
class Solution {
public:
    int mySqrt(int x) {
        if (x == 0) {
            return 0;
        }
        int ans = exp(0.5 * log(x));
        return ((long long)(ans + 1) * (ans + 1) <= x ? ans + 1 : ans);
    }
};
```
</details>
<details>
    <summary>袖珍计算器算法Java</summary>

```Java [sol1-Java]
class Solution {
    public int mySqrt(int x) {
        if (x == 0) {
            return 0;
        }
        int ans = (int) Math.exp(0.5 * Math.log(x));
        return (long) (ans + 1) * (ans + 1) <= x ? ans + 1 : ans;
    }
}
```
</details>
<details>
    <summary>袖珍计算器算法Python3</summary>

```Python [sol1-Python3]
class Solution:
    def mySqrt(self, x: int) -> int:
        if x == 0:
            return 0
        ans = int(math.exp(0.5 * math.log(x)))
        return ans + 1 if (ans + 1) ** 2 <= x else ans
```
</details>
<details>
    <summary>袖珍计算器算法Golang</summary>

```golang [sol1-Golang]
func mySqrt(x int) int {
    if x == 0 {
        return 0
    }
    ans := int(math.Exp(0.5 * math.Log(float64(x))))
    if (ans + 1) * (ans + 1) <= x {
        return ans + 1
    }
    return ans
}
```
</details>


**复杂度分析**

* 时间复杂度：*O(1)*，由于内置的 `exp` 函数与 `log` 函数一般都很快，我们在这里将其复杂度视为 *O(1)*。

* 空间复杂度：*O(1)*。

# 方法二：二分查找

由于 *x* 平方根的整数部分 ans 是**满足 k^2<=x  的最大 *k* 值**，因此我们可以对 *k* 进行二分查找，从而得到答案。

二分查找的下界为 *0*，上界可以粗略地设定为 *x*。在二分查找的每一步中，我们只需要比较中间元素 mid 的平方与 *x* 的大小关系，并通过比较的结果调整上下界的范围。由于我们所有的运算都是整数运算，不会存在误差，因此在得到最终的答案 ans 后，也就不需要再去尝试 ans+1 了。

<details>
    <summary>二分查找C++</summary>


```C++ [sol2-C++]
class Solution {
public:
    int mySqrt(int x) {
        int l = 0, r = x, ans = -1;
        while (l <= r) {
            int mid = l + (r - l) / 2;
            if ((long long)mid * mid <= x) {
                ans = mid;
                l = mid + 1;
            } else {
                r = mid - 1;
            }
        }
        return ans;
    }
};
```
</details>
<details>
    <summary>二分查找Java</summary>


```Java [sol2-Java]
class Solution {
    public int mySqrt(int x) {
        int l = 0, r = x, ans = -1;
        while (l <= r) {
            int mid = l + (r - l) / 2;
            if ((long) mid * mid <= x) {
                ans = mid;
                l = mid + 1;
            } else {
                r = mid - 1;
            }
        }
        return ans;
    }
}
```
</details>
<details>
    <summary>二分查找Python3</summary>


```Python [sol2-Python3]
class Solution:
    def mySqrt(self, x: int) -> int:
        l, r, ans = 0, x, -1
        while l <= r:
            mid = (l + r) // 2
            if mid * mid <= x:
                ans = mid
                l = mid + 1
            else:
                r = mid - 1
        return ans
```
</details>
<details>
    <summary>二分查找Golang</summary>


```golang [sol2-Golang]
func mySqrt(x int) int {
    l, r := 0, x
    ans := -1
    for l <= r {
        mid := l + (r - l) / 2
        if mid * mid <= x {
            ans = mid
            l = mid + 1
        } else {
            r = mid - 1
        }
    }
    return ans
}
```
</details>

**复杂度分析**

* 时间复杂度：O(logx)，即为二分查找需要的次数。

* 空间复杂度：*O(1)*。

# 方法三：牛顿迭代

**思路**

[牛顿迭代法](https://baike.baidu.com/item/%E7%89%9B%E9%A1%BF%E8%BF%AD%E4%BB%A3%E6%B3%95)是一种可以用来快速求解函数零点的方法。参见[^2]




[^1]:https://leetcode-cn.com/problems/sqrtx/solution/er-fen-cha-zhao-niu-dun-fa-python-dai-ma-by-liweiw/

[^2]:https://leetcode-cn.com/problems/sqrtx/solution/x-de-ping-fang-gen-by-leetcode-solution/