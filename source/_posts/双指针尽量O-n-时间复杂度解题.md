---
title: 双指针尽量O(n)时间复杂度解题
date: 2020-10-30 21:16:56
tags:
	- 双指针
	- O(n)时间复杂度
---

### 双指针相关解释

+ 双指针个人从遇到的一些题目上来看，一般包括两类
  + 快慢指针（比如解决链表中是否包含环）
  + 左右指针（主要解决数组或字符串的问题）



<!--more-->

#### 快慢指针

> 快慢指针一般都初始化指向链表的头结点 head，前进时快指针 fast 在前，慢指针 slow 在后，巧妙解决一些链表中的问题。

+ 判定链表中是否含有环

  + 单链表一般都是只有next指针指向下一个节点，定义如下：

  + ```python
    class Node(object):
      	def __inti__(self, value):
        		self.value = value
        		self.next = None
    ```

  + 一般不包含环的链表遍历一遍都是以遇到None为结束

  + ```python
    class Process(object):
      	def solve(self, node):
        		while(node):
          		node = node.next
        		return '没有环，不会进入死循环'
    ```

  + 若链表中有环则会一直死循环，此时，用两个指针，fast指针和slow指针，若有环，fast指针肯定会追上slow指针，如下

  + ```python
    class Process(object):
      	def solve(self, node):
          	fast = slow = node
            while(fast!=None and fast.next!=None):
              	fast = fast.next.next
                slow = slow.next
                if fast==slow:
                  	return True
            return False
    ```

+ 以知链表有环，找到该环的起始节点

  + 仍然以快慢指针为例，先让它两第一次相遇，此时假设slow 走了 k 步，则 fast一定走了 2k 步，若相遇，则肯定是在环上相遇，假设从相遇点回退到环的起点为 m 步，则 k-m 的距离就是链表的起点到环的起点的距离，而我们知道 fast 是比 slow 多走了k步，多走的这k步肯定是绕环走出来的，且肯定刚好是环的n倍，又知道从相遇点回退到环的起点为 m 步，则若从相遇点继续走k-m步，不是刚好到环的起点吗

  + ```python
    class Process(object):
      	def solve(self, node):
          	fast = slow = node
            while(fast!=None and fast.next!=None):
              	fast = fast.next.next
                slow = slow.next
                if fast==slow:
                  	break
            slow = node
            while(slow!=fast):
              	slow = slow.next
                fast = fast.next
            return slow
    ```

+ 查找链表的中间节点

  + 仍然以快慢节点为例，从上题我们就知道，若fast 和 slow 相遇，则 fast一定比slow 多走k步，所以，如果fast走到尾节点，此时slow不就刚好比fast 少走一半吗？

  + ```python
    class Process(object):
      	def solve(self, node):
          	fast = slow = node
            while(fast!=None):
              	fast = fast.next.next
                slow = slow.next
            return slow
    ```

+ 查找链表的倒数第k个节点

  + 还是双指针，可以让第一个指针先走k步，然后让两个指针同步走，当第一个指针到尾节点时，另一个指针就刚好是倒数第k个

  + ```python
    class Process(object):
      	def solve(self, node, k):
          	fast = slow = node
            while(k>0):
              	fast = fast.next
                k -= 1
            
            while(fast!=None):
              	fast = fast.next
                slow = slow.next
            return slow
    ```

#### 左右指针

> 左右指针在数组中实际是指两个索引值，一般初始化为 left = 0, right = nums.length - 1 。

+ 最常见一般是二分查找算法的实现，常见代码如下：

  + ```python
    class BinarySearch(object):
      	def solve(self, nums, target):
          	left,right = 0,len(nums)-1
            while(left<=right):
              	middle = (left+right)//2
                if nums[middle]==target:
                  	return True
                elif nums[middle]<target:
                  	left = middle+1
                else:
                  	right = middle-1
            return False
    ```

+ 升序列表中找两个数之和等于给定的目标值

  + 给出生序列表肯定是有它的意义的，如果头尾两数相加小于target，则我们知道，肯定是头节点往后，如果大于target，肯定是尾节点往前，

  + ```python
    class Process(object):
      	def solve(self, nums, target):
          	left,right = 0,len(nums)-1
            while(left<=right):
              	sum = nums[left]+nums[right]
                if sum<target:
                  	left += 1
                elif sum>target:
                  	right -= 1
                else:
                  return [left, right]
    ```

+ 给定一个数，判定该数是否可以表示成两个数的平方和

  + ```python
    class Process(object):
      	def solve(self, num):
          	left,right = 0, int(num**0.5)
            while(left<right):
              	sum = left**2 + right**2
                if sum<target:
                  	left += 1
                elif sum>target:
                  	right -= 1
                else:
                  return [left, right]
    ```

+ 回文字符串，可以删除一个字符串，在判定是否可以构成回文字符串

  + ```python
    class Process(object):
      	def solve(self, s):
          	left,right = 0, len(s)-1
            while(left<right):
              	if s[left]!=s[right]:
                  	return helper(s, left+1, right) | helper(s, left, right-1)
                left += 1
                right -= 1
            return True
          
        def helper(self, s, left, right):
          	while(left<right):
              	if s[left]!=s[right]:
                  	return False
                left += 1
                right -= 1
            return True
    ```

