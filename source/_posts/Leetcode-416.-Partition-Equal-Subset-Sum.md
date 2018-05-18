---
title: Leetcode 416. Partition Equal Subset Sum
date: 2018-05-17 20:15:00
tags: [leetcode,dp]
---

# 题目说明

给出一个数组 nums 求能否用里面的数字组合求和，使子数组之和为总和的一半。

# 分析

动态规划问题，背包问题的变形题目。开始的时候想要使用一个一维数组 dp[i] （0 <= i <= sum / 2）记录是否能得到 i，只需遍历一遍把可以直接得到的 dp 置为 true，然后再从 1 - sum/2 遍历一遍看 dp[sum/2] 是否为 true。dp[i] = dp[i] && dp[i-j] 如果dp[i] == true 则检查 i + 1。但是这种思路是错误的，因为数组中的数字只能被用一次，而我们的状态转移方程中明显没有考虑这一点，用 `[1, 2, 5]` 这组数字就可以证明。

那么我们给 dp 数组再加一维，使之变成 dp\[i][j] 表示利用前 i 个数可以构造 j。那么在进行计算的时候通过 i 就可以判断是不是重复使用了一些数字。显然有 dp\[0][0] = true，因为0不需要任何数字就可得到。对于 dp\[i][0] （i > 0) 来说始终都是 true，原因同上。对于 dp\[0][j] 初始化为 false，因为我们不可能不用数组中的数字就得到 j 这个数字。同时可以发现，如果前 i 个数字能得到 j，那么前 i + x (x >= 0) 个数字一定也能得到 j。换句话说，如果我不用第 i 个数，要看能否达到 j，只需看前 i-1 个数能否达到j：因此有第一个状态转移方程：`dp[i][j] = dp[i-1][j]` ；如果我用第 i 个数，要看能否达到 j，那么只需要看前 i - 1 个数能否构成 j-nums[i-1]（这里需要注意的是第 i 个数其实是 nums[i-1]，我们说的第1个数其实是nums[0]）：`dp[i][j] = dp[i-1][j-nums[i-1]]` ，当然如果 j 只要能达到话是肯定可以构成的，因此对上式用了逻辑或运算 dp\[i][j] 。

```java
public boolean canPartition(int[] nums) {
    int sum = 0;
    
    for (int num : nums) {
        sum += num;
    }
    
    if ((sum & 1) == 1) {
        return false;
    }
    sum = sum >> 1;

    int n = nums.length;
    boolean[][] dp = new boolean[n + 1][sum + 1];
    dp[0][0] = true;
    
    for (int i = 1; i < n + 1; i++) {
        dp[i][0] = true;
    }
    for (int j = 1; j < sum + 1; j++) {
        dp[0][j] = false;
    }
    
    for (int i = 1; i < n + 1; i ++) {
        for (int j = 1; j < sum + 1; j ++) {
            dp[i][j] = dp[i-1][j];
            if (j >= nums[i-1]) {
                dp[i][j] = (dp[i][j] || dp[i-1][j-nums[i-1]]);
            }
        }
    }
   
    return dp[n][sum];
}
```

分析最后的状态转移方程可以发现我们其实只用到了 dp[i] 和 dp[i-1] 这两行数据，因此可以把空间复杂度进一步降低。

```java
public boolean canPartition(int[] nums) {
    int sum = 0;
    
    for (int num : nums) {
        sum += num;
    }
    
    if ((sum & 1) == 1) {
        return false;
    }
    sum = sum >> 1;
    
    int n = nums.length;
    boolean[] dp = new boolean[sum+1];
    dp[0] = true;
    
    for (int num : nums) {
        for (int i = sum; i > 0; i--) {
            if (i >= num) {
                dp[i] = dp[i] || dp[i-num];
            }
        }
    }
    
    return dp[sum];
}
```

其实思路就是把 nums 中的数字逐个加入考虑了的划分中，从而考虑全部可以构成的数的值。

# 难

动态规划的题目还是难，能想出思路来就不容易，何况很多时候还可以进一步进行优化，优化后的代码可能可读性会比较差，这类问题要系统的学习还是建议去看 dd_engi 的背包九讲。

