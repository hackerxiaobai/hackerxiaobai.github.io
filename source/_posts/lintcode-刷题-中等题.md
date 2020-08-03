---
title: lintcode 刷题(中等题)
date: 2020-04-14 13:53:16
tags: lintcode
---

### 统计数字

<!-- more -->

**样例 1：**

```
输入：
k = 1, n = 1
输出：
1
解释：
在 [0, 1] 中，我们发现 1 出现了 1 次 (1)。
```

**样例 2：**

```
输入：
k = 1, n = 12
输出：
5
解释：
在 [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12] 中，我们发现 1 出现了 5 次 (1, 10, 11, 12)(注意11中有两个1)。
```

```python
class Solution:
    def digitCounts(self, k, n):
        # write your code here
        s =''.join([str(i) for i in range(n+1)])
        count = 0
        for i in range(len(s)):
            if s[i]==str(k):
                count += 1
        return count
```

### 第k大元素

**样例 1：**

```
输入：
n = 1, nums = [1,3,4,2]
输出：
4
```

**样例 2：**

```
输入：
n = 3, nums = [9,3,2,4,8]
输出：
4
```

```python
class Solution:
    def kthLargestElement(self, n, nums):
        # write your code here
        return sorted(nums,reverse = True)[n-1]
```

### 全排列

**样例 1：**

```
输入：[1]
输出：
[
  [1]
]
```

**样例 2：**

```
输入：[1,2,3]
输出：
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]
```

```python
class Solution:
    def permute(self, nums):
        # write your code here
        if len(nums)==0:
            return [nums]
        if len(nums)==1:
            return [nums]
        sl = []
        for i in range(len(nums)):
            for j in self.permute(nums[0:i]+nums[i+1:]):
                sl.append([nums[i]] + j)
        return sl
```

### 带重复元素的排列

**样例 1：**

```
输入：[1,1]
输出：
[
  [1,1]
]
```

**样例 2：**

```
输入：[1,2,2]
输出：
[
  [1,2,2],
  [2,1,2],
  [2,2,1]
]
```

```python
class Solution:
    def permuteUnique(self, nums):
        # write your code here
        if len(nums)==0:
            return [nums]
        if len(nums)==1:
            return [nums]
        sl = []
        for i in range(len(nums)):
            for j in self.permuteUnique(nums[0:i]+nums[i+1:]):
                sl.append([nums[i]]+j)
        index = []
        for i in sl:
            if i not in index:
                index.append(i)
        return index
```

### 子集

给定一个含不同整数的集合，返回其所有的子集。

**样例 1：**

```
输入：[0]
输出：
[
  [],
  [0]
]
```

**样例 2：**

```
输入：[1,2,3]
输出：
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```

```python
class Solution:
    def subsets(self, nums):
        self.ret = []
        self.helper(sorted(nums),[],0)
        return self.ret
        
    def helper(self,nums,s,index):
        if len(nums)==index:
            self.ret.append(s)
            return
        self.helper(nums,s+[nums[index]],index+1)
        self.helper(nums,s,index+1)
```

### 交叉字符串

给出三个字符串:*s1*、*s2*、*s3*，判断*s3*是否由*s1*和*s2*交叉构成。

**样例 1：**

```
输入:
"aabcc"
"dbbca"
"aadbbcbcac"
输出:
true
```

**样例 2：**

```
输入:
""
""
"1"
输出:
false
```

**样例 3：**

```
输入:
"aabcc"
"dbbca"
"aadbbbaccc"
输出:
false
```

```python
class Solution:
    def isInterleave(self,s1, s2, s3):
        # write your code here
        if len(s1)+len(s2)!=len(s3):
            return False
        dp = [[False for _ in range(len(s2)+1)] for _ in range(len(s1)+1)]
        dp[0][0] = True
        for i in range(1, len(s1)+1):
            if s1[i-1]==s3[i-1]:
                dp[i][0] = True
        for i in range(1, len(s2)+1):
            if s2[i-1]==s3[i-1]:
                dp[0][i] = True
        
        for i in range(1,len(s1)+1):
            for j in range(1,len(s2)+1):
                k = i + j
                if s1[i-1] == s3[k-1]:
                    dp[i][j] = dp[i][j] or dp[i-1][j]
                if s2[j-1] == s3[k-1]:
                    dp[i][j] = dp[i][j] or dp[i][j-1]
        return dp[len(s1)][len(s2)]
```

### 插入区间

给出一个**无重叠的**按照区间起始端点排序的区间列表。

在列表中插入一个新的区间，你要确保列表中的区间仍然有序且**不重叠**（如果有必要的话，可以合并区间）。

**样例 1：**

```
输入:
(2, 5) into [(1,2), (5,9)]
输出:
[(1,9)]
```

**样例 2：**

```
输入:
(3, 4) into [(1,2), (5,9)]
输出:
[(1,2), (3,4), (5,9)]
```

```python
"""
Definition of Interval.
class Interval(object):
    def __init__(self, start, end):
        self.start = start
        self.end = end
"""

class Solution:
    def insert(self, intervals, newInterval):
        res = []
        pos = 0
        for interval in intervals:
            if interval.end < newInterval.start:
                res.append(interval)
                pos += 1
            elif interval.start > newInterval.end:
                res.append(interval)
            else:
                newInterval.start = min(interval.start, newInterval.start)
                newInterval.end = max(interval.end, newInterval.end)
        res.insert(pos, newInterval)
        return res
```

### 最大子数组 II

给定一个整数数组，找出两个 *不重叠* 子数组使得它们的和最大。
每个子数组的数字在数组中的位置应该是连续的。
返回最大的和。

例1:

```
输入:
[1, 3, -1, 2, -1, 2]
输出:
7
解释:
最大的子数组为 [1, 3] 和 [2, -1, 2] 或者 [1, 3, -1, 2] 和 [2].
```

例2:

```
输入:
[5,4]
输出:
9
解释:
最大的子数组为 [5] 和 [4].
```

```python
class Solution:
    def maxTwoSubArrays(self, nums):
        flag = -10000
        count = 0
        left = []
        for i in nums:
            count += i
            flag = max(flag,count)
            count = max(0,count)
            left.append(flag)
            
        flag = -10000
        count = 0
        right = [0 for _ in range(len(nums))]
        for i in range(len(nums)-1,-1,-1):
            count += nums[i]
            flag = max(flag,count)
            count = max(0,count)
            right[i] = flag
            
        result = -10000
        for i in range(len(left)-1):
            result = max(result,left[i]+right[i+1])
        return result
```

### 主元素 II

给定一个整型数组，找到主元素，它在数组中的出现次数严格大于数组元素个数的三分之一。

例1:

```
输入: [99,2,99,2,99,3,3], 
输出: 99.
```

例2:

```
输入: [1, 2, 1, 2, 1, 3, 3], 
输出: 1.
```

```python
class Solution:
    def majorityNumber(self, nums):
        # write your code here
        hash_table = {}  
        for i in nums:  
            if i not in hash_table:  
                hash_table[i] = 1  
            else:  
                hash_table[i] += 1  
  
            if len(hash_table) == 3:  
                for key in hash_table:  
                    hash_table[key] -= 1  
                temp = {}  
                for key in hash_table:  
                    if hash_table[key] != 0:  
                        temp[key] = hash_table[key]  
                hash_table = temp  
  
        for key in hash_table:  
            hash_table[key] = 0  
  
        for i in nums:  
            if i in hash_table:  
                hash_table[i] += 1  
  
        return max(hash_table.items(), key=lambda x: x[1])[0]
```

### 二叉树的层次遍历 II

给出一棵二叉树，返回其节点值从底向上的层次序遍历（按从叶节点所在层到根节点所在的层遍历，然后逐层从左往右遍历

### **样例**

例1:

```
输入:
{1,2,3}
输出:
[[2,3],[1]]
解释:
    1
   / \
  2   3
它将被序列化为 {1,2,3}
层次遍历
```

例2:

```
输入:
{3,9,20,#,#,15,7}
输出:
[[15,7],[9,20],[3]]
解释:
    3
   / \
  9  20
    /  \
   15   7
它将被序列化为 {3,9,20,#,#,15,7}
层次遍历
```

```python
"""
Definition of TreeNode:
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left, self.right = None, None
"""

class Solution:
    def levelOrderBottom(self, root):
        # write your code here
        if root is None:
            return []
        ret = []
        q = [root]
        while q:
            new_q = []
            ret.append([n.val for n in q])
            for node in q:
                if node.left:
                    new_q.append(node.left)
                if node.right:
                    new_q.append(node.right)
            q = new_q
        return list(reversed(ret))
```

### 二叉树的锯齿形层次遍历

给出一棵二叉树，返回其节点值的锯齿形层次遍历（先从左往右，下一层再从右往左，层与层之间交替进行） 

**样例 1:**

```
输入:{1,2,3}
输出:[[1],[3,2]]
解释:
    1
   / \
  2   3
它将被序列化为 {1,2,3}
```

**样例 2:**

```
输入:{3,9,20,#,#,15,7}
输出:[[3],[20,9],[15,7]]
解释:
    3
   / \
  9  20
    /  \
   15   7
它将被序列化为 {3,9,20,#,#,15,7}
```

```python
"""
Definition of TreeNode:
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left, self.right = None, None
"""

class Solution:
    def zigzagLevelOrder(self, root):
        # write your code here
        if root is None:
            return []
        tmp = []
        q = [root]
        flag = 1
        while q:
            new_q = []
            for node in q:
                if node.left:
                    new_q.append(node.left)
                if node.right:
                    new_q.append(node.right)
            if flag%2!=0:
                tmp.append([i.val for i in q])
            else:
                tmp.append([i.val for i in q[::-1]])
            flag += 1
            q = new_q
        return tmp
```

### 最长上升子序列

给定一个整数序列，找到最长上升子序列（LIS），返回LIS的长度。

### **样例**

```
样例 1:
	输入:  [5,4,1,2,3]
	输出:  3
	
	解释:
	LIS 是 [1,2,3]


样例 2:
	输入: [4,2,4,5,3,7]
	输出:  4
	
	解释: 
	LIS 是 [2,4,5,7]
```

```python
class Solution:
    def longestIncreasingSubsequence(self, nums):
        if len(nums)==0:
            return 0
        if len(nums)==1:
            return 1
        dp = [1 for _ in range(len(nums))]
        for i in range(len(nums)):
            for j in range(i):
                if nums[i]>nums[j] and dp[j]+1 > dp[i]:
                    dp[i] = dp[j]+1
        return max(dp)
```

### 最长公共子序列

给出两个字符串，找到最长公共子序列(LCS)，返回LCS的长度。

### **样例**

```
样例 1:
	输入:  "ABCD" and "EDCA"
	输出:  1
	
	解释:
	LCS 是 'A' 或  'D' 或 'C'


样例 2:
	输入: "ABCD" and "EACB"
	输出:  2
	
	解释: 
	LCS 是 "AC"
```

```python
class Solution:
    def longestCommonSubsequence(self, A, B):
        dp = [[0 for _ in range(len(B)+1)] for _ in range(len(A)+1)]
        for i in range(1,len(A)+1):
            for j in range(1,len(B)+1):
                if A[i-1]==B[j-1]:
                    dp[i][j] = dp[i-1][j-1]+1
                else:
                    dp[i][j] = max(dp[i-1][j],dp[i][j-1])
        return dp[len(A)][len(B)]
```

### 最长公共子串

给出两个字符串，找到最长公共子串，并返回其长度。

### **样例**

```
样例 1:
	输入:  "ABCD" and "CBCE"
	输出:  2
	
	解释:
	最长公共子串是 "BC"


样例 2:
	输入: "ABCD" and "EACB"
	输出:  1
	
	解释: 
	最长公共子串是 'A' 或 'C' 或 'B'
```

```python
class Solution:
    def longestCommonSubstring(self, A, B):
        ans = 0
        for i in range(len(A)):
            for j in range(len(B)):
                l = 0
                while i + l < len(A) and j + l < len(B) \
                    and A[i + l] == B[j + l]:
                    l += 1
                if l > ans:
                    ans = l
        return ans
```

### 最小调整代价

### **描述**

给一个整数数组，调整每个数的大小，使得相邻的两个数的差不大于一个给定的整数target，调整每个数的代价为调整前后的差的绝对值，求调整代价之和最小是多少。

**你可以假设数组中每个整数都是正整数，且小于等于100。**

样例 1: 	

输入:  [1,4,2,3], target=1  

输出:  2 

样例 2: 	

输入:  [3,5,4,7], target=2 	
输出:  1 	

```python
class Solution:
    """
    @param: A: An integer array
    @param: target: An integer
    @return: An integer
    """
    def MinAdjustmentCost(self, A, target):
        # write your code here
        if len(A)==1:
            return 0
        dp = [[float('inf') for _ in range(101)] for _ in range(len(A))]
        
        for i in range(101):
            diff = abs(i-A[0])
            dp[0][i] = diff
            
        for i in range(1,len(A)):
            for j in range(101):
                diff = abs(j-A[i])
                m = min(100,j+target)
                n = max(0,j-target)
                for k in range(n,m+1):
                    dp[i][j] = min(dp[i][j], dp[i-1][k]+diff) 
        result = float('inf')
        for i in range(101):
            result = min(result,dp[len(A)-1][i])
        return result
```



### 背包问题

描述 

在n个物品中挑选若干物品装入背包，最多能装多满？假设背包的大小为m，每个物品的大小为A[i]

样例 1: 	

输入:  [3,4,8,5], backpack size=10 	

输出:  9  

样例 2: 	

输入:  [2,3,5,7], backpack size=12 	

输出:  12 	

```python
class Solution:
    """
    @param m: An integer m denotes the size of a backpack
    @param A: Given n items with size A[i]
    @return: The maximum size
    """
    def backPack(self, m, A):
        # write your code here
        # dp = [[0 for j in range(m+1)] for i in range(len(A))]
        # for j in range(m+1):
        #     if A[0] > j:
        #         dp[0][j] = 0
        #     else:
        #         dp[0][j] = A[0]
        #     for i in range(1,len(A)):
        #         if A[i] > j:
        #             dp[i][j] = dp[i-1][j]
        #         else:
        #             dp[i][j] = max(dp[i-1][j-A[i]] + A[i], dp[i-1][j] )
        # return dp[len(A)-1][m]
        
        dp = [0 for _ in range(m+1)]
        for i in range(len(A)):
            for j in range(m,0,-1):
                if j >= A[i]:
                    dp[j] = max(dp[j],dp[j-A[i]] + A[i])
        return dp[-1]
```



