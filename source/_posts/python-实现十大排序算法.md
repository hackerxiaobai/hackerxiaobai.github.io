---
title: python 实现十大排序算法
date: 2020-09-09 21:46:15
tags:
	- 排序算法
	- python
---

```python
def bubbleSort(nums):
    """
    冒泡排序
    :param nums:
    :return:
    """
    for i in range(len(nums)):
        for j in range(1, len(nums) - i):
            if nums[j - 1] > nums[j]:
                nums[j - 1], nums[j] = nums[j], nums[j - 1]

    return nums
```
<!-- more -->

```python
def selectionSort(nums):
    """
    选择排序
    :param nums:
    :return:
    """
    for i in range(len(nums)):
        minIndex = i
        for j in range(i + 1, len(nums)):
            if nums[j] < nums[minIndex]:
                minIndex = j
        nums[i], nums[minIndex] = nums[minIndex], nums[i]
    return nums
```

```python
def insertSort(nums):
    """
    插入排序
    :param nums:
    :return:
    """
    for i in range(1, len(nums)):
        curNum, preIndex = nums[i], i
        while preIndex > 0 and curNum < nums[preIndex - 1]:
            nums[preIndex] = nums[preIndex - 1]
            preIndex -= 1
        nums[preIndex] = curNum
    return nums
```

```python
def quickSort(nums):
    """
    快速排序
    :param nums:
    :return:
    """
    if len(nums) <= 1:
        return nums
    pivot = nums[0]  # 基准值
    left = [nums[i] for i in range(1, len(nums)) if nums[i] < pivot]
    right = [nums[i] for i in range(1, len(nums)) if nums[i] >= pivot]
    return quickSort(left) + [pivot] + quickSort(right)
```

```python
def heapSort(nums):
    """
    堆排序
    :param nums:
    :return:
    """

    # 调整堆
    def adjustHeap(nums, i, size):
        # 非叶子结点的左右两个孩子
        lchild = 2 * i + 1
        rchild = 2 * i + 2
        # 在当前结点、左孩子、右孩子中找到最大元素的索引
        largest = i
        if lchild < size and nums[lchild] > nums[largest]:
            largest = lchild
        if rchild < size and nums[rchild] > nums[largest]:
            largest = rchild
        # 如果最大元素的索引不是当前结点，把大的结点交换到上面，继续调整堆
        if largest != i:
            nums[largest], nums[i] = nums[i], nums[largest]
            # 第 2 个参数传入 largest 的索引是交换前大数字对应的索引
            # 交换后该索引对应的是小数字，应该把该小数字向下调整
            adjustHeap(nums, largest, size)

    # 建立堆
    def builtHeap(nums, size):
        for i in range(len(nums) // 2)[::-1]:  # 从倒数第一个非叶子结点开始建立大根堆
            adjustHeap(nums, i, size)  # 对所有非叶子结点进行堆的调整
        # print(nums)  # 第一次建立好的大根堆

    # 堆排序
    size = len(nums)
    builtHeap(nums, size)
    for i in range(len(nums))[::-1]:
        # 每次根结点都是最大的数，最大数放到后面
        nums[0], nums[i] = nums[i], nums[0]
        # 交换完后还需要继续调整堆，只需调整根节点，此时数组的 size 不包括已经排序好的数
        adjustHeap(nums, 0, i)
    return nums  # 由于每次大的都会放到后面，因此最后的 nums 是从小到大排列

```

```python
def shellSort(nums):
  	"""
    希尔排序
    :param nums:
    :return:
    """
    lens = len(nums)
    gap = 1  
    while gap < lens // 3:
        gap = gap * 3 + 1  # 动态定义间隔序列
    while gap > 0:
        for i in range(gap, lens):
            curNum, preIndex = nums[i], i - gap  # curNum 保存当前待插入的数
            while preIndex >= 0 and curNum < nums[preIndex]:
                nums[preIndex + gap] = nums[preIndex] # 将比 curNum 大的元素向后移动
                preIndex -= gap
            nums[preIndex + gap] = curNum  # 待插入的数的正确位置
        gap //= 3  # 下一个动态间隔
    return nums
```

