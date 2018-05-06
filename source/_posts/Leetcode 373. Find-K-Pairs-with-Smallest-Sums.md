---
title: Leetcode 373. Find K Pairs with Smallest Sums
date: 2018-05-03 20:16:00
tags: [leetcode]
---

# 题目描述

给出两个排序后的数组，求 k 个数字对，要求数字对的两个数字和是从小到大递增的顺序，且数字对中的数字第一个来自第一个数组，第二个来自第二个数组。例如：

```
Given nums1 = [1,7,11], nums2 = [2,4,6],  k = 3

Return: [1,2],[1,4],[1,6]

The first 3 pairs are returned from the sequence:
[1,2],[1,4],[1,6],[7,2],[7,4],[11,2],[7,6],[11,4],[11,6]


Given nums1 = [1,1,2], nums2 = [1,2,3],  k = 2

Return: [1,1],[1,1]

The first 2 pairs are returned from the sequence:
[1,1],[1,1],[1,2],[2,1],[1,2],[2,2],[1,3],[1,3],[2,3]
```

# 分析

第一感觉是给两个数组各一个游标，从小往大试。但是这样的话会导致第一个数组的游标一旦走到后面，什么时候应该回位到前面，以及应该回到前面哪个位置和第二个游标对应的构成对不重的问题都不好解决。

其实仔细想一下，这个题目有点像TOP-K问题的变形，只不过是对两个数字和求TOP-K小的。那么我们可以用一个大小为K的小根堆来解决该问题。

**首先是初始化堆的问题。**

我们随机选择一个数组的前K位数字与另一个数组的第一个数字组合建堆，这样做的原因是最多的情况下，一个数组的K个数字被使用到求和计算中。因此我们只需要先强制把其中一个数组前K个数字计算入堆中，对第二个数组进行向后的K次计算，就一定可以求得TOP-K。

如果选的这个数组不够K个怎么办呢？那就把它所有的数字都算进去，因为按照最多的可能计算，每个数字都可能被用到。

还要注意的一点是如果两个数组长度超过了K，那么我们最多给出两个数组的乘积个数字对。相应的，如果K为0或者有一个数组长度是0，我们直接返回一个空数组即可。

**然后我们继续考虑堆中应该存放的信息有哪些。**

两个数组中各选出的数字是必须的，但是还要**维护一个第二个数组走到的位置**，因为我们的堆是按照两数字和的递增顺序建的，这样就**无法保证加和后所取到的头元素所对应的第二个元素所在的位置**（尤其是当数组中出现重复数字的时候，我们不知道它是第几个出现的数字，根据题目的示例，一个相同的数对是可以出现多次的）

综上，我们可以得到代码：

```java
public List<int[]> kSmallestPairs(int[] nums1, int[] nums2, int k) {
    PriorityQueue<int[]> queue = new PriorityQueue<>((a, b) -> a[0] + a[1] - b[0] - b[1]);
    List<int[]> list = new ArrayList<>();

    if (k > nums1.length * nums2.length) {
        k = nums1.length * nums2.length;
    }
    if (k == 0) {
        return list;
    }

    for (int i = 0; i < nums1.length && i < k; i ++) {
        queue.offer(new int[]{ nums1[i], nums2[0], 0 });
    }

    while (k-- > 0 && !queue.isEmpty()) {
        int[] cur = queue.poll();
        list.add(new int[]{ cur[0], cur[1] });
        if (cur[2] == nums2.length - 1) {
            continue;
        }
        queue.offer(new int[]{ cur[0], nums2[cur[2]+1], cur[2] + 1 });
    }

    return list;
}
```

