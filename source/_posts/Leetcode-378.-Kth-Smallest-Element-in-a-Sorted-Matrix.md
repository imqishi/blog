---
title: Leetcode 378. Kth Smallest Element in a Sorted Matrix
date: 2018-05-06 16:14:00
tags: [leetcode]
---

# 题目要求

给出一个 n x n 矩阵，横向和纵向都是严格的递增序，求第k小的元素。

## 思路1 大根堆

我们知道大根堆可以得到从小到大的一个序列，那么只需要构造一个大小为 k 的大根堆，把所有元素加进去，然后 poll k 次即可。此法比较暴力，直接用 PriorityQueue 传一个 lambda 表达式 `(a, b) -> b - a` 然后都进堆最后得到堆顶元素即可。

## 思路2 小根堆

其实我们完全不需要把所有的元素都加到堆里去，分析给的数组我们可以发现一个规律，我们把数字按照列来看，每一列的下面的数字都小于上面的数字，那么我们只需要先把每列最小的数字（第一行的所有数字）构建一个小根堆，然后每次把堆顶元素弹出，并把堆顶元素对应的那一列的游标向下移动一位并入堆（前提是这一列还有数字）只需进行上述过程 k - 1 次得到（因为把第一行全部载入本身有1次向堆offer的过程）

使用这种方式需要注意的是，我们需要记录存在小根堆的元素的行和列值，以便进一步进行更新。

```java
class Tuple implements Comparable<Tuple> {
    int x, y, val;
    public Tuple(int x, int y, int val) {
        this.x = x;
        this.y = y;
        this.val = val;
    }

    @Override
    public int compareTo(Tuple o) {
        return this.val - o.val;
    }
}
public int kthSmallest(int[][] matrix, int k) {
    PriorityQueue<Tuple> heap = new PriorityQueue<>();
    for (int j = 0; j < matrix.length; j ++) {
        heap.offer(new Tuple(0, j, matrix[0][j]));
    }

    for (int i = 0; i < k - 1; i ++) {
        Tuple t = heap.poll();
        if (t.x == matrix.length - 1) {
            continue;
        }
        heap.offer(new Tuple(t.x + 1, t.y, matrix[t.x + 1][t.y]));
    }

    return heap.poll().val;
}
```

## 思路3 二分搜索

这里的二分搜索与普通的不太一样，不是以下标作为游走标准，而是以是不断取当前段的最大数字和最小数字的平均数作为判断的标准，利用该数数对每一行做一个划分，这样的话可以得到比它大的有几个，小的有几个，我们要做的就是让小的个数等于k。这里写了个简单的方法，最里层的while循环可以改为利用二分的方式查找可以更快。

我们采用实际值作为判断标准，那么最后得到的结果会不会不在矩阵中呢？答案是不会的，要注意程序中循环的结束条件并不是统计个数等于k，而是l < r，这意味着即便小于等于中值的元素个数刚好是k，我们也会继续循环，这样做的目的是让 lo 和 hi 逼近矩阵中存在的第k个元素。

```java
public int kthSmallestBinarySearch(int[][] matrix, int k) {
    int lo = matrix[0][0], hi = matrix[matrix.length - 1][matrix[0].length - 1];
    while (lo < hi) {
        int mid = lo + hi / 2;
        int count = 0, j = matrix[0].length - 1;
        for (int i = 0; i < matrix.length; i ++) {
            while (j >= 0 && matrix[i][j] > mid) {
                j --;
            }
            count += (j + 1);
        }
        if (count < k) {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }

    return lo;
}
```

# Leetcode 287. Find the Duplicate Number

这个题目的其实可以用上题中二分的思想来做，方法类似。因为题目说明了有 n+1 个元素，并且元素范围是 1 - n 的。那么我们可以从 1 和 n 的均值开始计算，假设均值是5，我们遍历数组查找小于等于5的元素个数，如果超过了5个，根据离散数学中学习的鸽巢原理，必然有一个重复数字，那么我们就把范围缩小继续如上过程，直到最后 low 和 high 逼近到重复的那个数字为止结束。 

```python
class Solution(object):
    def findDuplicate(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        low = 1
        high = len(nums)-1
        
        while low < high:
            mid = low+(high-low)/2
            count = 0
            for i in nums:
                if i <= mid:
                    count+=1
            if count <= mid:
                low = mid+1
            else:
                high = mid
        return low
```

## 其他方法

发现这个题之前没整理过，那就再说一下时间和空间都是O(n)的方法吧，比较投机取巧。

把这个问题进行一个转化 [参考回答](https://leetcode.com/problems/find-the-duplicate-number/discuss/72845/Java-O(n)-time-and-O(1)-space-solution.-Similar-to-find-loop-in-linkedlist.) 其实变成了一个链表判圈找交点的问题

```java
public int findDuplicate3(int[] nums)
{
	if (nums.length > 1) {
		int slow = nums[0];
		int fast = nums[nums[0]];
		while (slow != fast) {
			slow = nums[slow];
			fast = nums[nums[fast]];
		}

		fast = 0;
		while (fast != slow) {
			fast = nums[fast];
			slow = nums[slow];
		}
		return slow;
	}
	return -1;
}
```

# 相关题目

{% post_link Leetcode 373. Find-K-Pairs-with-Smallest-Sums %}