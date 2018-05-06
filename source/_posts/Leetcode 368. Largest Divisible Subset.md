---
title: Leetcode 368. Largest Divisible Subset
date: 2018-04-27 17:16:00
tags: [leetcode,dp]
---

# 题目描述

给出一组不重复的数字，求一个最大的集合，使其中的数字任意两个之间都可以整除。如：

```
nums: [1,2,3]
Result: [1,2] (of course, [1,3] will also be ok)

nums: [1,2,4,8]
Result: [1,2,4,8]
```

# 题目分析

首先，1比较特殊，所有的数字都可以被1整除，所以我们要把1单独看。

然后，先将数字进行排序。从小到大看，如果已经有一组可以互相整除的数字，可以发现，如果一个新的数可以加入这个集合，那么它一定可以被这个集合的最大数和最小数整除，进一步说，能被最大的整除，那就能被最小的整除。

我们使用一个数组 dp[i] 表示如果 i 处位置在一个集合中，这个集合中的数字数量，容易得到如果一个之前被计算过位置 j 处的数字可以和这个数字整除，那么 dp[i] = dp[j] + 1，然后我们使用一个数组 parent[i] 记录 i 处取得最大 dp 值时它的上一元素的下标，最终可以得到整条路径。

需要注意的是，下面的代码选择了从后往前计算，这样计算有几个优点：首先是对于1的特殊处理可以跳过，因为1到最后才会被计算（如果有的话）；其次如果我们从前向后计算，最后得到的序列需要从后往前找，如果题目要求从小到大输出，就又得逆着写一遍（当然你非说没什么区别那也没关系...

# 代码

```java
public List<Integer> largestDivisibleSubset(int[] nums) {
    int[] dp = new int[nums.length];
    int[] parent = new int[nums.length];
    int m = 0, mi = 0;
    Arrays.sort(nums);

    for (int i = nums.length - 1; i >= 0; i --) {
        for (int j = i; j < nums.length; j ++) {
            if (nums[j] % nums[i] == 0 && dp[i] < dp[j] + 1) {
                dp[i] = dp[j] + 1;
                parent[i] = j;

                if (dp[i] > m) {
                    m = dp[i];
                    mi = i;
                }
            }
        }
    }

    List<Integer> list = new ArrayList<>();
    for (int i = 0; i < m; i ++) {
        list.add(nums[mi]);
        mi = parent[mi];
    }

    return list;
}
```

