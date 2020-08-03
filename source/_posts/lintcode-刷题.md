---
title: lintcode 刷题
date: 2020-04-01 10:48:28
tags: lintcode
---

# 简单题

## A + B 问题

> 给出两个整数，求和（用位运算）

<!--more-->

```python
class Solution:
	def aplusb(self, a, b):
		# write your code here
		if a == b: return a <<1
		if a == -b: return 0
		while(b != 0):
			tmpA = a^b
			b = (a&b)<<1
			a = tmpA
		return a
```

##  尾部的零

> 计算出n阶乘中尾部零的个数
>
> ```
> 样例  1:
> 	输入: 11
> 	输出: 2
> 	
> 	样例解释: 
> 	11! = 39916800, 结尾的0有2个。
> 
> 样例 2:
> 	输入:  5
> 	输出: 1
> 	
> 	样例解释: 
> 	5! = 120， 结尾的0有1个。
> ```

```python
class Solution:
    def trailingZeros(self, n):
        sum = 0
        while n!=0:
            sum += n // 5
            n = n // 5
        return sum
```

## 合并排序数组 II

> 合并两个有序升序的整数数组A和B变成一个新的数组。新数组也要有序。
>
> ```
> 输入: A=[1], B=[1]
> 输出:[1,1]	
> 样例解释: 返回合并后的数组。
> ```
>
> ```
> 输入: A=[1,2,3,4], B=[2,4,5,6]
> 输出: [1,2,2,3,4,4,5,6]	
> 样例解释: 返回合并后的数组。
> ```

```python
# 方法A
# 最简单的实现方式，但是当一个数组个数很少，一个很多时，是否可以优化
class Solution:
    """
    @param A: sorted integer array A
    @param B: sorted integer array B
    @return: A new sorted integer array
    """
    def mergeSortedArray(self, A, B):
        # write your code here
        return sorted(A+B)
     
# 方法B
class Solution:
    """
    @param A: sorted integer array A
    @param B: sorted integer array B
    @return: A new sorted integer array
    """
    def mergeSortedArray(self, A, B):
        i, j = 0, 0
        C = []
        while i < len(A) and j < len(B):
            if A[i] < B[j]:
                C.append(A[i])
                i += 1
            else:
                C.append(B[j])
                j += 1
        while i < len(A):
            C.append(A[i])
            i += 1
        while j < len(B):
            C.append(B[j])
            j += 1
            
        return C
```

## 旋转字符串

> 给定一个字符串（以字符数组的形式给出）和一个偏移量，根据偏移量`原地`旋转字符串(从左向右旋转)。

```
输入:  str="abcdefg", offset = 3
输出:  str = "efgabcd"	
样例解释:  注意是原地旋转，即str旋转后为"efgabcd"
```

```
输入: str="abcdefg", offset = 0
输出: str = "abcdefg"	
样例解释: 注意是原地旋转，即str旋转后为"abcdefg"
```

```python
class Solution:
    """
    @param str: An array of char
    @param offset: An integer
    @return: nothing
    """
    def rotateString(self, str, offset):
        # write your code here
        if offset==0 or not str:
            return str
        offset %= len(str)
        str = self.reverse(str,0,len(str))
        str = self.reverse(str,0,offset)
        str = self.reverse(str,offset,len(str))
        return str
    def reverse(self,str,start,end):
        while(start<end):
            tmp = str[start]
            str[start] = str[end-1]
            str[end-1] = tmp
            start += 1
            end -= 1
        return str

class Solution:
    # @param s: a list of char
    # @param offset: an integer 
    # @return: nothing
    def rotateString(self, s, offset):
        # write you code here
        if len(s) > 0:
            offset = offset % len(s)
        s = list(s)
        temp = (s + s)[len(s) - offset : 2 * len(s) - offset]

        for i in range(len(temp)):
            s[i] = temp[i]
        return ''.join(s)
```

## 字符串查找

> 对于一个给定的 source 字符串和一个 target 字符串，你应该在 source 字符串中找出 target 字符串出现的第一个位置(从0开始)。如果不存在，则返回 `-1`。

```
输入: source = "source" ， target = "target"
输出:-1	
样例解释: 如果source里没有包含target的内容，返回-1
```

```
输入: source = "abcdabcdefg" ，target = "bcd"
输出: 1	
样例解释: 如果source里包含target的内容，返回target在source里第一次出现的位置
```

```python
class Solution:
    def strStr(self, source, target):
        # write your code here
        if source is None or target is None:
            return -1
        len_s = len(source)
        len_t = len(target)
        for i in range(len_s - len_t + 1):
            j = 0
            while (j < len_t):
                if source[i + j] != target[j]:
                    break
                j += 1
            if j == len_t:
                return i
        return -1
```

## 二分查找

> 给定一个排序的整数数组（升序）和一个要查找的整数`target`，用`O(logn)`的时间查找到target第一次出现的下标（从0开始），如果target不存在于数组中，返回`-1`。

```
样例  1:
	输入:[1,4,4,5,7,7,8,9,9,10]，1
	输出: 0
	
	样例解释: 
	第一次出现在第0个位置。

样例 2:
	输入: [1, 2, 3, 3, 4, 5, 10]，3
	输出: 2
	
	样例解释: 
	第一次出现在第2个位置
	
样例 3:
	输入: [1, 2, 3, 3, 4, 5, 10]，6
	输出: -1
	
	样例解释: 
	没有出现过6， 返回-1
```

```python
class Solution:
    def binarySearch(self, nums, target):
        # write your code here
        if nums[0] > target or nums[len(nums)-1] < target:
            return -1
        pos = len(nums)//2
        while nums[pos] > target:
            pos //= 2
        
        for i in range(pos,2*pos+1):
            if nums[i] == target:
                pos = i
                break
            if i == 2*pos and nums[i] != target:
                return -1
        while pos>=1 and nums[pos] == nums[pos-1]:
            pos -= 1
        return pos
```

## 列表扁平化

```
样例  1:
	输入: [[1,1],2,[1,1]]
	输出:[1,1,2,1,1] 
	
	样例解释:
	将其变成一个只包含整数的简单列表。


样例 2:
	输入: [1,2,[1,2]]
	输出:[1,2,1,2]
	
	样例解释: 
	将其变成一个只包含整数的简单列表。
	
样例 3:
	输入:[4,[3,[2,[1]]]]
	输出:[4,3,2,1]
	
	样例解释: 
	将其变成一个只包含整数的简单列表。
```

```python
class Solution(object):
    def flatten(self, nestedList):
        # Write your code here
        a=nestedList
        if a is None: 
            return []
        self.b=[]
        if type(a)!=list:
            self.b.append(a)
        else:
            self.digui(a)
        return self.b

    def digui(self,c):
        if c is None: 
            return
        for d in c:
            if type(d) == list: 
                self.digui(d)
            else:   
                self.b.append(d)
```

### 搜索二维矩阵

```
样例  1:
	输入: [[5]],2
	输出: false
	
	样例解释: 
  没有包含，返回false。

样例 2:
	输入:  
[
  [1, 3, 5, 7],
  [10, 11, 16, 20],
  [23, 30, 34, 50]
],3
	输出: true
	
	样例解释: 
	包含则返回true。
```

```python
class Solution:
    def searchMatrix(self, matrix, target):
        # write your code here
        for eles in matrix:
            if eles[-1]<target:
                continue
            else:
                for j in eles:
                    if j == target:
                        return True
        return False
```

### 反转链表

```
输入: 1->2->3->null
输出: 3->2->1->null
```

```python
"""
Definition of ListNode

class ListNode(object):

    def __init__(self, val, next=None):
        self.val = val
        self.next = next
"""

class Solution:
    """
    @param head: n
    @return: The new head of reversed linked list.
    """
    def reverse(self, head):
        # write your code here
        tmp = None
        while head:
            cur = head.next
            head.next = tmp
            tmp = head
            head = cur
        return tmp
```

### 反转一个3位整数

> 反转一个只有3位数的整数。
>
> 你可以假设输入一定是一个只有三位数的整数，这个整数大于等于100，小于1000。

```
输入: number = 123
输出: 321
```

```
输入: number = 900
输出: 9
```

```python
class Solution:
    def reverseInteger(self, number):
        # write your code here
        number = str(number)[::-1]
        return int(number)
```

### 恢复旋转排序数组

### **说明**

什么是旋转数组？

- 比如，原始数组为[1,2,3,4], 则其旋转数组可以是[1,2,3,4], [2,3,4,1], [3,4,1,2], [4,1,2,3]

### **样例**

**Example1:**
`[4, 5, 1, 2, 3]` -> `[1, 2, 3, 4, 5]`
**Example2:**
`[6,8,9,1,2]` -> `[1,2,6,8,9]`

```C
class Solution:
    def recoverRotatedSortedArray(self, nums):
        # write your code here
        n = len(nums)
        for i in range(n):
            if nums[0] >= nums[-1]: 
                tmp = nums[0]
                nums.remove(nums[0])
                nums.append(tmp)
            else:
                return
```

### 最大子数组

> 给定一个整数数组，找到一个具有最大和的子数组，返回其最大和。

```
输入：[−2,2,−3,4,−1,2,1,−5,3]
输出：6
解释：符合要求的子数组为[4,−1,2,1]，其最大和为 6。
```

```
输入：[1,2,3,4]
输出：10
解释：符合要求的子数组为[1,2,3,4]，其最大和为 10。
```

```python
class Solution:
    def maxSubArray(self, nums):
        # write your code here
        flag = -10000
        count = 0
        for i in nums:
            count += i
            if count >= flag:
                flag = count
            if count < 0:
                count = 0
        return flag
```

### 主元素

> 给定一个整型数组，找出主元素，它在数组中的出现次数严格大于数组元素个数的二分之一。

```
输入: [1, 1, 1, 1, 2, 2, 2]
输出: 1
```

```python
class Solution:
    def majorityNumber(self, nums):
        # write your code here
        # nums = sorted(nums)
        # n = int(len(nums)/2)
        # return nums[n]
        
        ele = nums[0]  
        count = 0  
        for i in nums:  
            if ele == i:  
                count += 1  
            else:  
                count -= 1  
                if count <= 0:  
                    ele = i  
        return ele  
```

### 数组剔除元素后的乘积

### **描述**

给定一个整数数组A。
定义`B[i] = A[0] * ... * A[i-1] * A[i+1] * ... * A[n-1]`， 计算B的时候请不要使用除法。请输出B。

```
输入: A = [1, 2, 3]
输出: [6, 3, 2]
解析：B[0] = A[1] * A[2] = 6; B[1] = A[0] * A[2] = 3; B[2] = A[0] * A[1] = 2
```

```
输入: A = [2, 4, 6]
输出: [24, 12, 8]
```

```python
class Solution:
    def productExcludeItself(self, A):
        # write your code here
        length ,B  = len(A) ,[]
        f = [ 0 for i in range(length + 1)]
        f[ length ] = 1
        for i in range(length - 1 , 0 , -1):
            f[ i ] = f[ i + 1 ] * A[ i ]
        tmp = 1
        for i in range(length):
            B.append(tmp * f[ i + 1 ])
            tmp *= A[ i ]
        return B
```

### 翻转字符串中的单词

### **说明**

- 单词的构成：无空格字母构成一个单词，有些单词末尾会带有标点符号
- 输入字符串是否包括前导或者尾随空格？可以包括，但是反转后的字符不能包括
- 如何处理两个单词间的多个空格？在反转字符串中间空格减少到只含一个

```
样例  1:
	输入:  "the sky is blue"
	输出:  "blue is sky the"
	
	样例解释: 
	返回逐字反转的字符串.

样例 2:
	输入:  "hello world"
	输出:  "world hello"
	
	样例解释: 
	返回逐字反转的字符串.
```

```python
class Solution:
    def reverseWords(self, s):
        # write your code here
        return ' '.join(reversed(s.strip().split()))
```

### 比较字符串

### **样例**

给出 A = `"ABCD"` B = `"ACD"`，返回 `true`

给出 A = `"ABCD"` B = `"AABC"`， 返回 `false`

```python
class Solution:
    def compareStrings(self, A, B):
        # write your code here
        if len(A)<len(B):
            return False
        if len(A)>=0 and len(B)==0:
            return True
        dictA,dictB = {},{}
        for i in range(len(B)):
            if B[i] not in dictB:
                dictB[B[i]] = 1
            else:
                dictB[B[i]] += 1
        for i in range(len(A)):
            if A[i] not in dictA:
                dictA[A[i]] = 1
            else:
                dictA[A[i]] += 1
        for index,value in dictB.items():
            if index in dictA and value<=dictA[index]:
                continue
            else:
                return False
                
        return True
```

### 两数之和

### **描述**

给一个整数数组，找到两个数使得他们的和等于一个给定的数 *target*。

你需要实现的函数`twoSum`需要返回这两个数的下标, 并且第一个下标小于第二个下标。注意这里下标的范围是 0 到 *n-1*。

### **样例**

```
Example1:
给出 numbers = [2, 7, 11, 15], target = 9, 返回 [0, 1].
Example2:
给出 numbers = [15, 2, 7, 11], target = 9, 返回 [1, 2].
```

```python
class Solution:
    def twoSum(self, numbers, target):
        for i in range(len(numbers)-1):
            for j in range(i+1,len(numbers)):
                if numbers[i] + numbers[j] == target:
                    return [i,j]
        return False
```

### 搜索插入位置

### **描述**

给定一个排序数组和一个目标值，如果在数组中找到目标值则返回索引。如果没有，返回到它将会被按顺序插入的位置。

你可以假设在数组中无重复元素。

### **样例**

**[1,3,5,6]**，5 → 2

**[1,3,5,6]**，2 → 1

**[1,3,5,6]**， 7 → 4

**[1,3,5,6]**，0 → 0

```python
class Solution:
    def searchInsert(self, A, target):
        # write your code here
        if len(A) == 0:
            return 0
        if A[0]>target:
            return 0
        if A[len(A)-1] < target:
            return len(A)
        for i,num in enumerate(A):
            if num >= target:
                return i
```

### 二叉树的前中后序遍历

**样例 1:**

```
输入：{1,2,3}
输出：[1,2,3]
解释：
   1
  / \
 2   3
它将被序列化为{1,2,3}
前序遍历
```

**样例 2:**

```
输入：{1,#,2,3}
输出：[1,2,3]
解释：
1
 \
  2
 /
3
它将被序列化为{1,#,2,3}
前序遍历
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
    def preorderTraversal(self, root):
        # write your code here
        if root is None:
            return []
        first = root.val
        left = self.preorderTraversal(root.left)
        right = self.preorderTraversal(root.right)
        return [first] + left + right
     
class Solution:
    def preorderTraversal(self, root):
        # write your code here
        if root is None:
            return []
        first = root.val
        left = self.preorderTraversal(root.left)
        right = self.preorderTraversal(root.right)
        return  left + [first] +  right
      
class Solution:
    def preorderTraversal(self, root):
        # write your code here
        if root is None:
            return []
        first = root.val
        left = self.preorderTraversal(root.left)
        right = self.preorderTraversal(root.right)
        return  left + right + [first] 
```

### 二叉树的层次遍历

```python
"""
Definition of TreeNode:
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left, self.right = None, None
"""

class Solution:
    def __init__(self):
        self.l = []
    def levelOrder(self, root):
        # write your code here
        if root is None:
            return []
        a = []
        b = []
        a.append(root)
        length = len(a)
        while a:
            while length>0:
                h = a.pop(0)
                b.append(h.val)
                if h.left:
                    a.append(h.left)
                if h.right:
                    a.append(h.right)
                length -= 1
            length = len(a)
            self.l.append(b)
            b = []
        return self.l
```

### 落单的数

### **描述**

给出 `2 * n + 1`个数字，除其中一个数字之外其他每个数字均出现两次，找到这个数字。

**样例 1:**

```
输入：[1,1,2,2,3,4,4]
输出：3
解释：
仅3出现一次
```

**样例 2:**

```
输入：[0,0,1]
输出：1
解释：
仅1出现一次
```

```python
class Solution:
    def singleNumber(self, A):
        # write your code here
        ans = 0 
        for i in range(len(A)):
            ans ^= A[i]
        return ans
```

### 平衡二叉树

### **描述**

给定一个二叉树,确定它是高度平衡的。对于这个问题,一棵高度平衡的二叉树的定义是：一棵二叉树中每个节点的两个子树的深度相差不会超过1。 

### **样例**

```
样例  1:
	输入: tree = {1,2,3}
	输出: true
	
	样例解释:
	如下，是一个平衡的二叉树。
		   1  
		  / \                
		 2   3

	
样例  2:
	输入: tree = {3,9,20,#,#,15,7}
	输出: true
	
	样例解释:
	如下，是一个平衡的二叉树。
		  3  
		 / \                
		9  20                
		  /  \                
		 15   7 

	
样例  2:
	输入: tree = {1,#,2,3,4}
	输出: false
	
	样例解释:
	如下，是一个不平衡的二叉树。1的左右子树高度差2
		  1  
		   \                
		   2                
		  /  \                
		 3   4
	
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
    def isBalanced(self, root):
        # write your code here
        return self.depth(root) != -1
    
    def depth(self, root):
        if root is None:
            return 0
        left = self.depth(root.left)
        right = self.depth(root.right)
        if left == -1 or right == -1 or abs(left-right) > 1:
            return -1
        return max(left,right)+1
```

### 二叉树的最大深度

### **描述**

给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的距离。

**样例 1:**

```
输入: tree = {}
输出: 0	
样例解释: 空树的深度是0。
```

**样例 2:**

```
输入: tree = {1,2,3,#,#,4,5}
输出: 3
样例解释: 树表示如下，深度是3
   1
  / \                
 2   3                
    / \                
   4   5
它将被序列化为{1,2,3,#,#,4,5}
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
    def maxDepth(self, root):
        # write your code here
        if root is None:
            return 0
        left = self.maxDepth(root.left)
        right = self.maxDepth(root.right)
        return max(left,right)+1
```

### 最小路径和

### **描述**

给定一个只含非负整数的m*n网格，找到一条从左上角到右下角的可以使数字和最小的路径。

### **样例**

```
样例 1:
	输入:  [[1,3,1],[1,5,1],[4,2,1]]
	输出: 7
	
	样例解释：
	路线为： 1 -> 3 -> 1 -> 1 -> 1。


样例 2:
	输入:  [[1,3,2]]
	输出:  6
	
	解释:  
	路线是： 1 -> 3 -> 2
```

```python
class Solution:
    """
    @param grid: a list of lists of integers
    @return: An integer, minimizes the sum of all numbers along its path
    """
    def minPathSum(self, grid):
        # write your code here
        m = len(grid)
        n = len(grid[0])
        
        dp = [[0 for _ in range(n)] for _ in range(m)]
        dp[0][0] = grid[0][0]
        for i in range(1,m):
            dp[i][0] = dp[i-1][0] + grid[i][0]
        for i in range(1,n):
            dp[0][i] = dp[0][i-1] + grid[0][i]
            
        for i in range(1, m):
            for j in range(1, n):
                dp[i][j] = min(dp[i-1][j], dp[i][j-1])+grid[i][j]
        return dp[m-1][n-1]
```

### 爬楼梯

### **描述**

假设你正在爬楼梯，需要n步你才能到达顶部。但每次你只能爬一步或者两步，你能有多少种不同的方法爬到楼顶部？

### **样例**

```
Example 1:
	Input:  n = 3
	Output: 3
	
	Explanation:
	1) 1, 1, 1
	2) 1, 2
	3) 2, 1
	total 3.


Example 2:
	Input:  n = 1
	Output: 1
	
	Explanation:  
	only 1 way.
```

```python
class Solution:
    def climbStairs(self, n):
        if n == 0:
            return 0
        if n == 1:
            return 1
        dp = [0 for _ in range(n)]
        dp[0] = 1
        dp[1] = 2
        for i in range(2,n):
            dp[i] = dp[i-1]+dp[i-2]
        return dp[-1]
```

### 删除排序链表中的重复元素

### **描述**

给定一个排序链表，删除所有重复的元素每个元素只留下一个。

### **样例**

```
样例 1:
	输入:  null
	输出: null


样例 2:
	输入: 1->1->2->null
	输出: 1->2->null

样例 3:
	输入: 1->1->2->3->3->null
	输出: 1->2->3->null
```

```python
"""
Definition of ListNode
class ListNode(object):
    def __init__(self, val, next=None):
        self.val = val
        self.next = next
"""

class Solution:
    """
    @param head: head is the head of the linked list
    @return: head of linked list
    """
    def deleteDuplicates(self, head):
        # write your code here
        if head is None:
            return
        if head.next is None:
            return head
        q = head
        while head and head.next:
            if head.val == head.next.val:
                head.next = head.next.next
            else:
                head = head.next
        return q
```

### 不同的路径

### **描述**

有一个机器人的位于一个 *m* × *n* 个网格左上角。

机器人每一时刻只能向下或者向右移动一步。机器人试图达到网格的右下角。

问有多少条不同的路径？

### **样例**

**Example 1:**

```
Input: n = 1, m = 3
Output: 1	
Explanation: Only one path to target position.
```

**Example 2:**

```
Input:  n = 3, m = 3
Output: 6	
Explanation:
	D : Down
	R : Right
	1) DDRR
	2) DRDR
	3) DRRD
	4) RRDD
	5) RDRD
	6) RDDR
```

```python
class Solution:
    def uniquePaths(self, m, n):
        dp = [[1 for i in range(n)] for j in range(m)]
        for i in range(1,m):
            for j in range(1,n):
                dp[i][j] = dp[i-1][j]+dp[i][j-1]
        return dp[m-1][n-1]
```

### 不同的路径 II

### **描述**

"[不同的路径](http://www.lintcode.com/problem/unique-paths/)" 的跟进问题：

现在考虑网格中有障碍物，那样将会有多少条不同的路径？

网格中的障碍和空位置分别用 1 和 0 来表示。

```python
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid):
        # write your code here
        m = len(obstacleGrid)
        n = len(obstacleGrid[0])
        dp = [[0 for _ in range(n)] for _ in range(m)]
        for i in range(m):
            if obstacleGrid[i][0] == 1:
                break
            else:
                dp[i][0] = 1
                
        for i in range(n):
            if obstacleGrid[0][i] == 1:
                break
            else:
                dp[0][i] = 1
        
        for i in range(1,m):
            for j in range(1,n):
                if obstacleGrid[i][j]==1:
                    dp[i][j] == 0
                else:
                    dp[i][j] = dp[i-1][j]+dp[i][j-1]
        return dp[m-1][n-1]
```

### 二叉树的最小深度

### **描述**

给定一个二叉树，找出其最小深度。

二叉树的最小深度为根节点到最近叶子节点的最短路径上的节点数量。

**样例 1:**

```
输入: {}
输出: 0
```

**样例 2:**

```
输入:  {1,#,2,3}
输出: 3	
解释:
	1
	 \ 
	  2
	 /
	3    
它将被序列化为 {1,#,2,3}
```

**样例 3:**

```
输入:  {1,2,3,#,#,4,5}
输出: 2	
解释: 
      1
     / \ 
    2   3
       / \
      4   5  
它将被序列化为 {1,2,3,#,#,4,5}
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
    def minDepth(self, root):
        # write your code here
        if root is None:
            return 0
        left = self.minDepth(root.left)
        right = self.minDepth(root.right)
        if (left==0 or right==0):
            ret = left+right+1
        else:
            ret = min(left,right)+1
        return ret
```

### 合并两个排序链表

### **样例**

```
样例 1:
	输入: list1 = null, list2 = 0->3->3->null
	输出: 0->3->3->null


样例2:
	输入:  list1 =  1->3->8->11->15->null, list2 = 2->null
	输出: 1->2->3->8->11->15->null
```

```python
"""
Definition of ListNode
class ListNode(object):
    def __init__(self, val, next=None):
        self.val = val
        self.next = next
"""

class Solution:
    def mergeTwoLists(self, l1, l2):
        # write your code here
        dummy = ListNode(0)
        tmp = dummy
        while l1 != None and l2 != None:
            if l1.val < l2.val:
                tmp.next = l1
                l1 = l1.next
            else:
                tmp.next = l2
                l2 = l2.next
            tmp = tmp.next
        if l1 != None:
            tmp.next = l1
        else:
            tmp.next = l2
        return dummy.next
```

### 链表求和

> 你有两个用链表代表的整数，其中每个节点包含一个数字。数字存储按照在原来整数中`相反`的顺序，使得第一个数字位于链表的开头。写出一个函数将两个整数相加，用链表形式返回和。

**样例 1:**

```
输入: 7->1->6->null, 5->9->2->null
输出: 2->1->9->null	
样例解释: 617 + 295 = 912, 912 转换成链表:  2->1->9->null
```

**样例 2:**

```
输入:  3->1->5->null, 5->9->2->null
输出: 8->0->8->null	
样例解释: 513 + 295 = 808, 808 转换成链表: 8->0->8->null
```

```python
"""
Definition of ListNode
class ListNode(object):
    def __init__(self, val, next=None):
        self.val = val
        self.next = next
"""

class Solution:
    def addLists(self, l1, l2):
        # write your code here
        head = ListNode(0)
        ptr = head
        count = 0
        while True:
            if l1:
                count += l1.val
                l1 = l1.next
            if l2:
                count += l2.val
                l2 = l2.next
            ptr.val = count%10
            count = count//10
            if l1 or l2 or count:
                ptr.next = ListNode(0)
                ptr = ptr.next
            else:
                break
        return head
```

### 删除链表中倒数第n个节点

### **样例**

```
Example 1:
	Input: list = 1->2->3->4->5->null， n = 2
	Output: 1->2->3->5->null


Example 2:
	Input:  list = 5->4->3->2->1->null, n = 2
	Output: 5->4->3->1->null
```

```python
"""
Definition of ListNode
class ListNode(object):
    def __init__(self, val, next=None):
        self.val = val
        self.next = next
"""

class Solution:
    def removeNthFromEnd(self, head, n):
        # write your code here
        
        if head is None and n > 0:
            return
        
        p = head
        q = head
        m = p
        while n >= 0 and q:
            q = q.next
            n -= 1
        if q is None:
            p = p.next
            return p
        while q and p:
            q = q.next
            p = p.next
        p.next = p.next.next
        return m
```

### 字符串置换

给定两个字符串，请设计一个方法来判定其中一个字符串是否为另一个字符串的置换。

置换的意思是，通过改变顺序可以使得两个字符串相等。

### **样例**

```
Example 1:
	Input:  "abcd", "bcad"
	Output:  True


Example 2:
	Input: "aac", "abc"
	Output:  False
```

```python
class Solution:
    def Permutation(self, A, B):
        # write your code here
        if len(A)!=len(B):
            return False
        a = sorted(A)
        b = sorted(B)
        if a == b:
            return True
        else:
            return False
```

### 二叉树的路径和

### **描述**

给定一个二叉树，找出所有路径中各节点相加总和等于给定 `目标值` 的路径。

一个有效的路径，指的是从根节点到叶节点的路径。

**样例1:**

```plain
输入:
{1,2,4,2,3}
5
输出: [[1, 2, 2],[1, 4]]
说明:
这棵树如下图所示：
	     1
	    / \
	   2   4
	  / \
	 2   3
对于目标总和为5，很显然1 + 2 + 2 = 1 + 4 = 5
```

**样例2:**

```plain
输入:
{1,2,4,2,3}
3
输出: []
说明:
这棵树如下图所示：
	     1
	    / \
	   2   4
	  / \
	 2   3
注意到题目要求我们寻找从根节点到叶子节点的路径。
1 + 2 + 2 = 5, 1 + 2 + 3 = 6, 1 + 4 = 5 
这里没有合法的路径满足和等于3.
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
    def binaryTreePathSum(self, root, target):
        # write your code here
        if root is None:
            return []
        if root.left is None and root.right is None and root.val == target:
            return [[root.val]]
        ret = []
        left = self.binaryTreePathSum(root.left,target-root.val)
        right = self.binaryTreePathSum(root.right,target-root.val)
        for i in left+right:
            ret.append([root.val]+i)
        return ret
```

### 最长上升连续子序列

### **描述**

给定一个整数数组（下标从 0 到 n-1， n 表示整个数组的规模），请找出该数组中的最长上升连续子序列。（最长上升连续子序列可以定义为从右到左或从左到右的序列。）

**样例 1：**

```
输入：[5, 4, 2, 1, 3]
输出：4
解释：
给定 [5, 4, 2, 1, 3]，其最长上升连续子序列（LICS）为 [5, 4, 2, 1]，返回 4。
```

**样例 2：**

```
输入：[5, 1, 2, 3, 4]
输出：4
解释：
给定 [5, 1, 2, 3, 4]，其最长上升连续子序列（LICS）为 [1, 2, 3, 4]，返回 4。
```

```python
class Solution:
    def longestIncreasingContinuousSubsequence(self, A):
        # write your code here
        if len(A)==0:
            return 0
        if len(A)==1:
            return 1
        dec = [1]
        inc = [1]
        cur , n = 1, len(A)
        while cur<n:
            if A[cur]>A[cur-1]:
                inc.append(inc[-1]+1)
                dec.append(1)
            else:
                dec.append(dec[-1]+1)
                inc.append(1)
            cur += 1
        return max(max(dec), max(inc))
```

### 有效的括号序列

### **描述**

给定一个字符串所表示的括号序列，包含以下字符： `'(', ')'`, `'{'`, `'}'`, `'['` and `']'`， 判定是否是有效的括号序列。

**样例 1：**

```
输入："([)]"
输出：False
```

**样例 2：**

```
输入："()[]{}"
输出：True
```

```python
class Solution:
    def isValidParentheses(self, s):
        # write your code here
        stack = []
        for i in s:
            if i=='(' or i=='[' or i=='{':
                stack.append(i)
            else:
                if not stack:
                    return False
                if i==']' and stack[-1]!='[' or i==')' and stack[-1]!='(' or i=='}' and stack[-1]!='{':
                    return False
                stack.pop()
    
        return not stack
```

### 二叉树的所有路径

### **描述**

给一棵二叉树，找出从根节点到叶子节点的所有路径。

**样例 1:**

```
输入：{1,2,3,#,5}
输出：["1->2->5","1->3"]
解释：
   1
 /   \
2     3
 \
  5
```

**样例 2:**

```
输入：{1,2}
输出：["1->2"]
解释：
   1
 /   
2     
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
    def binaryTreePaths(self, root):
        # write your code here
        if root is None:
            return []
        ret = []
        if root.left is None and root.right is None:
            return [str(root.val)]
        left = self.binaryTreePaths(root.left)
        right = self.binaryTreePaths(root.right)
        for i in left+right:
            ret.append(str(root.val)+'->'+str(i))
        return ret
```

### 栅栏染色

我们有一个栅栏，它有`n`个柱子，现在要给柱子染色，有`k`种颜色可以染。
必须保证不存在超过2个相邻的柱子颜色相同，求有多少种染色方案。

```python
class Solution:
    def numWays(self, n, k):
        # write your code here
        if n==1:
            return k
        if n==2:
            return k*k
        if k<=1:
            return 0
        dp = [0 for i in range(n)]
        dp[0] = k
        dp[1] = k*k
        for i in range(2,n):
            dp[i] = dp[i-1]*(k-1) + dp[i-2]*(k-1)
        return dp[-1]
```

