---
title: 全排列问题 -- Permutation
date: 2018-05-05 16:00:00
tags: leetcode
---

2017-09-01 23:25:18 第二版重制，增加相关问题推荐链接

2017-08-13 18:54:28 第一版

好久不写Leetcode了，果然手生的很，思路也僵化了，本科的时候都会写的全排列竟然折腾了一天才想明白怎么回事。还是要多加练习。

[推荐链接](https://leetcode.com/problems/permutations/discuss/18239/A-general-approach-to-backtracking-questions-in-Java-(Subsets-Permutations-Combination-Sum-Palindrome-Partioning))

## 问题分析

实现一个不会有重复字符的串的全排列（这里以仅以实现数字为例）

所谓**全排列**呢，就是问你n个数字从左往右排不重样有多少种摆法，高中的时候基本大家都算过有多少种情况(**n!**种情况)还经常和**组合**（Cmn种情况，在有重复的元素的时候不考虑同样元素的先后顺序进行排列，e而全排列则会考虑先后顺序）放到一块考，到现在排列组合对我都是梦魇般的存在，高考的时候上天眷顾走了狗屎运这类问题都蒙对了因而超常发挥了一把，想不到读了大学、研究生还是得用，想想就都是泪，不提了...言归正传...

以[1, 2, 3]为例，我们按照*字典序*对其全排列结果如下：

```
1, 2, 3
1, 3, 2
2, 1, 3
2, 3, 1
3, 1, 2
3, 2, 1
```

不难看出其规律，就是：

**把这串数字从左往右分别固定住每个数字，然后对右方的数字进行交换。**

n个数字的问题不断细化，会发现每个子问题都是这种解决方法。显然的，我们可以使用递归的方式进行算法实现。

## 递归实现

递归递归，书上说是“最符合正常人脑回路的想法”，但是据我了解很多人都感觉递归比普通迭代理解起来难...我们很容易自己把自己绕到子问题的具体实现里面，对于这种时候，比较好的做法是：**把要实现的递归函数看做一个黑盒系统，只知道输入与输出，不要想内部的具体逻辑。**

假设我们定义函数为`perm()`，首先让我们想一下这个函数可能需要哪些参数：

```
nums - array 输入的原始数组，必然要有
start - int 当前函数运行到的数组位置，必然要有
# end - int 数组的结束位置，这个可忽略，因为end的位置始终是固定的，就是输入数组的长度
# 现在，我们得到函数原型：
perm(nums, start = 0)
```

根据上述分析，每个元素都要与后面的元素进行交换，我们是从左往右一个一个进行元素的“确定”工作的，因此递归式可以确定为：

```
perm(nums, start + 1)
```

递归条件又如何呢？应该是从`start`位置起，一直到数组末尾的所有都需要进行交换，我们以一个新的变量`i`作为这个循环的游标：

```python
i = start
while i < len(nums):
    nums[i], nums[start] = nums[start], nums[i] # swap value
    perm(nums, start + 1) # recursion
    # something may be lost here...
    
    i += 1
```

为什么 i = start 而不是 start + 1 呢？**因为start + 1时会丢失当前轮的组合**
**这样的话得到的结果里会有部分缺失，并且有部分重复。原因是：上述操作把i位置和start位置元素互换并进行进一步递归，当下级递归结束返回的时候再进行i+1与start位置元素互换，这个时候的nums不是初始状态的nums了！（黑人问号）**

**所以，正确的方法应该是：**

```python
i = start
while i < len(nums):
    nums[i], nums[start] = nums[start], nums[i] # swap value
    perm(nums, start + 1) # recursion
    nums[i], nums[start] = nums[start], nums[i] # reset nums to origin state
    
    i += 1
```

对于结束条件，显然是当`start`超出`nums`的范围时，此时意味着一个全排列已经生成，可以打印or保存到返回列表中去。

总结一下，最终代码如下：

```python
def perm(nums, start = 0):
    if start >= len(nums) - 1:
        print nums
        return
    else:
        i = start
        while i < len(nums):
            nums[i], nums[start] = nums[start], nums[i]
            self.perm(nums, start+1)
            nums[i], nums[start] = nums[start], nums[i]
            
            i += 1
            
```

## 非递归实现

非递归实现，顾名思义，就是一段代码直接一个接一个的得到，而不是一个函数套一个函数让人整不明白O.O~看到这个问题让我想起了上学期的时候殷王回来曾经问我们一个滴滴面试的问题，给你几个数和它们的一组排列所形成的数字，求比这个数大的数里最接近这个数的数。现在想想其实就是Leetcode 31题，求下一个全排列...当时宿舍三四个人找了半天规律T.T~言归正传，来看看怎么非递归找全排列~

以`3 1 4 5 2`为例，它的下一个应该是`3 1 5 2 4`，获得得算法如下：

```
1. 从后向前找第一双相邻的递增数字，称前一个数字为替换数，替换数的下标称为替换点(4)
2. 再从后面找一个比替换数大的最小数（这个数必然存在），将该数字和替换数交换(3 1 5 4 2)
3. 将替换点后的字符串颠倒即得到(3 1 5 2 4)
```

由此可以由一个序列推得下一个序列，当推回到最初的序列时，说明所有的全排列均已找到。

根据一个排列寻找下一排列的代码如下，求所有的排列将下属代码逐次使用上次结果进行调用即可：

```python
def permPart(nums)
    end = len(nums) - 1
    reverse = True
    swap_pos = False
    while end > 0:
        if nums[end-1] < nums[end]:
            reverse = False
            swap_pos = end - 1
            break
        else:
            end -= 1
            continue

    if reverse is True:
        nums.sort()
    else:
        i = swap_pos + 1
        bak_pos = False
        while i < len(nums):
            if nums[i] > nums[swap_pos]:
                if bak_pos is False:
                    bak_pos = i
                else:
                    bak_pos = i if nums[i] <= nums[bak_pos] else bak_pos #从右往左找第一个数，因此要加等号的判断条件
            i += 1

        nums[swap_pos], nums[bak_pos] = nums[bak_pos], nums[swap_pos]
        bak = nums[swap_pos+1:]
        bak.reverse()
        nums[swap_pos+1:] = bak
    print nums
```


## New！2017-09-01 Leetcode 47

47题和上面这道求所有的全排列的题目很像。但是不同之处就在于上面题目说明了给出的数字不会重复，**但是47题会给出重复的数字，需要去重**

其实我个人的想法是如果能用数据结构来简化的问题尽量避免去搞事情，一来代码更容易理解，别人接手容易，并且重要的是自己写起来也容易呀！再者说了...现在内存不是很不值钱吗…可以允许的范围内就尽量简化代码啦…（说起来还是自己算法渣，懒！）

果断改了改上面的代码，把插入的动作改成插入`tuple`，然后最后利用Python的set进行去重： `self.result = list(set(self.result))` 最后 `self.result = [ list(item) for item in self.result]` 强制类型转换一下返回结果完事儿。

提交以后超时了！wtf...怎么玩呢...

于是又把 `result` 改成了 dict，在插入的时候以 `'-'.join(str(e) for e in nums)` 作为键，记得哈希的搜索应该挺快的吧，插入前用 `key in self.result` 判断一下总行了吧...

结果坑爹的dict并不奏效，依然超时...然而我在网上看别人Java写的用HashMap就可以通过啊 =。= 真是醉，看来不能走捷径只能好好学算法了 T  T

#### **New DFS**

新的想法与上面的有所差异，其实对于DFS算法来说，这个写法应该更具有普适应性，更容易理解。前面的算法我们还是有“交换”的思想在里面，而我所认为的更普适应性的算法应该是更具备“常理性”，也即不应该变换原来的数据，而是在原来数据上不断地去往下搜。

根据对上面递归算法的理解，我们知道原来的函数具备的 **start** 变量用于交换时下标的记录。这里既然我们要更“普适应”，应该是每次进入递归函数都跑一遍所有的数据。我们需要增加一个参数 **sub_result** 用于记录当前轮所构成的部分排列，一个参数 **used** 记录 nums 中每个位置数字被使用过没有。**个人感觉两算法差别关键在此，之前我们是不断交换、记录，并没有单独以变量记录下来，这导致了在判断重复的时候就难以判断。**

接下来就是**决定什么时候可以跳过该轮的组合生成**，看了几个解释，感觉都不太好，并且基本都是抄的差不多的。最后在 [这里](https://www.tianmaying.com/tutorial/LC47) 看到了一个更容易理解 **used** 变量条件控制的解释。大体意思简洁版如下：

重复是如何产生的呢？结合 nums 的下标变化，不难发现，**在我们枚举的过程中是以下标为基准进行枚举，而下标的不同组合所构成的实际组合答案却是一样的**。

不难想到，这样产生的重复解决方法是，对于一个位置的同一个取值，只枚举一次，重复时跳过不进行枚举。因此，我们可以预处理一下 nums 将其先**排序**。

used 数组先预置为 False，当一个位置被走过，那么置为 True，每当退出 sub_permute，当前位置 used 重置为 False。很显然的一点是如果 used 当前位置已经置为 True 此轮循环可以直接跳过。

那么当挨着的数据相同时如何判断是否需要交换呢？这时候就要用到 used[i - 1] 咯。按照 [这里](https://www.tianmaying.com/tutorial/LC47) 的理解由于每次使用的一定是所有相同数字中最右侧的一个，所以对于一个取值，如果它右侧的数字是已经被使用过了的，就同样说明这个数是当前所有相同数字中最右侧的可用的了。可以反推得知，如果一个数左侧的数字没被使用过，说明这个数字不可用。

换种思路理解，随着DFS的进行，大pos位的数字会被不断换到左侧，同样的l两个数字在构成新串的时候就会出现 used 数组右边的先于左边的被访问，这个时候就会出现重复（比如仅考虑1 2 2，第一轮得到下标为0，1，2的一个组合，第二轮会得到0，2，1的组合，这两个组合的结果其实是一样的，在第二轮的时候2位置先被访问，此时他左侧的位置是1，used[1]为False，说明已经有一个同样的组合了，因而可以跳过。由此可推多个数字重复的时候左侧相同的将会不断被跳过，直到出现不同的数字为止）

至于有没有更好的理解方法...也希望大神们邮件我0.0~~感谢！

代码如下：

```python
import copy
class Solution(object):
    result = []
    def permuteUnique(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        self.result = []
        nums.sort()
        used = [ False ] * len(nums)
        self.sub_permute(nums, used, [])
        return self.result

    def sub_permute(self, nums, used, sub_result):
        if len(sub_result) == len(nums):
            self.result.append(copy.copy(sub_result))
            return

        i = 0
        while i < len(nums):
            if used[i] is False:
                if i > 0 and nums[i] == nums[i-1] and used[i-1] is False:
                    i += 1
                    continue
                sub_result.append(nums[i])
                used[i] =  True
                self.sub_permute(nums, used, sub_result)
                used[i] = False
                sub_result.pop()

            i += 1
```

## Leetcode 377. Combination Sum IV

给一个无重复数字的数组，但是里面的数字可以重复使用，求有多少种情况可以得到给定的 target。注意， 1 1 2 和 2 1 1 被认定是不同的组合情况。

老思路，不让求具体的情况那就是动态规划。这个题的关键在于怎么进行动态规划。我们设置数组 dp[i] 用来表示构成 i 有几种情况，那么只需要从 1 计算到 target 就可以得到所有的情况数。对于每一个新的 i，我们计算所有小于等于 i 的 nums[j] 可以和 dp 进行多少组合，也即

```
dp[i] += dp[i - nums[j]]    (i - nums[j] >= 0)
```

因为计算dp的时候本身就是算了有重复数字的（比如2的话 1 1 已经是算进去的了）因此不会出现漏算有重复数字的情况。

代码如下：

```java
public int combinationSum4(int[] nums, int target) {
    int[] dp = new int[target + 1];
    dp[0] = 1;
    for (int i = 1; i <= target; i ++) {
        for (int j = 0; j < nums.length; j ++) {
            if (i - nums[j] >= 0) {
                dp[i] += dp[i - nums[j]];
            }
        }
    }

    return dp[target];
}
```

