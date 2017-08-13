---
title: 全排列问题 -- Permutation
date: 2017-08-13 18:54:28
tags: leetcode
---

好久不写Leetcode了，果然手生的很，思路也僵化了，本科的时候都会写的全排列竟然折腾了一天才想明白怎么回事。还是要多加练习。

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

