---

title: 剑指 Offer II 093. 最长斐波那契数列

date: 2021-12-14 20:37:12

categories: 2021年12月

tags: [LeetCode, Array, Dynamic Programming, Hash Table]


---
 
给定一个严格递增的正整数数组形成序列 arr ，找到 arr 中最长的斐波那契式的子序列的长度。如果一个不存在，返回0 。

 
<!-- more -->

[TOC]

如果序列 X_1, X_2, ..., X_n 满足下列条件，就说它是 斐波那契式 的：

n >= 3
对于所有 i + 2 <= n，都有 X_i + X_{i+1} = X_{i+2}
给定一个严格递增的正整数数组形成序列 arr ，找到 arr 中最长的斐波那契式的子序列的长度。如果一个不存在，返回0 。

（回想一下，子序列是从原序列 arr 中派生出来的，它从 arr 中删掉任意数量的元素（也可以不删），而不改变其余元素的顺序。例如， [3, 5, 8] 是 [3, 4, 5, 6, 7, 8] 的一个子序列）



示例 1：
    
    输入: arr = [1,2,3,4,5,6,7,8]
    输出: 5
    解释: 最长的斐波那契式子序列为 [1,2,3,5,8] 。
示例 2：
    
    输入: arr = [1,3,7,11,12,14,18]
    输出: 3
    解释: 最长的斐波那契式子序列有 [1,11,12]、[3,11,14] 以及 [7,11,18] 。


提示：
    
    3 <= arr.length <= 1000
    1 <= arr[i] < arr[i + 1] <= 10^9

本题与[873. 最长的斐波那契子序列的长度](https://leetcode-cn.com/problems/length-of-longest-fibonacc//i-subsequence/ )相同： 

# 方法一：使用 Set 的暴力法

**思路**

每个斐波那契式的子序列都依靠两个相邻项来确定下一个预期项。例如，对于 `2, 5`，我们所期望的子序列必定以 `7, 12, 19, 31` 等继续。

我们可以使用 `Set` 结构来快速确定下一项是否在数组 `A` 中。由于这些项的值以指数形式增长，最大值 <=10^9 的斐波那契式的子序列最多有 43 项。

**算法**

对于每个起始对 `A[i], A[j]`，我们保持下一个预期值 `y = A[i] + A[j]` 和此前看到的最大值 `x = A[j]`。如果 `y` 在数组中，我们可以更新这些值 `(x, y) -> (y, x+y)`。

此外，由于子序列的长度大于等于 3 只能是斐波那契式的，所以我们必须在最后进行检查 `ans >= 3 ? ans : 0`。

<details>
    <summary>cpp</summary>
    
```cpp [RGECz9nf-C++]
class Solution {
public:
    int lenLongestFibSubseq(vector<int>& A) {
        int N = A.size();
        unordered_set<int> S(A.begin(), A.end());

        int ans = 0;
        for (int i = 0; i < N; ++i)
            for (int j = i+1; j < N; ++j) {
                /* With the starting pair (A[i], A[j]),
                 * y represents the future expected value in
                 * the fibonacci subsequence, and x represents
                 * the most current value found. */
                int x = A[j], y = A[i] + A[j];
                int length = 2;
                while (S.find(y) != S.end()) {
                    int z = x + y;
                    x = y;
                    y = z;
                    ans = max(ans, ++length);
                }
            }

        return ans >= 3 ? ans : 0;
    }
};
```
</details>
<details>
    <summary>Java</summary>
    
```java [RGECz9nf-Java]
class Solution {
    public int lenLongestFibSubseq(int[] A) {
        int N = A.length;
        Set<Integer> S = new HashSet();
        for (int x: A) S.add(x);

        int ans = 0;
        for (int i = 0; i < N; ++i)
            for (int j = i+1; j < N; ++j) {
                /* With the starting pair (A[i], A[j]),
                 * y represents the future expected value in
                 * the fibonacci subsequence, and x represents
                 * the most current value found. */
                int x = A[j], y = A[i] + A[j];
                int length = 2;
                while (S.contains(y)) {
                    // x, y -> y, x+y
                    int tmp = y;
                    y += x;
                    x = tmp;
                    ans = Math.max(ans, ++length);
                }
            }

        return ans >= 3 ? ans : 0;
    }
}
```
</details>
<details>
    <summary>Python</summary>
    
```python [RGECz9nf-Python]
class Solution(object):
    def lenLongestFibSubseq(self, A):
        S = set(A)
        ans = 0
        for i in xrange(len(A)):
            for j in xrange(i+1, len(A)):
                """
                With the starting pair (A[i], A[j]),
                y represents the future expected value in
                the fibonacci subsequence, and x represents
                the most current value found.
                """
                x, y = A[j], A[i] + A[j]
                length = 2
                while y in S:
                    x, y = y, x + y
                    length += 1
                ans = max(ans, length)
        return ans if ans >= 3 else 0
```


</details>




---

# 方法二：动态规划

**思路**

将斐波那契式的子序列中的两个连续项 `A[i], A[j]` 视为单个结点 `(i, j)`，整个子序列是这些连续结点之间的路径。

例如，对于斐波那契式的子序列 `(A[1] = 2, A[2] = 3, A[4] = 5, A[7] = 8, A[10] = 13)`，结点之间的路径为 `(1, 2) <-> (2, 4) <-> (4, 7) <-> (7, 10)`。

这样做的动机是，只有当 `A[i] + A[j] == A[k]` 时，两结点 `(i, j)` 和 `(j, k)` 才是连通的，我们需要这些信息才能知道这一连通。现在我们得到一个类似于 *最长上升子序列* 的问题。

**算法**

设 `longest[i, j]` 是结束在 `[i, j]` 的最长路径。那么 如果 `(i, j)` 和 `(j, k)` 是连通的， `longest[j, k] = longest[i, j] + 1`。

由于 `i` 由 `A.index(A[k] - A[j])` 唯一确定，所以这是有效的：我们在 `i` 潜在时检查每组 `j < k`，并相应地更新 `longest[j, k]`。


<details>
    <summary>cpp</summary>
    
```cpp [UNSjQ9SB-C++]
class Solution {
public:
    int lenLongestFibSubseq(vector<int>& A) {
        int N = A.size();
        unordered_map<int, int> index;
        for (int i = 0; i < N; ++i)
            index[A[i]] = i;

        unordered_map<int, int> longest;
        int ans = 0;
        for (int k = 0; k < N; ++k)
            for (int j = 0; j < k; ++j) {
                if (A[k] - A[j] < A[j] && index.count(A[k] - A[j])) {
                    int i = index[A[k] - A[j]];
                    longest[j * N + k] = longest[i * N + j] + 1;
                    ans = max(ans, longest[j * N + k] + 2);
                }
            }

        return ans >= 3 ? ans : 0;
    }
};
```
</details>
<details>
    <summary>Java</summary>
    
```java [UNSjQ9SB-Java]
class Solution {
    public int lenLongestFibSubseq(int[] A) {
        int N = A.length;
        Map<Integer, Integer> index = new HashMap();
        for (int i = 0; i < N; ++i)
            index.put(A[i], i);

        Map<Integer, Integer> longest = new HashMap();
        int ans = 0;

        for (int k = 0; k < N; ++k)
            for (int j = 0; j < k; ++j) {
                int i = index.getOrDefault(A[k] - A[j], -1);
                if (i >= 0 && i < j) {
                    // Encoding tuple (i, j) as integer (i * N + j)
                    int cand = longest.getOrDefault(i * N + j, 2) + 1;
                    longest.put(j * N + k, cand);
                    ans = Math.max(ans, cand);
                }
            }

        return ans >= 3 ? ans : 0;
    }
}
```
</details>
<details>
    <summary>Python</summary>
    
```python [UNSjQ9SB-Python]
class Solution(object):
    def lenLongestFibSubseq(self, A):
        index = {x: i for i, x in enumerate(A)}
        longest = collections.defaultdict(lambda: 2)

        ans = 0
        for k, z in enumerate(A):
            for j in xrange(k):
                i = index.get(z - A[j], None)
                if i is not None and i < j:
                    cand = longest[j, k] = longest[i, j] + 1
                    ans = max(ans, cand)

        return ans if ans >= 3 else 0
```

</details>

# 动态规划简明版

上面这个状态转移比较难以理解

以下是解释

下面是`Time Limit Error`代码

<details>
    <summary>cpp</summary>
    
```cpp
	for (int i = 0; i < n; i++) {
		for (int j = i + 1; j < n; j++) {
			for (int k = i - 1; k >= 0; k--) {
				if (A[k] + A[i] == A[j]) {
					dp[i][j] = max(dp[i][j], dp[k][i] + 1);
				}
			}
			MAX = max(MAX, dp[i][j]);
		}
	}
```

</details>

## 代码

<details>
    <summary>cpp</summary>
    
```cpp
class Solution {
public:
    int lenLongestFibSubseq(vector<int>& A) {
        int n = A.size();
	if (n == 0) { return 0; }
	unordered_map<int, int> intMap;
	for (int i = 0; i < n; i++) {
		intMap[A[i]] = i;
	}
	vector<vector<int>> dp(n, vector<int>(n, 0));
	for (int i = 0; i < n; i++) {
		for (int j = i + 1; j < n; j++) {
			dp[i][j] = 2;
		}
	}
	int MAX = 0;

	for (int i = 0; i < n; i++) {
		for (int j = i + 1; j < n; j++) {
			int diff = A[j] - A[i];
			if (intMap.count(diff)) {
				int index = intMap[diff];
				if (index < i) {
					dp[i][j] = max(dp[i][j], dp[index][i] + 1);
				}
			}
			MAX = max(MAX, dp[i][j]);
		}
	}

	return MAX > 2? MAX : 0;
    }
};
```

</details>


<details>
    <summary>Java</summary>
    
```Java
class Solution {
        //dp[i][j]: 以A[i],A[j] 结尾的最长的长度-2
        //如果存在A[k]=A[j]-A[i] 且k<i 则 dp[i][j]=dp[k][i]+1
       public int lenLongestFibSubseq(int[] arr) {
        int max = 0;
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < arr.length; i++) {
            map.put(arr[i], i);
        }
        int[][] dp = new int[arr.length][arr.length];
        for (int i = 0; i < arr.length; i++) {
            for (int j = i + 1; j < arr.length; j++) {
                    if (map.containsKey(arr[j]-arr[i])) {
                        int k = map.get(arr[j] - arr[i]);
                        if (k < i) {
                            dp[i][j] = Math.max(dp[i][j], dp[k][i] + 1);
                            max = Math.max(max, dp[i][j] + 2);
                        }
                    }
            }
        }
        return max;
    }
}
```

</details>

如果对 `max = Math.max(max, dp[i][j] + 2);`这一步理解有困难，也可以将dp初始化为全2的矩阵，然后`max = Math.max(max, dp[i][j]);`即可

[^1]:https://leetcode-cn.com/problems/length-of-longest-fibonacci-subsequence/solution/zhuang-tai-ding-yi-hen-shi-zhong-yao-by-christmas_/