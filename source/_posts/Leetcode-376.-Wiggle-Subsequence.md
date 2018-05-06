---
title: Leetcode 376. Wiggle Subsequence
date: 2018-05-10 12:44:00
tags: [leetcode]
---

# 题目

把一个波动序列定义为：如果相连的数字两两的差值为正数、负数交替。比如  `1 7 4 9 2 5` 就是一个波动数列。注意，如果只有两个不同数字，那么我们认为这也是一个波动序列。

给出一个数组，求里面最长的波动子序列长度，注意这个子序列中的数字在原数组中并不一定是连续的。如

```
Input: [1,7,4,9,2,5]
Output: 6
The entire sequence is a wiggle sequence.

Input: [1,17,5,10,13,15,10,5,16,8]
Output: 7
There are several subsequences that achieve this length. One is [1,17,10,13,10,16,8].

Input: [1,2,3,4,5,6,7,8,9]
Output: 2
```

# 分析

只走一遍取得最大值，首先想到的是贪心的思想，每找到一个符合的数字就加入到子序列中，当不符合的时候就把子序列中的最后一个数字改为当前数字，走到最后的时候的长度就是答案。

反问一下，这真的是答案吗？开始的时候我也对这个思路产生了怀疑。会不会有这种情况出现，比如第一对数字符合波动的要求，但是他们和之后的数字“不兼容”，也即后面存在一个更长的波动子序列，与前面的无法组合匹配，因为我们是从前往后开始计算子序列的，所以首先要考虑的就是这种情况是否存在。

假设这种情况存在，也即存在 p q x y z 五个数字，满足下列条件：

1. p 和 q 组成长度为2的波动数， x y z 组成长度为3的波动数且两个波动数不能再发生组合
2. p > q
3. x \< y > z
4. p < x（ p < x 一定有 p < y）
5. p < z

我们分析一下就可以发现这种情况是不可能存在的，因为 q < x < y 那么 p q y z 就可以组合成一个更长的。

更为通用的分析是，假设 x \< y > z 已经存在，我们往 x 前面加一个数字 p：

1. 如果 p < x，那么 p y z 就是一个相同长度的子序列
2. 如果 p > x，那么p x y z 就是一个更长的子序列

因此，第一个数字是一定会包含在子序列中的，这也证明了为什么我们可以从前两个数字作为初始情况计算。

# 代码

```java
public int wiggleMaxLength(int[] nums) {
    if (nums.length <= 1) {
        return nums.length;
    }
    int k = 0;
    while (k < nums.length - 1 && nums[k] == nums[k + 1]) {
        k ++;
    }
    if (k == nums.length - 1) {
        return 1;
    }

    int res = 2;
    boolean needBig = nums[k] < nums[k + 1];
    for (int i = k + 1; i < nums.length - 1; i ++) {
        if (needBig && nums[i] > nums[i + 1]) {
            nums[res] = nums[i + 1];
            res ++;
            needBig = ! needBig;
        } else {
            if (! needBig && nums[i] < nums[i + 1]) {
                nums[res] = nums[i + 1];
                res ++;
                needBig = ! needBig;
            }
        }
    }
    return res;
}
```

