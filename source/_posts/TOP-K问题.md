---
title: TOP-K问题
date: 2018-01-22 21:23:00
tags: [leetcode]
---

在无序数组中寻找第K大或者第K小值的问题称为TOP-K问题。对于该问题我们通常有如下思路：

1. 排序，然后取第K大（小）值
2. 利用特殊数据结构，如优先队列（ Java Priority Queue ）设置合适的队列长度即可获得
3. BFPRT算法

前两种方法比较简单，容易想到。但是显然他们并不是最优的。对于方法1来讲，如果K是一个中间数，我们并不需要对所有的数字进行排序；对于方法2来讲，队列的长度与K值大小相关。因此，我们需要一个更6的方法来解决这件事情。

Blum、Floyd、Pratt、Rivest、Tarjan最终提出了解决该问题的好方法：BFPRT算法。通过该算法我们可以用最坏时间复杂度为O(n)的方式解决TOP-K问题。该算法是从快速排序算法的基础上演变而来的，我们知道，快速排序算法通过选取一个“主元”进行Partition操作，使一次调整后的结果达到主元左侧的数据都小于它，右侧数据都大于它的结果。因此，我们或许并不需要把左右侧的数据进行彻底排序，就可以获得第K个数是什么。如果这一遍排序后主元的位置并不是K，那么我们只需要对左侧或者右侧再次进行相应的Partition就可以了。

选取主元是对FBPRT算法来说很重要的一步，姿势对了才能让工作量尽量小的情况下找到TOP-K元素。选取方法如下：

1. 将n个元素划分为⌊n/5⌋个组，每组5个元素，若有剩余，舍去； 
2. 使用插入排序找到⌊n/5⌋个组中每一组的中位数； 
3. 对于（2）中找到的所有中位数，调用BFPRT算法求出它们的中位数，作为主元； 

之后，以选取的该主元为分界点，把小于主元的放在左边，大于主元的放在右边。最终判断主元的位置与k的大小，有选择的对左边或右边递归。[关于为什么要取5是有科学依据的，点击这里查看](https://www.61mon.com/index.php/archives/175/)

```python
class BFPRT(object):
    def go(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        numLen = len(nums)
        k = numLen - k + 1
        index = self.bfprt(nums, 0, numLen - 1, k)
        return nums[index]
    
    def insertSort(self, arr, left, right):
        for i in range(left + 1, right + 1):
            t = arr[i]
            j = i - 1
            while j >= left and arr[j] > t:
                arr[j + 1] = arr[j]
                j -= 1
            arr[j + 1] = t

        return ((right - left) / 2) + left;

    def getPivotIndex(self, arr, left, right):
        if right - left < 5:
            return self.insertSort(arr, left, right)

        i = j = left
        # every 5 elements as a team and find median
        # then put them to left
        while i + 4 <= right:
            index = self.insertSort(arr, i, i + 4)
            arr[j], arr[index] = arr[index], arr[j]
            j += 1
            i += 5
        
        return self.bfprt(arr, left, j, ((j - left + 1) / 2) + 1)
    
    def partition(self, arr, left, right, pivot_index):
        # put pivot element to end
        arr[pivot_index], arr[right] = arr[right], arr[pivot_index]

        # prepare for put nums which less than pivot to divide_index ++
        divide_index = left
        for i in range(left, right):
            if arr[i] < arr[right]:
                arr[divide_index], arr[i] = arr[i], arr[divide_index]
                divide_index += 1

        # re-put pivot to new-pivot pos
        # here left is all less than pivot and right is all more than pivot
        arr[divide_index], arr[right] = arr[right], arr[divide_index]

        return divide_index

    def bfprt(self, arr, left, right, k):
        pivot_index = self.getPivotIndex(arr, left, right)
        divide_index = self.partition(arr, left, right, pivot_index)
        num = divide_index - left + 1;

        if num == k:
            # find target
            return divide_index
        elif num > k:
            # find in left
            return self.bfprt(arr, left, divide_index - 1, k)
        else:
            # find in right
            return self.bfprt(arr, divide_index + 1, right, k - num)

obj = BFPRT()
print obj.go([1,5,4,6,2,3], 6)
```

相关题目：Leetcode 215
