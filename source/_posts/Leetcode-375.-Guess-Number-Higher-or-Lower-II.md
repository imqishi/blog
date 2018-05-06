---
title: Leetcode 375. Guess Number Higher or Lower II
date: 2018-05-04 22:08:00
tags: [leetcode，dp]
---

# 题目

玩一个猜数字的游戏，给一个数字 n 表示需要猜的数字范围在 1 - n。每当你猜一个数字，一旦猜错了，就需要支付所猜错的数字那么多的钱，如果猜中了则不需要付钱。求至少要有多少钱才能确保赢这个游戏。

# 分析

其实这个题目Leetcode上给出了太多繁杂的例子，并且例子只是举了一个如何交钱的场景，并不是真正的解，所以不要被误导了...

因为我们并不知道会选中哪个数字，所以其实我们要算的是所有最多的情况里最小的（有点像阿里方块手拉手的那个题目）

使用一个数组 dp\[i][j] 表示保证 i 到 j 位置能获胜所需要的最少钱数，那么 dp\[1][n] 就是给出 n 范围时要赢游戏所需要的最少钱数。距离为0的时候 dp 为 0，因为这种情况说明要猜的数字与范围相同（就是 i），因此花费为0。我们从距离为 1 开始计算，一直推到距离为整个长度为止。

猜数推断 i - j 的过程可以以中间点 k 为分界线分解为前后两个部分取最大，因此可以得知状态转移方程中关于中间过程的计算公式：

```
k + Math.max(dp[i][k - 1], dp[k + 1][j])
```

那么我们要求的是 k 在 i - j 位置中使这个值最小的，因此状态转移方程为：

```
dp[i][j] = Math.min(dp[i][j], k + Math.max(dp[i][k - 1], dp[k + 1][j]))
```

需要注意的是 k 为 i 或 j 的时候，dp\[i][i-1] = 0，dp\[j+1][j] = 0。

# 代码

```java
public int getMoneyAmount(int n) {
    if (n == 1) {
        return 0;
    }
    int[][] dp = new int[n + 1][n + 1];
    for (int len = 1; len < n; len ++) {
        for (int i = 0; i + len <= n; i ++) {
            int j = i + len;
            dp[i][j] = Integer.MAX_VALUE;
            for (int k = i; k <= j; k ++) {
                dp[i][j] = Math.min(dp[i][j], k + Math.max(k - 1 >= i ? dp[i][k-1] : 0, j >= k + 1 ? dp[k+1][j] : 0));
            }
        }
    }

    return dp[1][n];
}
```

