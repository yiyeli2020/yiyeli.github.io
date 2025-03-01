---
title: 剑指 Offer II 088. 爬楼梯的最少成本

date: 2021-12-16 16:12:12  

categories: 2021年12月

tags: [LeetCode, Array, Dynamic Programming]

---

数组的每个下标作为一个阶梯，第 i 个阶梯对应着一个非负数的体力花费值 cost[i]（下标从 0 开始）。

每当你爬上一个阶梯你都要花费对应的体力值，一旦支付了相应的体力值，你就可以选择向上爬一个阶梯或者爬两个阶梯。

请你找出达到楼层顶部的最低花费。在开始时，你可以选择从下标为 0 或 1 的元素作为初始阶梯。


<!-- more -->

[TOC]



示例 1：

    输入：cost = [10, 15, 20]
    输出：15
    解释：最低花费是从 cost[1] 开始，然后走两步即可到阶梯顶，一共花费 15 。
 示例 2：

    输入：cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1]
    输出：6
    解释：最低花费方式是从 cost[0] 开始，逐个经过那些 1 ，跳过 cost[3] ，一共花费 6 。


提示：

    cost 的长度范围是 [2, 1000]。
    cost[i] 将会是一个整型数据，范围为 [0, 999] 。

注意：本题与[746. 使用最小花费爬楼梯](https://leetcode-cn.com/problems/min-cost-climbing-stairs/)相同： 

# 方法一：动态规划[^1]

<details>
    <summary>Java</summary>

```Java [sol1-Java]
class Solution {
    public int minCostClimbingStairs(int[] cost) {
        int n = cost.length;
        int[] dp = new int[n + 1];
        dp[0] = dp[1] = 0;
        for (int i = 2; i <= n; i++) {
            dp[i] = Math.min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);
        }
        return dp[n];
    }
}
```
</details>
<details>
    <summary>C#</summary>
    
    
```C# [sol1-C#]
public class Solution {
    public int MinCostClimbingStairs(int[] cost) {
        int n = cost.Length;
        int[] dp = new int[n + 1];
        dp[0] = dp[1] = 0;
        for (int i = 2; i <= n; i++) {
            dp[i] = Math.Min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);
        }
        return dp[n];
    }
}
```
</details>
<details>
    <summary>JavaScript</summary>
    
    
```JavaScript [sol1-JavaScript]
var minCostClimbingStairs = function(cost) {
    const n = cost.length;
    const dp = new Array(n + 1);
    dp[0] = dp[1] = 0;
    for (let i = 2; i <= n; i++) {
        dp[i] = Math.min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);
    }
    return dp[n];
};
```
</details>
<details>
    <summary>C++</summary>
    
    
```C++ [sol1-C++]
class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        int n = cost.size();
        vector<int> dp(n + 1);
        dp[0] = dp[1] = 0;
        for (int i = 2; i <= n; i++) {
            dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);
        }
        return dp[n];
    }
};
```
</details>
<details>
    <summary>Golang</summary>
    
    
```Go [sol1-Golang]
func minCostClimbingStairs(cost []int) int {
    n := len(cost)
    dp := make([]int, n+1)
    for i := 2; i <= n; i++ {
        dp[i] = min(dp[i-1]+cost[i-1], dp[i-2]+cost[i-2])
    }
    return dp[n]
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```
</details>
<details>
    <summary>Python3</summary>
    
    
```Python [sol1-Python3]
class Solution:
    def minCostClimbingStairs(self, cost: List[int]) -> int:
        n = len(cost)
        dp = [0] * (n + 1)
        for i in range(2, n + 1):
            dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2])
        return dp[n]
```
</details>
<details>
    <summary>C</summary>
    
    
```C [sol1-C]
int minCostClimbingStairs(int* cost, int costSize) {
    int dp[costSize + 1];
    dp[0] = dp[1] = 0;
    for (int i = 2; i <= costSize; i++) {
        dp[i] = fmin(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);
    }
    return dp[costSize];
}
```
</details>

上述代码的时间复杂度和空间复杂度都是 *O(n)* 。可以使用滚动数组的思想，将空间复杂度优化到 *O(1)*。

<details>
    <summary>Java</summary>
    
    
```Java [sol2-Java]
class Solution {
    public int minCostClimbingStairs(int[] cost) {
        int n = cost.length;
        int prev = 0, curr = 0;
        for (int i = 2; i <= n; i++) {
            int next = Math.min(curr + cost[i - 1], prev + cost[i - 2]);
            prev = curr;
            curr = next;
        }
        return curr;
    }
}
```
</details>
<details>
    <summary>JavaScript</summary>
    
    
```JavaScript [sol2-JavaScript]
var minCostClimbingStairs = function(cost) {
    const n = cost.length;
    let prev = 0, curr = 0;
    for (let i = 2; i <= n; i++) {
        let next = Math.min(curr + cost[i - 1], prev + cost[i - 2]);
        prev = curr;
        curr = next;
    }
    return curr;
};
```
</details>
<details>
    <summary>C++</summary>
    
    
```C++ [sol2-C++]
class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        int n = cost.size();
        int prev = 0, curr = 0;
        for (int i = 2; i <= n; i++) {
            int next = min(curr + cost[i - 1], prev + cost[i - 2]);
            prev = curr;
            curr = next;
        }
        return curr;
    }
};
```
</details>
<details>
    <summary>C#</summary>
    
    
```C# [sol2-C#]
public class Solution {
    public int MinCostClimbingStairs(int[] cost) {
        int n = cost.Length;
        int prev = 0, curr = 0;
        for (int i = 2; i <= n; i++) {
            int next = Math.Min(curr + cost[i - 1], prev + cost[i - 2]);
            prev = curr;
            curr = next;
        }
        return curr;
    }
}
```
</details>
<details>
    <summary>Golang</summary>
    
    
```Go [sol2-Golang]
func minCostClimbingStairs(cost []int) int {
    n := len(cost)
    pre, cur := 0, 0
    for i := 2; i <= n; i++ {
        pre, cur = cur, min(cur+cost[i-1], pre+cost[i-2])
    }
    return cur
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```
</details>
<details>
    <summary>Python3</summary>
    
    
```Python [sol2-Python3]
class Solution:
    def minCostClimbingStairs(self, cost: List[int]) -> int:
        n = len(cost)
        prev = curr = 0
        for i in range(2, n + 1):
            nxt = min(curr + cost[i - 1], prev + cost[i - 2])
            prev, curr = curr, nxt
        return curr
```
</details>
<details>
    <summary>C</summary>
    
    
```C [sol2-C]
int minCostClimbingStairs(int* cost, int costSize) {
    int prev = 0, curr = 0;
    for (int i = 2; i <= costSize; i++) {
        int next = fmin(curr + cost[i - 1], prev + cost[i - 2]);
        prev = curr;
        curr = next;
    }
    return curr;
}
```
</details>

**复杂度分析**

- 时间复杂度：*O(n)*，其中 *n* 是数组cost 的长度。需要依次计算每个dp 值，每个值的计算需要常数时间，因此总时间复杂度是 *O(n)*。

- 空间复杂度：*O(1)*。使用滚动数组的思想，只需要使用有限的额外空间。


[^1]:https://leetcode-cn.com/problems/min-cost-climbing-stairs/solution/shi-yong-zui-xiao-hua-fei-pa-lou-ti-by-l-ncf8/