---
title: '旋转排序数组系列问题'
date: 2020-03-31 16:19:24
tags: [排序,leetcode]
categories: algorithm
published: true
hideInList: false
feature: 
isTop: false
---
本文涉及 4 道「搜索旋转排序数组」题：

[LeetCode 33 题](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)：搜索旋转排序数组
[LeetCode 81 题](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii/)：搜索旋转排序数组-ii
[LeetCode 153 题](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)：寻找旋转排序数组中的最小值
[LeetCode 154 题](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)：寻找旋转排序数组中的最小值-ii

<!-- more -->

## 搜索旋转排序数组

假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。

搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。

你可以假设数组中不存在重复的元素。

你的算法时间复杂度必须是 O(log n) 级别。

示例 1:

> 输入: nums = [4,5,6,7,0,1,2], target = 0
> 输出: 4

示例 2:

> 输入: nums = [4,5,6,7,0,1,2], target = 3
> 输出: -1

要求时间复杂度为O(log n)，所以必须要用二分法

首先我们应该想到的是将这样的一个旋转排序数组二分后的两个数组分别有什么特点

1. 一个是排序数组，左端点小于右端点；
2. 一个是旋转排序数组，左端点小于右端点；

代码如下

```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        i=0
        j=len(nums)-1
        while i<=j:
            mid=(i+j)//2
            if nums[mid]==target:
                return mid
            if nums[i]<=nums[mid]:
                if nums[i]<=target<nums[mid]:
                    j=mid-1
                else:
                    i=mid+1
            else:
                if nums[mid]<target<=nums[j]:
                    i=mid+1
                else:
                    j=mid-1
        return -1
```



## 搜索旋转排序数组-ii

这题与上题基本一致，但数组可能存在重复元素

如何处理上一题中，如果出现左右端点相等的情况呢：

>1. 一个是排序数组，左端点小于右端点；
>2. 一个是旋转排序数组，左端点小于右端点；

可以预想的结果有如下三种例外情况

1. Start == Middle == End 将起始和结尾都缩短 即Start++，End++再分析
2. Start == Middle < End or Start < Middle == End 根据有小于那一段是排序数组进行搜索
3. Start == Middle > End or Start > Middle == End 根据有大于那一段是旋转排序数组进行搜索

### 

```Python
class Solution:
    def search(self, nums: List[int], target: int) -> bool:        
        l=0
        r=len(nums)-1
        while(l<=r):
            mid=(l+r)//2
            if(nums[mid]==target):
                return True
            if(nums[mid]==nums[l]==nums[r]):
                l+=1
                r-=1
            elif(nums[mid]>=nums[l]):
                if(nums[l]<=target<nums[mid]):
                    r=mid-1
                else:
                    l=mid+1
            else:
                if(nums[mid]<target<=nums[r]):
                    l=mid+1
                else:
                    r=mid-1
        return False
```



## 寻找旋转排序数组中的最小值

假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。

请找出其中最小的元素。

你可以假设数组中不存在重复元素。

示例 1:

>输入: [3,4,5,1,2]
输出: 1

示例 2:

>输入: [4,5,6,7,0,1,2]
输出: 0

### 暴力搜索法

一种很简单的方法是暴力搜索法，即从数组的开始向数组末端搜索，出现的第一个小于它前一个元素的元素即为该数组中的最小数组：

```Python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        if not nums:return None
        for i in range(1,len(nums)):
            if nums[i] <= nums[i-1]: return nums[i]
        return nums[0]
```

但是这种解法并不是最优解

题目中没有提到数组的长度，若数组过长，该方法可能行不通。

比较好的方法是二分法

### 二分法

很容易想到使用递归或者循环的方式：下面使用循环

```Python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        left = 0
        right = len(nums) - 1
        while left < right:
            mid = (left + right) >> 1
            if nums[mid] > nums[right]:         
                left = mid + 1
            else:                               
                right = mid
        return nums[left]
```



## 寻找旋转排序数组中的最小值-ii

同样也是上一题的升级版，这里和`搜索旋转排序数组-ii`解法一致，给出二分法的代码。

```Python
class Solution:
    def findMin(self, nums) -> int:
        left = 0
        right = len(nums) - 1
        while left < right:
            mid = (left + right) >> 1
            if nums[left] == nums[right] == nums[mid]:
                left = left + 1
                right = right - 1
            elif nums[mid] > nums[right]:         
                left = mid + 1
            else:                               
                right = mid
        return nums[left]
```





