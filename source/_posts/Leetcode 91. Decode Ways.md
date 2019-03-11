---
title: Leetcode 91. Decode Ways
date: 2017-09-23 16:30:00
tags: [leetcode,dp]
---

### 题目翻译

一个包含了字符A-Z的消息被用下列方式编码：

```
'A' -> 1
'B' -> 2
...
'Z' -> 26
```

现在给出一个编码后的数字串，求一共有多少种解码可能。如`"12"`可以被解析为`"AB"` (1 2) 或 `"L"` (12)这两种情况，因此返回值为2。

### 题目分析

一般来说求这种有多少种而不是让你求所有具体情况的题目都会有捷径（DP）走，本题亦不例外。但是今天就是有根筋没搭对我就是想DFS遍历一遍…（当然试了几次以后放弃了因为有好几种情况你不看答案的话因为带了0你不知道题目到底想让你怎么处理）最后还是用DP做啦~

**先来做点准备工作。**

按照固定的思路，我们设计一个res数组用来存储中间每一步的一个结果，**每个位置的值表示的是从这个位置往后遍历s串有多少种走法**。从这句话显然可以看出来，我们要想利用“上一步的结果”来获得本次的结果，得从后往前遍历，“从小的堆积到大的”。为了方便计算存储最后的结果，我们让res数组的size比s串长度大1。

**然后来确定这个状态转移方程。**

1. 如果当前位置是0，0是个比较特殊的数字，因为它只能跟在1和2后面构成两位数，自己是翻译不出来东西的，所以它的res值为0。
2. 又因为题目中最大数是到26，因此最多向右看2个位置的数，如果这两个位置组成的数字大于26，说明当前这个位置不能走两步，只能走一步，故 `res[i] = res[i+1]`。
3. 而如果小于26的话可以看的就多了，既可以是走一步跨两个位置，亦可以走一步跨两次，有两种情况，因此 `res[i] = res[i+1] + res[i+2]`

**最后我们来看看初始条件应该是什么样子的呢？**

**首先要注意，s[i]对应的结果是res[i]。**最右侧的res值我们定为1，它是起始的一个预置条件，目的是为了在开始阶段给res[i+2]一个值。为什么不能是0呢？看上面推的状态转移方程，再结合实际情况最右侧的 (s[n-1]) 如果不是0，那么倒数第二右侧的 (s[n-2]) 如果不是0的话 res[n-2] 应该是 res[n-1] 和 res[n] 的和，表示的是 s[n-2] 这个位置可能有多少种走法。如果我们给 res[n] 为0，那么这里 res[n-2] 算出来就会少1，显然是不对的。当最右侧是0的时候同理也可以推出来应该把 res[n] 初始化为1。

剩下的就是利用s初始化res了，s[n-1] 如果是1那么 res[n-1] 就应该是1，反之为0。

### 实现代码

```python
class Solution(object):
    def numDecodings(self, s):
        """
        :type s: str
        :rtype: int
        """
        n = len(s)
        if n == 0:
            return 0
        res = [0] * (n + 1)
        res[n] = 1
        res[n-1] = 1 if s[n-1] != '0' else 0
        i = n - 2
        while i >= 0:
            if s[i] == '0':
              	i -= 1
                continue
            t = int(s[i:i+2])
            res[i] = res[i+1] + res[i+2] if t >= 1 and t <= 26 else res[i+1]
            i -= 1
        
        return res[0]
```



### 另外

备份一个错误的DFS代码0.0可以看出如果在不知道题目到底想给你什么测试用例的情况下想要对所有情形做出一个遍历还是挺难的，因为你得找到所有可能走1步也走2步，只能走1步，只能走2步的情况，并且这里还有0干扰，就更不好判断了...

```python
class Solution(object):
    def numDecodingsFail(self, s):
        """
        :type s: str
        :rtype: int
        """
        self.nums = 0
        s = s.lstrip('0')
        if len(s) == 0:
            return 0
        if s[-1] == '0':
            s = s.rstrip('0')
            s += '0'
        self.subNum(s)
        return self.nums
    
    def subNum(self, s):
        if s == '':
            self.nums += 1
            return

        if len(s) > 3 and s[2] == '0':
            self.subNum(s[1:])
            self.subNum(s[2:])
        elif len(s) > 2 and s[2] == '0':
            self.subNum(s[1:])
        elif len(s) > 1 and s[1] == '0':
            self.subNum(s[2:])
        else:
            self.subNum(s[1:])
            if len(s) >= 2 and int(s[:2]) < 27:
                self.subNum(s[2:])

```

