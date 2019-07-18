---
title: Permutations全排列问题总结
date: 2018-10-11 10:33:14
categories: 2018年10月
tags: [LeetCode,Backtracking]

---
 

总结几个常见的全排列问题
46.Permutations 全排列
47.PermutationsII 全排列II
31.Next Permutations 下一个排列
60.Permutation Sequence 第k个排列
266.Palindrome Permutation 回文排列
267.Palindrome PermutationII 回文排列II

<!-- more -->


## 46.Permutations 全排列

给定一个没有重复数字的序列，返回其所有可能的全排列。

示例:

	输入: [1,2,3]
	输出:
	[
	  [1,2,3],
	  [1,3,2],
	  [2,1,3],
	  [2,3,1],
	  [3,1,2],
	  [3,2,1]
	]


### 回溯思路
回溯的写法，每次交换nums里面的两个数字，经过回溯可以生成所有的排列情况

	class Solution {
	public:
	    vector<vector<int>> permute(vector<int>& nums) {
	        vector<vector<int>> result;
	        permuteRecursive(nums,0,result);
	        return result;
	    }
	    void output(vector<vector<int>> result){
	        cout<<"start to print"<<endl;
	        for(int i=0;i<result.size();i++){
	            for(int j=0;j<result[i].size();j++){
	                cout<<result[i][j];
	            }cout<<endl;
	        }
	        cout<<"end to print"<<endl;
	    }
	    void permuteRecursive(vector<int> &num,int begin,vector<vector<int>> &result){
	        // cout<<"begin="<<begin<<endl;
	        // cout<<"   num=";
	        // for(int i=0;i<num.size();i++) cout<<num[i]<<" ";
	        // cout<<endl;
	        // output(result);
	        if(begin>=num.size()){
	            result.push_back(num);
	            return ;
	        }
	        for(int i=begin;i<num.size();i++){
	            swap(num[begin],num[i]);
	            permuteRecursive(num,begin+1,result);
	            swap(num[begin],num[i]);
	        }
	    }
	};

### DFS思路

这种方法是CareerCup书上的方法，也挺不错的，这道题是思想是这样的：

当n=1时，数组中只有一个数a1，其全排列只有一种，即为a1

当n=2时，数组中此时有a1a2，其全排列有两种，a1a2和a2a1，那么此时我们考虑和上面那种情况的关系，我们发现，其实就是在a1的前后两个位置分别加入了a2

当n=3时，数组中有a1a2a3，此时全排列有六种，分别为a1a2a3, a1a3a2, a2a1a3, a2a3a1, a3a1a2, 和 a3a2a1。那么根据上面的结论，实际上是在a1a2和a2a1的基础上在不同的位置上加入a3而得到的。

_ a1 _ a2 _ : a3a1a2, a1a3a2, a1a2a3

_ a2 _ a1 _ : a3a2a1, a2a3a1, a2a1a3


	class Solution {
	public:
	vector<vector<int> > permute(vector<int> &num) {
	    vector<vector<int> > ans;
	    dfs(num, ans);
	    return ans;
	}

	void dfs(vector<int> &num, vector<vector<int>> &ans) {
	    if (num.size() == 1) {
	        vector<int> tmp(num.begin(), num.end());
	        ans.push_back(tmp);
	        return;
	    }

	    vector<vector<int> > ans1;
	    vector<int> num1(num.begin()+1, num.end());
	    dfs(num1, ans);

	    for(int i = 0; i < ans.size(); ++i) {
	        for(int j = 0; j <= ans[i].size(); ++j) {
	            vector<int> tmp = ans[i];
	            tmp.insert(tmp.begin()+j, num[0]);
	            ans1.push_back(tmp);
	        }
	    }

	    ans = ans1;
	}
	};

## 47.PermutationsII 全排列II

给定一个可包含重复数字的序列，返回所有不重复的全排列。

示例:

	输入: [1,1,2]
	输出:
	[
	  [1,1,2],
	  [1,2,1],
	  [2,1,1]
	]

### 思路：

  这里我们先考虑一下，它与上题唯一的不同在于：在DFS函数中，做循环遍历时，如果与当前元素相同的一个元素已经被取用过，则要跳过所有值相同的元素。
  举个例子：对于序列<1,1,2,3>。在DFS首遍历时，1 作为首元素被加到list中，并进行后续元素的添加；那么，当DFS跑完第一个分支，遍历到1 (第二个)时，这个1 不再作为首元素添加到list中，因为1 作为首元素的情况已经在第一个分支中考虑过了。
  为了实现这一剪枝思路，有了如下的解题算法。



解题算法：

  1. 先对给定的序列nums进行排序，使得大小相同的元素排在一起。
  2. 新建一个used数组，大小与nums相同，用来标记在本次DFS读取中，位置i的元素是否已经被添加到list中了。
  3. 根据思路可知，我们选择跳过一个数，当且仅当这个数与前一个数相等，并且前一个数未被添加到list中。

	class Solution {
	public:
	    void recursion(vector<int> num, int i, int j, vector<vector<int> > &res) {
	        if (i == j-1) {
	            res.push_back(num);
	            return;
	        }
	        for (int k = i; k < j; k++) {
	            if (i != k && num[i] == num[k]) continue;
	            swap(num[i], num[k]);
	            recursion(num, i+1, j, res);
	        }
	    }
	    vector<vector<int> > permuteUnique(vector<int> &num) {
	        sort(num.begin(), num.end());
	        vector<vector<int> >res;
	        recursion(num, 0, num.size(), res);
	        return res;
	    }
	};


## 31.Next Permutations 下一个排列
实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。

如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。

必须原地修改，只允许使用额外常数空间。

以下是一些例子，输入位于左侧列，其相应输出位于右侧列。

	1,2,3 → 1,3,2
	3,2,1 → 1,2,3
	1,1,5 → 1,5,1


### 思路

  这里，先考虑一个序列的最大最小情况。当一个序列为非递减序列时，它必然是该组数的最小的排列数；同理，当一个序列为非递增序列时，它必然是该组数的最大的排列数。  


  由此，我们可以知道，本题的关键即是求出数组末尾的最长的非递增子序列。
  不妨假设在数组nums中，nums[k+1]…nums[n]均满足前一个元素大于等于后一个元素，即这一子序列非递增。
  那么，我们要做的，就是把nums[k]与其后序列中稍大于nums[k]的数交换，接着再逆序nums[k+1]…nums[n]即可。

	class Solution {
	public:
	    void nextPermutation(vector<int>& nums) {
	    	int n = nums.size(), k, l;
	    	for (k = n - 2; k >= 0; k--) {
	            if (nums[k] < nums[k + 1]) {
	                break;
	            }
	        }
	    	if (k < 0) {
	    	    reverse(nums.begin(), nums.end());
	    	} else {
	    	    for (l = n - 1; l > k; l--) {
	                if (nums[l] > nums[k]) {
	                    break;
	                }
	            }
	    	    swap(nums[k], nums[l]);
	    	    reverse(nums.begin() + k + 1, nums.end());
	        }
	    }
	};

## 60.Permutation Sequence 第k个排列

给出集合 [1,2,3,…,n]，其所有元素共有 n! 种排列。

按大小顺序列出所有排列情况，并一一标记，当 n = 3 时, 所有排列如下：

	"123"
	"132"
	"213"
	"231"
	"312"
	"321"
给定 n 和 k，返回第 k 个排列。

说明：

	给定 n 的范围是 [1, 9]。
	给定 k 的范围是[1,  n!]。
示例 1:

	输入: n = 3, k = 3
	输出: "213"
示例 2:

	输入: n = 4, k = 9
	输出: "2314"

### 思路：

  这里我们先考虑一个特殊情况，当n=4时，序列为[1,2,3,4]，有以下几种情况：
  “1+(2,3,4)的全排列”
  “2+(1,3,4)的全排列”
  “3+(1,2,4)的全排列”
  “4+(1,2,3)的全排列”
  我们已经知道，对于n个数的全排列，有n!种情况。所以，3个数的全排列就有6种情况。
  
  如果我们这里给定的k为14，那么它将会出现在：
    “3+(1,2,4)的全排列”
  这一情况中。

  我们可以程式化地得到这个结果：取k=13(从0开始计数)，(n-1)!=3!=6，k/(n-1)!=2，而3在有序序列[1,2,3,4]中的索引就是2。
  同理，我们继续计算，新的k=13%6=1，新的n=3，那么1/(n-1)!=2/2=0。在序列[1,2,4]中，索引0的数是1。那么，此时的字符串为”31”。
  继续迭代，新的k=1%2=1，新的n=2，那么k/(n-1)!=1/1=1。在序列[2,4]中，索引为1的数是4。那么，此时的字符串为”314”。最后在串尾添上仅剩的2，可以得到字符串”3142”。
  经过验算，此串确实是序列[1,2,3,4]的全排列数中第14大的序列。



解题算法：

  1. 创建一个长度为n 的数组array，存放对应下标n的阶乘值。
  2. 再新建一个长度为n 的数组nums，初始值为nums[i]=i+1，用来存放待选的字符序列。
  3. 将得到的k减1后，开始迭代。迭代的规则是：迭代n次，每次选nums数组中下标为k/(n-1)!的数放在字符串的末尾，新的k=k%(n-1)!，新的n=n-1。  
  4. 最后，返回得到的字符串。

	class Solution {
	public:
	    string getPermutation(int n, int k) {
	        // initialize a dictionary that stores 1, 2, ..., n. This string will store the permutation.
	        string dict(n, 0);
	        iota(dict.begin(), dict.end(), '1');

	        // build up a look-up table, which stores (n-1)!, (n-2)!, ..., 1!, 0!
	        vector<int> fract(n, 1);
	        for (int idx = n - 3; idx >= 0; --idx) {
	            fract[idx] = fract[idx + 1] * (n - 1 - idx);
	        }

	        // let k be zero base
	        --k;

	        // the main part.
	        string ret(n, 0);
	        for (int idx = 0; idx < n; ++idx) {
	            int select = k / fract[idx];
	            k %= fract[idx];
	            ret[idx] = dict[select];
	            dict.erase(next(dict.begin(), select)); // note that it is an O(n) operation
	        }
	        return ret;
	    }
	};
