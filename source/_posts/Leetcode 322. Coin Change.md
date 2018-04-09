---
title: Leetcode 322. Coin Change
date: 2018-04-09 22:00:00
tags: [leetcode,dp]
---

给出无限量的硬币面额和目标值，求如何用最少的硬币得到目标值。如果不存在这种组合，返回-1。

例如：

```
coins = [1, 2, 5], amount = 11
return 3 (11 = 5 + 5 + 1)

coins = [2], amount = 3
return -1
```

最经典最原始的动态规划问题，使用一个一维数组 `dp[i] (0 <= i <= amount)` 表示达到目标值为 i 需要的最少硬币数。那么可以得到状态转移方程：

`dp[i] = min(dp[i-coins[j]] + 1, dp[i])`

可得代码如下：

```java
public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        for (int i = 1; i <= amount; i ++) {
            dp[i] = Integer.MAX_VALUE;
        }
        Arrays.sort(coins);

        for (int i = 1; i <= amount; i ++) {
            for (int j = 0; j < coins.length; j ++) {
                if (coins[j] <= i && i - coins[j] >= 0 && dp[i-coins[j]] != Integer.MAX_VALUE) {
                    dp[i] = Math.min(dp[i-coins[j]] + 1, dp[i]);
                }
            }
        }
        if (dp[amount] == Integer.MAX_VALUE) {
            return -1;
        } else {
            return dp[amount];
        }
    }
```

