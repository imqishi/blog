---
title: Leetcode 309. Best Time to Buy and Sell Stock with Colldown
date: 2018-04-03 22:00:00
tags: [leetcode,dp]
---

## 题目简述

给出一个一维数组，每位表示当天股票价格。可以多次买入和卖出，但是卖出和买入之间需要隔一天。必须要先买入才能卖出（这句是废话）。求可以达到的最大收入是多少。

## 问题分析

显然这是一道有约束条件的动态规划问题。我们先从最简单的思路入手，一步一步进行优化：

### 初步分析

既然有三种条件选择，那么我们分别给三种条件设定一个数组 x[i] 表示第 i 天前的序列以该选择作为结尾**（该选择后面还可以有rest）**可以获得的最大利润。有如下：

```java
int[] buy = new int[pricesLen];
int[] sell = new int[pricesLen];
int[] rest = new int[pricesLen];
```

然后我们逐个分析：

对于 `buy[i]` 来说，它代表的是第 i 天前最后操作是买入可以获得的最大利润，那么有三种情况取最大即可，一种是 i - 1 天前一直休息 第 i 天前买入， 一种是 i - 1 天前买入 i 天前不变，一种是 i - 2 天前卖出 i 天前买入（这种情况其实就是第一种情况，后面我们会对它进行合并）。因此有

```
buy[i] = Math.max(Math.max(rest[i-1] - prices[i], buy[i-1]), sell[i-2] - prices[i])
```

对于 `sell[i]` 来说，它代表的是第 i 天前最后动作是卖出（之后可以有rest）可以获得的最大利润，同样有两种情况，一种是 i - 1 天前买入 i 天前卖出，一种是 i - 1 天前卖出之后一直rest（于是这个值不会变，就是`sell[i-1]`）。因此有

```
sell[i] = Math.max(buy[i-1] + prices[i], sell[i-1])
```

对于 `rest[i]` 来说，它代表的是如果第 i 天休息可以获得的最大利润，这次有三种情况，即 i - 1 天前买入、卖出、休息的最大值。因此有

```
rest[i] = Math.max(Math.max(buy[i-1], sell[i-1]), rest[i-1])
```

由此可得到第一版代码，注意初始条件

```java
public int maxProfit(int[] prices) {
    int pricesLen = prices.length;
    if (pricesLen < 2) {
        return 0;
    }
    buy[0] = - prices[0];
    buy[1] = Math.max(rest[0] - prices[1], buy[0]);
    sell[1] = Math.max(buy[0] + prices[1], 0);
    for (int i = 2; i < pricesLen; i ++) {
        buy[i] = Math.max(Math.max(rest[i-1] - prices[i], buy[i-1]), sell[i-2] - prices[i]);
        sell[i] = Math.max(buy[i-1] + prices[i], sell[i-1]);
        rest[i] = Math.max(Math.max(buy[i-1], sell[i-1]), rest[i-1]);
    }
    
    return Math.max(rest[pricesLen - 1], sell[pricesLen - 1]);
}
```

### 进一步分析

如果 i-1 天前最后买入花了钱，那么他的盈利肯定比 i-1 天前最后是休息少。因此有：`buy[i-1] <= rest[i-1]`  而第 i 天（前）买入要花钱，第 i - 1 天（前）买入也要花钱，所以 `buy[i] <= buy[i-1]` 所以进一步有：`rest[i] = max(sell[i-1], rest[i-1])` 

同时，第 i-1 天休息肯定比第 i-1 天卖出的（最大期望）盈利要小，因此有 `rest[i-1] <= sell[i-1]`

由以上条件可以得知：

```
rest[i] = sell[i-1]
```

因此可以把第一步分析的 `rest[i-1]` 条件用 `sell[i-2]` 替换，得到新的代码如下

```java
public int maxProfit(int[] prices) {
    int pricesLen = prices.length;
    if (pricesLen < 2) {
        return 0;
    }
	int[] buy = new int[pricesLen];
    int[] sell = new int[pricesLen];
    int[] rest = new int[pricesLen];
    // second version
    buy[0] = - prices[0];
    buy[1] = Math.max(rest[0] - prices[1], buy[0]);
    sell[1] = Math.max(buy[0] + prices[1], 0);
    for (int i = 2; i < pricesLen; i ++) {
        buy[i] = Math.max(sell[i-2] - prices[i], buy[i-1]);
        sell[i] = Math.max(buy[i-1] + prices[i], sell[i-1]);
    }

    return sell[pricesLen - 1];
}
```

这里要注意的是对 1 位置的初始化中 `sell[1]` 应该初始化为 1 的时候卖出可以获得的最大利润

### 最后的优化

我们可以看到，其实我们对于 `sell` 和 `buy` 数组用到的只是 `i-1` 和 `i-2` 处的值，因此并不需要用一个数组记录它，只需要有四个额外的变量就可以了，由此得到最终代码

```java
public int maxProfit(int[] prices) {
    int pricesLen = prices.length;
    if (pricesLen < 2) {
        return 0;
    }
    int sell = 0, prev_sell = 0, buy = Integer.MIN_VALUE, prev_buy;
    for (int price : prices) {
        prev_buy = buy;
        buy = Math.max(prev_sell - price, prev_buy);
        prev_sell = sell;
        sell = Math.max(prev_buy + price, prev_sell);
    }

    return sell;
}
```

## 小结

本题目卡住的主要问题在于以前的题目都是简单的用一个一维或者二维数组可以进行状态表示的，而本题需要用三个数组（不优化的状态下）才能比较直观的进行状态的确定与状态转移方程的推导。并且后续的优化需要利用题目的隐藏条件（如卖出前必须有买入，买入后不能再买入…）进行。还需要多学习总结动态规划问题对于状态表示的套路。
