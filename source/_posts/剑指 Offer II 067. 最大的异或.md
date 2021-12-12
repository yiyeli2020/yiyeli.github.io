---

title: 剑指 Offer II 067. 最大的异或

date: 2021-12-12 10:12:12

categories: 2021年12月

tags: [Leetcode, Bit Manipulation, Array, Trie, Hash Table]

---


给定一个32位整数num，你可以将一个数位从0变为1。请编写一个程序，找出你能够获得的最长的一串1的长度。


<!-- more -->


[TOC]

给你一个整数数组 nums ，返回 nums[i] XOR nums[j] 的最大运算结果，其中 0 ≤ i ≤ j < n 。

进阶：你可以在 O(n) 的时间解决这个问题吗？

注意：本题与主站 [421. 数组中两个数的最大异或值](https://leetcode-cn.com/problems/maximum-xor-of-two-numbers-in-an-array/)相同




示例 1：
    
    输入：nums = [3,10,5,25,2,8]
    输出：28
    解释：最大运算结果是 5 XOR 25 = 28.
示例 2：

    输入：nums = [0]
    输出：0
示例 3：

    输入：nums = [2,4]
    输出：6
示例 4：
    
    输入：nums = [8,10,2]
    输出：10
示例 5：

    输入：nums = [14,70,53,83,49,91,36,80,92,51,66,70]
    输出：127


提示：

    1 <= nums.length <= 2 * 104
    0 <= nums[i] <= 231 - 1


# 方法一：哈希表[^1]

**思路与算法**

假设我们已经确定了 *x* 最高的若干个二进制位，当前正在确定第 *k* 个二进制位。根据「前言」部分的分析，我们希望第 *k* 个二进制位能够取到 *1*。

**代码**

<details>
    <summary>C++</summary>
    
```C++ [sol1-C++]
class Solution {
private:
    // 最高位的二进制位编号为 30
    static constexpr int HIGH_BIT = 30;

public:
    int findMaximumXOR(vector<int>& nums) {
        int x = 0;
        for (int k = HIGH_BIT; k >= 0; --k) {
            unordered_set<int> seen;
            // 将所有的 pre^k(a_j) 放入哈希表中
            for (int num: nums) {
                // 如果只想保留从最高位开始到第 k 个二进制位为止的部分
                // 只需将其右移 k 位
                seen.insert(num >> k);
            }

            // 目前 x 包含从最高位开始到第 k+1 个二进制位为止的部分
            // 我们将 x 的第 k 个二进制位置为 1，即为 x = x*2+1
            int x_next = x * 2 + 1;
            bool found = false;
            
            // 枚举 i
            for (int num: nums) {
                if (seen.count(x_next ^ (num >> k))) {
                    found = true;
                    break;
                }
            }

            if (found) {
                x = x_next;
            }
            else {
                // 如果没有找到满足等式的 a_i 和 a_j，那么 x 的第 k 个二进制位只能为 0
                // 即为 x = x*2
                x = x_next - 1;
            }
        }
        return x;
    }
};
```
</details>
<details>
    <summary>Java</summary>
    
```Java [sol1-Java]
class Solution {
    // 最高位的二进制位编号为 30
    static final int HIGH_BIT = 30;

    public int findMaximumXOR(int[] nums) {
        int x = 0;
        for (int k = HIGH_BIT; k >= 0; --k) {
            Set<Integer> seen = new HashSet<Integer>();
            // 将所有的 pre^k(a_j) 放入哈希表中
            for (int num : nums) {
                // 如果只想保留从最高位开始到第 k 个二进制位为止的部分
                // 只需将其右移 k 位
                seen.add(num >> k);
            }

            // 目前 x 包含从最高位开始到第 k+1 个二进制位为止的部分
            // 我们将 x 的第 k 个二进制位置为 1，即为 x = x*2+1
            int xNext = x * 2 + 1;
            boolean found = false;
            
            // 枚举 i
            for (int num : nums) {
                if (seen.contains(xNext ^ (num >> k))) {
                    found = true;
                    break;
                }
            }

            if (found) {
                x = xNext;
            } else {
                // 如果没有找到满足等式的 a_i 和 a_j，那么 x 的第 k 个二进制位只能为 0
                // 即为 x = x*2
                x = xNext - 1;
            }
        }
        return x;
    }
}
```
</details>
<details>
    <summary>C#</summary>
 
```C# [sol1-C#]
public class Solution {
    // 最高位的二进制位编号为 30
    const int HIGH_BIT = 30;

    public int FindMaximumXOR(int[] nums) {
        int x = 0;
        for (int k = HIGH_BIT; k >= 0; --k) {
            ISet<int> seen = new HashSet<int>();
            // 将所有的 pre^k(a_j) 放入哈希表中
            foreach (int num in nums) {
                // 如果只想保留从最高位开始到第 k 个二进制位为止的部分
                // 只需将其右移 k 位
                seen.Add(num >> k);
            }

            // 目前 x 包含从最高位开始到第 k+1 个二进制位为止的部分
            // 我们将 x 的第 k 个二进制位置为 1，即为 x = x*2+1
            int xNext = x * 2 + 1;
            bool found = false;
            
            // 枚举 i
            foreach (int num in nums) {
                if (seen.Contains(xNext ^ (num >> k))) {
                    found = true;
                    break;
                }
            }

            if (found) {
                x = xNext;
            } else {
                // 如果没有找到满足等式的 a_i 和 a_j，那么 x 的第 k 个二进制位只能为 0
                // 即为 x = x*2
                x = xNext - 1;
            }
        }
        return x;
    }
}
```
</details>
<details>
    <summary>Python3</summary>
 
```Python [sol1-Python3]
class Solution:
    def findMaximumXOR(self, nums: List[int]) -> int:
        # 最高位的二进制位编号为 30
        HIGH_BIT = 30

        x = 0
        for k in range(HIGH_BIT, -1, -1):
            seen = set()
            # 将所有的 pre^k(a_j) 放入哈希表中
            for num in nums:
                # 如果只想保留从最高位开始到第 k 个二进制位为止的部分
                # 只需将其右移 k 位
                seen.add(num >> k)

            # 目前 x 包含从最高位开始到第 k+1 个二进制位为止的部分
            # 我们将 x 的第 k 个二进制位置为 1，即为 x = x*2+1
            x_next = x * 2 + 1
            found = False
            
            # 枚举 i
            for num in nums:
                if x_next ^ (num >> k) in seen:
                    found = True
                    break

            if found:
                x = x_next
            else:
                # 如果没有找到满足等式的 a_i 和 a_j，那么 x 的第 k 个二进制位只能为 0
                # 即为 x = x*2
                x = x_next - 1
        
        return x
```
</details>
<details>
    <summary>JavaScript</summary>
 
```JavaScript [sol1-JavaScript]
var findMaximumXOR = function(nums) {
    const HIGH_BIT = 30;
    let x = 0;
    for (let k = HIGH_BIT; k >= 0; --k) {
        const seen = new Set();
        // 将所有的 pre^k(a_j) 放入哈希表中
        for (const num of nums) {
            // 如果只想保留从最高位开始到第 k 个二进制位为止的部分
            // 只需将其右移 k 位
            seen.add(num >> k);
        }

        // 目前 x 包含从最高位开始到第 k+1 个二进制位为止的部分
        // 我们将 x 的第 k 个二进制位置为 1，即为 x = x*2+1
        const xNext = x * 2 + 1;
        let found = false;
        
        // 枚举 i
        for (const num of nums) {
            if (seen.has(xNext ^ (num >> k))) {
                found = true;
                break;
            }
        }

        if (found) {
            x = xNext;
        } else {
            // 如果没有找到满足等式的 a_i 和 a_j，那么 x 的第 k 个二进制位只能为 0
            // 即为 x = x*2
            x = xNext - 1;
        }
    }
    return x; 
};
```
</details>
<details>
    <summary>Golang</summary>
 
```go [sol1-Golang]
func findMaximumXOR(nums []int) (x int) {
    const highBit = 30 // 最高位的二进制位编号为 30
    for k := highBit; k >= 0; k-- {
        // 将所有的 pre^k(a_j) 放入哈希表中
        seen := map[int]bool{}
        for _, num := range nums {
            // 如果只想保留从最高位开始到第 k 个二进制位为止的部分
            // 只需将其右移 k 位
            seen[num>>k] = true
        }

        // 目前 x 包含从最高位开始到第 k+1 个二进制位为止的部分
        // 我们将 x 的第 k 个二进制位置为 1，即为 x = x*2+1
        xNext := x*2 + 1
        found := false

        // 枚举 i
        for _, num := range nums {
            if seen[num>>k^xNext] {
                found = true
                break
            }
        }

        if found {
            x = xNext
        } else {
            // 如果没有找到满足等式的 a_i 和 a_j，那么 x 的第 k 个二进制位只能为 0
            // 即为 x = x*2
            x = xNext - 1
        }
    }
    return
}
```
</details>
<details>
    <summary>C</summary>
 
```C [sol1-C]
const int HIGH_BIT = 30;

struct HashTable {
    int key;
    UT_hash_handle hh;
};

int findMaximumXOR(int* nums, int numsSize) {
    int x = 0;
    for (int k = HIGH_BIT; k >= 0; --k) {
        struct HashTable* hashTable = NULL;
        // 将所有的 pre^k(a_j) 放入哈希表中
        for (int i = 0; i < numsSize; i++) {
            // 如果只想保留从最高位开始到第 k 个二进制位为止的部分
            // 只需将其右移 k 位
            int x = nums[i] >> k;
            struct HashTable* tmp;
            HASH_FIND_INT(hashTable, &x, tmp);
            if (tmp == NULL) {
                tmp = malloc(sizeof(struct HashTable));
                tmp->key = x;
                HASH_ADD_INT(hashTable, key, tmp);
            }
        }

        // 目前 x 包含从最高位开始到第 k+1 个二进制位为止的部分
        // 我们将 x 的第 k 个二进制位置为 1，即为 x = x*2+1
        int x_next = x * 2 + 1;
        bool found = false;

        // 枚举 i
        for (int i = 0; i < numsSize; i++) {
            int x = x_next ^ (nums[i] >> k);
            struct HashTable* tmp;
            HASH_FIND_INT(hashTable, &x, tmp);
            if (tmp != NULL) {
                found = true;
                break;
            }
        }

        if (found) {
            x = x_next;
        } else {
            // 如果没有找到满足等式的 a_i 和 a_j，那么 x 的第 k 个二进制位只能为 0
            // 即为 x = x*2
            x = x_next - 1;
        }
    }
    return x;
}
```

</details>

# 方法二：字典树

<details>
    <summary>C++</summary>
 

```C++ [sol2-C++]
struct Trie {
    // 左子树指向表示 0 的子节点
    Trie* left = nullptr;
    // 右子树指向表示 1 的子节点
    Trie* right = nullptr;

    Trie() {}
};

class Solution {
private:
    // 字典树的根节点
    Trie* root = new Trie();
    // 最高位的二进制位编号为 30
    static constexpr int HIGH_BIT = 30;

public:
    void add(int num) {
        Trie* cur = root;
        for (int k = HIGH_BIT; k >= 0; --k) {
            int bit = (num >> k) & 1;
            if (bit == 0) {
                if (!cur->left) {
                    cur->left = new Trie();
                }
                cur = cur->left;
            }
            else {
                if (!cur->right) {
                    cur->right = new Trie();
                }
                cur = cur->right;
            }
        }
    }

    int check(int num) {
        Trie* cur = root;
        int x = 0;
        for (int k = HIGH_BIT; k >= 0; --k) {
            int bit = (num >> k) & 1;
            if (bit == 0) {
                // a_i 的第 k 个二进制位为 0，应当往表示 1 的子节点 right 走
                if (cur->right) {
                    cur = cur->right;
                    x = x * 2 + 1;
                }
                else {
                    cur = cur->left;
                    x = x * 2;
                }
            }
            else {
                // a_i 的第 k 个二进制位为 1，应当往表示 0 的子节点 left 走
                if (cur->left) {
                    cur = cur->left;
                    x = x * 2 + 1;
                }
                else {
                    cur = cur->right;
                    x = x * 2;
                }
            }
        }
        return x;
    }

    int findMaximumXOR(vector<int>& nums) {
        int n = nums.size();
        int x = 0;
        for (int i = 1; i < n; ++i) {
            // 将 nums[i-1] 放入字典树，此时 nums[0 .. i-1] 都在字典树中
            add(nums[i - 1]);
            // 将 nums[i] 看作 ai，找出最大的 x 更新答案
            x = max(x, check(nums[i]));
        }
        return x;
    }
};
```
</details>
<details>
    <summary>Java</summary>
 
```Java [sol2-Java]
class Solution {
    // 字典树的根节点
    Trie root = new Trie();
    // 最高位的二进制位编号为 30
    static final int HIGH_BIT = 30;

    public int findMaximumXOR(int[] nums) {
        int n = nums.length;
        int x = 0;
        for (int i = 1; i < n; ++i) {
            // 将 nums[i-1] 放入字典树，此时 nums[0 .. i-1] 都在字典树中
            add(nums[i - 1]);
            // 将 nums[i] 看作 ai，找出最大的 x 更新答案
            x = Math.max(x, check(nums[i]));
        }
        return x;
    }

    public void add(int num) {
        Trie cur = root;
        for (int k = HIGH_BIT; k >= 0; --k) {
            int bit = (num >> k) & 1;
            if (bit == 0) {
                if (cur.left == null) {
                    cur.left = new Trie();
                }
                cur = cur.left;
            }
            else {
                if (cur.right == null) {
                    cur.right = new Trie();
                }
                cur = cur.right;
            }
        }
    }

    public int check(int num) {
        Trie cur = root;
        int x = 0;
        for (int k = HIGH_BIT; k >= 0; --k) {
            int bit = (num >> k) & 1;
            if (bit == 0) {
                // a_i 的第 k 个二进制位为 0，应当往表示 1 的子节点 right 走
                if (cur.right != null) {
                    cur = cur.right;
                    x = x * 2 + 1;
                } else {
                    cur = cur.left;
                    x = x * 2;
                }
            } else {
                // a_i 的第 k 个二进制位为 1，应当往表示 0 的子节点 left 走
                if (cur.left != null) {
                    cur = cur.left;
                    x = x * 2 + 1;
                } else {
                    cur = cur.right;
                    x = x * 2;
                }
            }
        }
        return x;
    }
}

class Trie {
    // 左子树指向表示 0 的子节点
    Trie left = null;
    // 右子树指向表示 1 的子节点
    Trie right = null;
}
```
</details>
<details>
    <summary>C#</summary>
 
```C# [sol2-C#]
public class Solution {
    // 字典树的根节点
    Trie root = new Trie();
    // 最高位的二进制位编号为 30
    const int HIGH_BIT = 30;

    public int FindMaximumXOR(int[] nums) {
        int n = nums.Length;
        int x = 0;
        for (int i = 1; i < n; ++i) {
            // 将 nums[i-1] 放入字典树，此时 nums[0 .. i-1] 都在字典树中
            Add(nums[i - 1]);
            // 将 nums[i] 看作 ai，找出最大的 x 更新答案
            x = Math.Max(x, Check(nums[i]));
        }
        return x;
    }

    public void Add(int num) {
        Trie cur = root;
        for (int k = HIGH_BIT; k >= 0; --k) {
            int bit = (num >> k) & 1;
            if (bit == 0) {
                if (cur.left == null) {
                    cur.left = new Trie();
                }
                cur = cur.left;
            }
            else {
                if (cur.right == null) {
                    cur.right = new Trie();
                }
                cur = cur.right;
            }
        }
    }

    public int Check(int num) {
        Trie cur = root;
        int x = 0;
        for (int k = HIGH_BIT; k >= 0; --k) {
            int bit = (num >> k) & 1;
            if (bit == 0) {
                // a_i 的第 k 个二进制位为 0，应当往表示 1 的子节点 right 走
                if (cur.right != null) {
                    cur = cur.right;
                    x = x * 2 + 1;
                } else {
                    cur = cur.left;
                    x = x * 2;
                }
            } else {
                // a_i 的第 k 个二进制位为 1，应当往表示 0 的子节点 left 走
                if (cur.left != null) {
                    cur = cur.left;
                    x = x * 2 + 1;
                } else {
                    cur = cur.right;
                    x = x * 2;
                }
            }
        }
        return x;
    }
}

class Trie {
    // 左子树指向表示 0 的子节点
    public Trie left = null;
    // 右子树指向表示 1 的子节点
    public Trie right = null;
}
```
</details>
<details>
    <summary>Python3</summary>
 
```Python [sol2-Python3]
class Trie:
    def __init__(self):
        # 左子树指向表示 0 的子节点
        self.left = None
        # 右子树指向表示 1 的子节点
        self.right = None

class Solution:
    def findMaximumXOR(self, nums: List[int]) -> int:
        # 字典树的根节点
        root = Trie()
        # 最高位的二进制位编号为 30
        HIGH_BIT = 30

        def add(num: int):
            cur = root
            for k in range(HIGH_BIT, -1, -1):
                bit = (num >> k) & 1
                if bit == 0:
                    if not cur.left:
                        cur.left = Trie()
                    cur = cur.left
                else:
                    if not cur.right:
                        cur.right = Trie()
                    cur = cur.right

        def check(num: int) -> int:
            cur = root
            x = 0
            for k in range(HIGH_BIT, -1, -1):
                bit = (num >> k) & 1
                if bit == 0:
                    # a_i 的第 k 个二进制位为 0，应当往表示 1 的子节点 right 走
                    if cur.right:
                        cur = cur.right
                        x = x * 2 + 1
                    else:
                        cur = cur.left
                        x = x * 2
                else:
                    # a_i 的第 k 个二进制位为 1，应当往表示 0 的子节点 left 走
                    if cur.left:
                        cur = cur.left
                        x = x * 2 + 1
                    else:
                        cur = cur.right
                        x = x * 2
            return x

        n = len(nums)
        x = 0
        for i in range(1, n):
            # 将 nums[i-1] 放入字典树，此时 nums[0 .. i-1] 都在字典树中
            add(nums[i - 1])
            # 将 nums[i] 看作 ai，找出最大的 x 更新答案
            x = max(x, check(nums[i]))

        return x
```
</details>
<details>
    <summary>Golang</summary>
 
```go [sol2-Golang]
const highBit = 30

type trie struct {
    left, right *trie
}

func (t *trie) add(num int) {
    cur := t
    for i := highBit; i >= 0; i-- {
        bit := num >> i & 1
        if bit == 0 {
            if cur.left == nil {
                cur.left = &trie{}
            }
            cur = cur.left
        } else {
            if cur.right == nil {
                cur.right = &trie{}
            }
            cur = cur.right
        }
    }
}

func (t *trie) check(num int) (x int) {
    cur := t
    for i := highBit; i >= 0; i-- {
        bit := num >> i & 1
        if bit == 0 {
            // a_i 的第 k 个二进制位为 0，应当往表示 1 的子节点 right 走
            if cur.right != nil {
                cur = cur.right
                x = x*2 + 1
            } else {
                cur = cur.left
                x = x * 2
            }
        } else {
            // a_i 的第 k 个二进制位为 1，应当往表示 0 的子节点 left 走
            if cur.left != nil {
                cur = cur.left
                x = x*2 + 1
            } else {
                cur = cur.right
                x = x * 2
            }
        }
    }
    return
}

func findMaximumXOR(nums []int) (x int) {
    root := &trie{}
    for i := 1; i < len(nums); i++ {
        // 将 nums[i-1] 放入字典树，此时 nums[0 .. i-1] 都在字典树中
        root.add(nums[i-1])
        // 将 nums[i] 看作 ai，找出最大的 x 更新答案
        x = max(x, root.check(nums[i]))
    }
    return
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```
</details>
<details>
    <summary>C</summary>
 
```C [sol2-C]
const int HIGH_BIT = 30;

struct Trie {
    // 左子树指向表示 0 的子节点
    struct Trie* left;
    // 右子树指向表示 1 的子节点
    struct Trie* right;
};

struct Trie* createTrie() {
    struct Trie* ret = malloc(sizeof(struct Trie));
    ret->left = ret->right = NULL;
    return ret;
}

void add(struct Trie* root, int num) {
    struct Trie* cur = root;
    for (int k = HIGH_BIT; k >= 0; --k) {
        int bit = (num >> k) & 1;
        if (bit == 0) {
            if (!cur->left) {
                cur->left = createTrie();
            }
            cur = cur->left;
        } else {
            if (!cur->right) {
                cur->right = createTrie();
            }
            cur = cur->right;
        }
    }
}

int check(struct Trie* root, int num) {
    struct Trie* cur = root;
    int x = 0;
    for (int k = HIGH_BIT; k >= 0; --k) {
        int bit = (num >> k) & 1;
        if (bit == 0) {
            // a_i 的第 k 个二进制位为 0，应当往表示 1 的子节点 right 走
            if (cur->right) {
                cur = cur->right;
                x = x * 2 + 1;
            } else {
                cur = cur->left;
                x = x * 2;
            }
        } else {
            // a_i 的第 k 个二进制位为 1，应当往表示 0 的子节点 left 走
            if (cur->left) {
                cur = cur->left;
                x = x * 2 + 1;
            } else {
                cur = cur->right;
                x = x * 2;
            }
        }
    }
    return x;
}

int findMaximumXOR(int* nums, int numsSize) {
    struct Trie* root = createTrie();
    int x = 0;
    for (int i = 1; i < numsSize; ++i) {
        // 将 nums[i-1] 放入字典树，此时 nums[0 .. i-1] 都在字典树中
        add(root, nums[i - 1]);
        // 将 nums[i] 看作 ai，找出最大的 x 更新答案
        x = fmax(x, check(root, nums[i]));
    }
    return x;
}
```

</details>


[^1]: https://leetcode-cn.com/problems/maximum-xor-of-two-numbers-in-an-array/solution/shu-zu-zhong-liang-ge-shu-de-zui-da-yi-h-n9m9/