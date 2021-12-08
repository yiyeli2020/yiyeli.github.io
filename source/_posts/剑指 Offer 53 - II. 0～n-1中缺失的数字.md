---
title: 剑指 Offer 53 - II. 0～n-1中缺失的数字

date: 2021-12-08 15:12:12

categories: 2021年12月

tags: [Leetcode, Bit Manipulation, Array, Hash Table, Math, Binary Search]

---

一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。


<!-- more -->


[TOC]



示例 1:
    
    输入: [0,1,3]
    输出: 2
示例 2:
    
    输入: [0,1,2,3,4,5,6,7,9]
    输出: 8


限制：
    
    1 <= 数组长度 <= 10000

此题由[268. 丢失的数字](https://leetcode-cn.com/problems/missing-number/solution/diu-shi-de-shu-zi-by-leetcode-solution-naow/)改编而来，但题干部分描述的不太清楚，多出来的数字应该是n

解法可参见[268. 丢失的数字](https://leetcode-cn.com/problems/missing-number/solution/diu-shi-de-shu-zi-by-leetcode-solution-naow/)

由于该题不同之处在于是有序的，所以还可以使用二分法

# 二分法解题思路[^1]：

- 排序数组中的搜索问题，首先想到 **二分法** 解决。
- 根据题意，数组可以按照以下规则划分为两部分。
  - **左子数组：** *nums[i] = i* ；
  - **右子数组：** *nums[i] != i*  ；
- 缺失的数字等于 **“右子数组的首位元素”** 对应的索引；因此考虑使用二分法查找 “右子数组的首位元素” 。

# 复杂度分析：

- **时间复杂度 *O(log N)*：** 二分法为对数级别复杂度。
- **空间复杂度 *O(1)*：** 几个变量使用常数大小的额外空间。

# 代码：

<details>
    <summary>Python</summary>
    
```Python []
class Solution:
    def missingNumber(self, nums: List[int]) -> int:
        i, j = 0, len(nums) - 1
        while i <= j:
            m = (i + j) // 2
            if nums[m] == m: i = m + 1
            else: j = m - 1
        return i
```
</details>
<details>
    <summary>Java</summary>
    
```Java []
class Solution {
    public int missingNumber(int[] nums) {
        int i = 0, j = nums.length - 1;
        while(i <= j) {
            int m = (i + j) / 2;
            if(nums[m] == m) i = m + 1;
            else j = m - 1;
        }
        return i;
    }
}
```

</details>

[^1]: https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/solution/mian-shi-ti-53-ii-0n-1zhong-que-shi-de-shu-zi-er-f/