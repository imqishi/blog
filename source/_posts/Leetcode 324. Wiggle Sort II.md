---
title: Leetcode 324. Wiggle Sort II
date: 2018-04-09 22:00:00
tags: [leetcode]
---

## 题目说明

Given an unsorted array `nums`, reorder it such that `nums[0] < nums[1] > nums[2] < nums[3]...`.

**Example:**
(1) Given `nums = [1, 5, 1, 1, 6, 4]`, one possible answer is `[1, 4, 1, 5, 1, 6]`. 
(2) Given `nums = [1, 3, 2, 2, 3, 1]`, one possible answer is `[2, 3, 1, 3, 1, 2]`.

**Note:**
You may assume all input has valid answer.

**Follow Up:**
Can you do it in O(n) time and/or in-place with O(1) extra space?

## 题目分析

这道题比较难，题目可以分成两个部分，一个是O(n)的时间复杂度寻找中位数**（TOP-K问题）**，一个是找到中位数以后对前半段和后半段进行交叉交换**（类比荷兰国旗问题）**。

### 首先来回顾一下TOP-K问题的方法

1. BFPRT算法
2. 堆排序
3. 利用快速排序Partition进行分割

这里我们复习一下第三种方案，好久不写快排了，不会写了！要注意的点有几个：

1. 内层的 while 循环判断的退出条件，一个是`i < j` 这个是必须的，第二个就是加不加等号的问题，因为我们选择的 pivot 是最后一个数字，所以当左边的游标找到 swap 值后右边的必须得往前走，不然就把 pivot 换走了，因此两个 while 内循环第一个不带等号而第二个带等号
2. 最后要把 i 和 hi 交换
3. m的计算，记得 +1，因为 i - lo 只是距离，第多少位还是得 +1
4. 对应的 k 的值，如果在 m 的右侧，这里直接减就可以了。因为 m 的算法是从1开始算的，第1个数开始而不是第0个数开始。

```java
public int quickSelect(int[] nums, int lo, int hi, int k) {
    int i = lo, j = hi, pivot = nums[hi];
    while (true) {
        while (i < j && nums[i] < pivot) {
            i ++;
        }
        while (i < j && nums[j] >= pivot) {
            j --;
        }
        if (i >= j) {
            break;
        }
        swap(nums, i, j);
    }
    swap(nums, i, hi);
    int m = i - lo + 1;
    if (m == k) {
        return i;
    } else if (m > k) {
        return quickSelect(nums, lo, i - 1, k);
    } else {
        return quickSelect(nums, i + 1, hi, k - m);
    }
}
```

### 然后回顾一下荷兰国旗问题

三种颜色，按照一定的顺序排列做到有序，使用O(n)时间复杂度。其思想是维持三个指针，一个指头，一个指尾，一个指中间。让中间的游标主动走。如果中间的游标走到的值比中位数小，那么头、中指针一起前进；如果中间指针走的值比中位数大，那么把这个数和尾指针的数交换，并且尾指针减1，**中间指针不动（精髓在此）**；否则只让中间指针前进。

```java
/*
 * Three Way Partition
 * Dutch National Flag Problem
 * Related: Leetcode 75
 * */
public void threeWayPartition(int[] a, int mid) {
    int i = 0, j = 0, n = a.length - 1;

    while (j <= n) {
        if (a[j] < mid) {
            swap(a, i, j);
            i ++;
            j ++;
        } else if (a[j] > mid) {
            swap(a, j, n);
            n --;
        } else {
            j ++;
        }
    }
    out.println(Arrays.toString(a));
}
```

### 最后说一下让人无语的置换

至今没咋看懂，套公式的话没问题，走一遍也是对的，但是对公式如何得来的还不太理解：

```java
public int newIndex(int index, int n) {
    return (1 + 2 * index) % (n | 1);
}
```

即把当前位的下标转换为 2 * index + 1 然后对 n 取余，如果n是奇数直接取余，否则如果是偶数那么取比它大一的奇数。

例如：

```
Original Indices:    0  1  2  3  4  5  6  7  8  9 10 11
Mapped Indices:      1  3  5  7  9 11  0  2  4  6  8 10
```

(its reverse mapping is)

```
Mapped Indices:      0  1  2  3  4  5  6  7  8  9 10 11
Original Indices:    6  0  7  1  8  2  9  3 10  4 11  5   (wiggled)
```

后来想了一下，是这样的：

如果要满足题意，我们可以把后半部分大的插入到前半部分，或者前半部分插入到后半部分。这个动作本质上就是坐标变换。

按照上例，先把本来的0 ~ 11分成两半，分别标偶数和奇数。插入的动作也就是按照Mapped Indices排序的过程。然后根据第二个来推由 Original 怎么得到 Mapped，就是那个 newIndex 函数做的事情。

### Finally

```java
public void wiggleSort(int[] nums) {
    /*
     * Assume that quickSelect use O(n) time
     * Actually we should use BFPRT algorithm
     * */
    int n = nums.length;
    int median = nums[n / 2];
    if (n > 1) {
        median = quickSelect(nums, 0, n - 1, (n + 1) / 2);
        median = nums[median];
    }
    int left = 0, i = 0, right = n - 1;

    while (i <= right) {
        if (nums[newIndex(i, n)] > median) {
            swap(nums, newIndex(left++, n), newIndex(i++, n));
        } else if (nums[newIndex(i, n)] < median) {
            swap(nums, newIndex(right--, n), newIndex(i, n));
        } else {
            i++;
        }
    }
}
```

