---
title: 剑指 Offer II 079. 所有子集

date: 2021-12-12 11:36:12  

categories: 2021年12月

tags: [LeetCode,Array,Backtracking,Bit Manipulation]

---

 

给定一个整数数组 nums ，数组中的元素 互不相同 。返回该数组所有可能的子集（幂集）。

解集 不能 包含重复的子集。你可以按 任意顺序 返回解集。

<!-- more -->

[TOC]


示例 1：

    输入：nums = [1,2,3]
    输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
示例 2：

    输入：nums = [0]
    输出：[[],[0]]


提示：

    1 <= nums.length <= 10
    -10 <= nums[i] <= 10
    nums 中的所有元素 互不相同


注意：本题与主站 [78.子集](https://leetcode-cn.com/problems/subsets/)题相同。

# 二进制枚举

<details>
    <summary>Java</summary>

```
class Solution {
    public List<List<Integer>> subsets(int[] nums) 
    {
        int n = nums.length;
        List<List<Integer>> res = new ArrayList<>();

        for (int state = 0; state < (1 << n); state ++)
        {
            List<Integer> cur = new ArrayList<>();
            for (int i = 0; i < n; i ++)
            {
                if (((state >> i) & 1) == 1)
                {
                    cur.add(nums[i]);
                }
            }
            res.add(cur);
        }

        return res;
    }
}

```
</details>
<details>
    <summary>C++</summary>

```
class Solution 
{
public:
    vector<vector<int>> subsets(vector<int>& nums) 
    {
        int n = nums.size();
        vector<vector<int>> res;
        for (int state = 0; state < (1 << n); state ++)
        {
            vector<int> cur;
            for (int i = 0; i < n; i ++)
            {
                if ((state >> i) & 1)
                {
                    cur.push_back(nums[i]);
                }
            }
            res.push_back(cur);
        }

        return res;
    }
};

```
</details>
<details>
    <summary>python</summary>

```
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        res = []
        for state in range(0, 1 << n):
            cur = []
            for i in range(n):
                if (state >> i) & 1:
                    cur.append(nums[i])
            res.append(cur)
        return res

```
</details>


# 深度优先遍历

<details>
    <summary>Java</summary>

```
class Solution 
{
    int [] nums;
    int n;
    List<List<Integer>> res = new ArrayList<>();

    public List<List<Integer>> subsets(int[] nums) 
    {
        this.nums = nums;
        n = nums.length;
        List<Integer> tmp = new ArrayList<>();
        dfs(0, tmp);
        return res;
    }

    public void dfs(int idx, List<Integer> path)
    {
        if (idx == n)
        {
            res.add(path);
            return ;
        }
        List<Integer> path1 = new ArrayList<>();     path1.addAll(path);
        dfs(idx + 1, path1);
        path.add(nums[idx]);
        List<Integer> path2 = new ArrayList<>();     path2.addAll(path);
        dfs(idx + 1, path2);
    }
}


```
</details>
<details>
    <summary>C++</summary>

```
class Solution 
{
public:
    vector<int> nums;
    int n;
    vector<vector<int>> res;

    vector<vector<int>> subsets(vector<int>& nums) 
    {
        this->nums = nums;
        n = nums.size();
        dfs(0, vector<int>{});
        return res;
    }

    void dfs(int idx, vector<int> path)
    {
        if (idx == n)
        {
            res.push_back(path);
            return ;
        }
        dfs(idx + 1, path);
        path.push_back(nums[idx]);
        dfs(idx + 1, path);
    }
};

```
</details>
<details>
    <summary>python</summary>

```
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        res = []

        def dfs(idx: int, path: List[int]) -> None:
            nonlocal res
            if idx == n:
                res.append(path)
                return 
            dfs(idx + 1, path + [nums[idx]])
            dfs(idx + 1, path)

        dfs(0, [])
        return res

```
</details>


# 回溯

<details>
    <summary>Java</summary>

```


class Solution 
{
    int [] nums;
    int n;
    List<Integer> path = new ArrayList<>();
    List<List<Integer>> res = new ArrayList<>();

    public List<List<Integer>> subsets(int[] nums) 
    {
        this.nums = nums;
        this.n = nums.length;
        backtrace(0);
        return res;
    }

    public void backtrace(int idx)
    {
        if (idx == n)
        {
            res.add(new ArrayList<>(path));
            return ;
        }

        backtrace(idx + 1);

        path.add(nums[idx]);
        backtrace(idx + 1);
        path.remove(path.size() - 1);
    }
}

```
</details>
<details>
    <summary>C++</summary>

```

class Solution 
{
public:
    vector<int> nums;
    int n;
    vector<int> path;
    vector<vector<int>> res;

    vector<vector<int>> subsets(vector<int>& nums) 
    {
        this->nums = nums;
        this->n = nums.size();
        backtrace(0);
        return res;
    }

    void backtrace(int idx)
    {
        if (idx == n)
        {
            res.push_back(path);
            return ;
        }

        backtrace(idx + 1);

        path.push_back(nums[idx]);
        backtrace(idx + 1);
        path.pop_back();
    }
};

```
</details>
<details>
    <summary>python</summary>

```
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        n = len(nums)
        res = []

        path = []
        def backtrace(idx: int) -> None:
            nonlocal res
            nonlocal path
            if idx == n:
                res.append(path[:])
                return 
                
            backtrace(idx + 1)

            path.append(nums[idx])
            backtrace(idx + 1)
            path.pop()

        backtrace(0)
        return res

```
</details>