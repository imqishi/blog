---
title: Leetcode 62&63 - Unique Paths
date: 2017-09-09 18:50:00
tags: [leetcode,dp]
---

这两道题目是类似的，大意是给你一个m x n的棋盘，问从左上角到右下角共有多少种走法。只不过第二个题目引入了墙的概念，也即有墙的地方不能走。

前两天做了好几个**求所有具体情况**的问题写了好几个DFS，今天也没仔细看题直接DFS跑起来了...结果上来就超时了…然后画了几个图感觉是个数学题，总感觉有规律，然而找不到...瞄了眼答案的确是数学题，不过也可以用DP来做…我感觉我脑回路可能真和一般人不一样，这题A率接近50%我特喵的就是没想起来怎么DP...看答案的DP式那么简单我还懵了几秒钟...简直哔了狗...

Ps：求数量一般都是有更容易的方法，比如数学题，比如DP。而求具体的所有情况一般才是遍历、DFS等。

以下先以第一道题为例进行分析：

### 法一：万年的DP不变的真理

我们按照从左上到右下的目标，并且只能往右或者往下走，不能回头。那么一旦我们走到一个位置，这个位置的路径数就确定了，不管后面我们往哪个方向走，都不会对这个数字产生影响（无后效性），因此可以使用动态规划算法解决。

定义二维数组 `dp[i][j]` 表示到达该位置的路径数量。那么显然要到 `[i][j]` 的位置要么从左边过来要么从上边过来，因此有： `dp[i][j] = dp[i-1][j] + dp[i][j-1]` 

初始条件：对于第一行和第一列，我们初始化这些位置为1，其他地方为0。有这两个条件，可以得到源代码如下：

```python
class Solution(object):
    def uniquePaths(self, m, n):
        """
        :type m: int
        :type n: int
        :rtype: int
        """
        # init dp, when i or j is 0 val = 1 else 0
        dp = []
        for i in range(m):
            dp.append([])
            for j in range(n):
                if i == 0:
                    dp[i].append(1)
                elif j == 0:
                    dp[i].append(1)
                else:
                    dp[i].append(0)

        i = 1
        while i < m:
            j = 1
            while j < n:
                dp[i][j] = dp[i-1][j] + dp[i][j-1]
                j += 1
            i += 1

        return dp[m-1][n-1]
```

到这里我们可以看到，虽然我们用了一个m x n的数组记录每个点的路径数，但是其实我们只用了上一次和这一次两轮的状态，所以可以将空间复杂度进行优化，只用两个一维的数组记录状态：

```python
class Solution(object):
  	def uniquePathsDP1(self, m, n):
        if m > n:
            m, n = n, m
        cur = [1] * m
        last = [1] * m
        j = 1
        while j < n:
            i = 1
            while i < m:
                cur[i] = cur[i-1] + last[j]
                i += 1
            cur, last = last, cur
            j += 1

        return last[m-1]
```

*这里要注意的是，我们的循环把 i 和 j 换过来了，因为我们是以列为基准往右推的。所以应该先保持列不动（j循环在外层而i在内层）*

再仔细看看，设了两个一维数组的目的是进行上一次状态的保存。而我们完全可以利用一个数组来解决这个问题。因为递推计算的过程是有序的，并且用的是上轮的数据，不会对未来产生影响。我们要加的是左侧和上侧的数值，上侧就是 dp[i-1]，而左侧其实就是 dp[i]，因此有如下优化：

```python
class Solution(object):
  	def uniquePathsDP2(self, m, n):
        if m > n:
            m, n = n, m
        dp = [1] * m
        j = 1
        while j < n:
            i = 1
            while i < m:
                dp[i] += dp[i-1]
                i += 1
            j += 1
        
        return dp[m-1]
```

### 法二：数学问题

引用一下 [这个解答](https://discuss.leetcode.com/topic/5864/simple-c-version-using-math/2)

每个路径都要走 m+n-2 步。也就意味着不管是怎么绕，都要往下走 m 步，往右走 n-1 步。那么这就变成了一个排列组合的问题...

UniqueStepNum = choose (m-1) from (m+n-2) = choose (n-1) from (m+n-2)

= (m+n-2)! / [(m-1)! * (n-1)!]

= ( (m+n-2) / (m-1) ) * ( (m+n-3) / (m-2) ) * ... * (n / 1)

所以...只要按公式推就好了 = =||

### 现在，我们来看看第二个题目

第二题是第一题的变种，增加了一些限制，说白了就是在初始化dp数组以及计算过程中加入了对“墙”的判断，一旦有墙挡路，那么那个位置就要归0：

```python
class Solution(object):
    def uniquePathsWithObstacles(self, obstacleGrid):
        """
        :type obstacleGrid: List[List[int]]
        :rtype: int
        """
        # init dp
        dp = []
        m = len(obstacleGrid)
        if m == 0:
            return 0
        if m == 1:
            try:
                obstacleGrid[0].index(1)
                return 0
            except:
                return 1
        else:
            n = len(obstacleGrid[0])

        for i in range(m):
            dp.append([])
            for j in range(n):
                if i == 0:
                    if obstacleGrid[i][j] == 1:
                        dp[i].append(0)
                    elif j > 0 and dp[i][j-1] == 0:
                        dp[i].append(0)
                    else:
                        dp[i].append(1)
                elif j == 0:
                    if obstacleGrid[i][j] == 1:
                        dp[i].append(0)
                    elif i > 0 and dp[i-1][j] == 0:
                        dp[i].append(0)
                    else:
                        dp[i].append(1)
                else:
                    dp[i].append(0)
        
        # let's go...
        i = 1
        while i < m:
            j = 1
            while j < n:
                if obstacleGrid[i][j] == 1:
                    dp[i][j] = 0
                else:
                    dp[i][j] = dp[i-1][j] + dp[i][j-1]
                j += 1
            i += 1

        return dp[m-1][n-1]
```

