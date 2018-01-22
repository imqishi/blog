---
title: 快速排序中Partition的两种算法
date: 2017-09-13 21:59:45
tags: [leetcode,sort]
---

### 相关题目

Leetcode 75 Sort Colors

### 题目大意

三种颜色，标记为0，1，2，按照升序进行排序。如：`2,1,2,0,0` 经过排序后的结果为 `0,0,1,2,2`

**要求：在原数组上进行排序操作，仅经过一次遍历，常数级空间复杂度**

### 分析

题目仅对三个数字进行排序为O(n)时间复杂度提供了机会：考虑快速排序中的Partition算法，正是以 pivot 为基准，将一个数组从某个位置进行划分，左侧均为比pivot小的，右侧均为比pivot大的。正与题目要求相呼应。

### 两种算法

以前上数据结构的时候曾经学过快排，不过当时课本上只介绍了一种方法，就是**一个游标i从左往右寻找比pivot大的，另一个游标 j 从右往左寻找比pivot小的，将两个数字不断交换，直至 i 和 j 相遇循环结束。最后，记得交换 j 位置的元素和最开始的元素**。

```python
def hoarePartition(nums, lo, hi):
    pivot = nums[lo]
    i, j = lo, hi
    while True:
    	while nums[i] < pivot and i <= hi:
         	i += 1
        while nums[j] >= pivot and j >= lo:
        	j -= 1
        if i >= j:
          	print nums
        	return j
        nums[i], nums[j] = nums[j], nums[i]
    nums[lo], nums[j] = nums[j], nums[lo]
```

这种方法为什么不能用在题目中呢？或者说能不能用呢？我也没有想出来...求大佬解释，我的邮箱 imqishi#hotmail.com。 一种不成熟的解释方法是，Hoare算法是双向迭代的，这就会导致不管从0开始还是从2开始，一旦走了就没办法回头了，因为我们并不知道这串数字里会不会有0或者2，假设没有2，你把最后一个置成2，然后往前继续搜，最后发现2放多了，得置成1或者0，这就不是一遍遍历了。因此想要一遍遍历必需得固定住一头，从另一头走过来，边走边覆盖值。也就是借助下面Partition算法的思想。

看答案解析的时候发现这种算法叫Hoare Partition，还有另一个Lomuto Partition，方法是：**i 指示最前面的大于 pivot 的元素位置，j 从前往后滑动来调整元素位置。每次 j 碰到小于 pivot 的元素，则 i 位置的元素和 j 位置的元素交换，i 指向下一个大于 pivot 的元素。最后，记得交换 i 位置的元素和最末尾的元素**。

```python
def lomutoPartition(nums, lo, hi):
    pivot = nums[hi]
    i = lo
    j = lo
    while j < hi:
      	if nums[j] <= pivot:
          	nums[i], nums[j] = nums[j], nums[i]
            i += 1
    	j += 1
   	
    nums[i], nums[hi] = nums[hi], nums[i]
    print nums
    return i
```

通过这种方法，可以看出 i 的位置始终是比 pivot 大的从左往右数的第一个数的位置。这样的话就很容易去进行数字的覆盖，按照这种思路，可以得到解决方案如下，形成一个 0~i~j~n的三个范围，分别对应了0，1，2：

```python
class Solution(object):
    def sortColors(self, nums):
        """
        :type nums: List[int]
        :rtype: void Do not return anything, modify nums in-place instead.
        """
        i = j = 0
        for k in range(len(nums)):
            v = nums[k]
            nums[k] = 2
            if v < 2:
                nums[j] = 1
                j += 1
            if v == 0:
                nums[i] = 0
                i += 1
        
        print nums
```

### 既然写到这里了

顺路带一下快速排序的入口代码：

```python
def quick_sort(arr, left, right) {
    if(left > right)
        return;
    j = partition(arr, left, right);
    quick_sort(arr, left, j - 1);
    quick_sort(arr, j + 1, right);

```

