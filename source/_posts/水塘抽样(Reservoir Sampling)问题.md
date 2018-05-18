---
title: 水塘抽样(Reservoir Sampling)问题
date: 2018-05-12 20:35:00
tags: [leetcode,math,random]
---

## 介绍

水塘抽样是一系列的随机算法，其目的在于从包含n个项目的集合S中选取k个样本，其中n为一很大或未知的数量，尤其适用于不能把所有n个项目都存放到主内存的情况。

## 方法

把前 k 个数放入水塘，对于第 k + 1 个数，我们以 k / (k + 1) 的概率决定是否把它换入水塘，换入时随机的选取水塘中一个数字作为替换项，一直做下去，对于任意的样本空间n，对每个数的选取概率都是 k / n，也即概率相等。伪码如下：

```
Init : a reservoir with the size： k
for i= k+1 to N
    M=random(1, i);
    if(M < k)
       SWAP the Mth value and ith value
end for 
```

## 证明

![证明思路](http://ow0f2jm1j.bkt.clouddn.com/%E6%B0%B4%E5%A1%98%E6%8A%BD%E6%A0%B7%E8%AF%81%E6%98%8E2.jpg)

## 例-Leetcode 398. Random Pick Index

本题目相当于K为1的情况，如果不是目标数字则往后找，如果是且水塘为空那么一定放入水塘，否则的话以 1 / (count + 1) 的概率决定是否换掉水塘中已有的数字，这个概率很容易实现，只需要取 0 - count 随机数，取到每个数的概率是一样的，随便让它等于 0 - count 都可以。

```java
public class Solution {
    int[] nums;
    Random rnd;

    public Solution(int[] nums) {
        this.nums = nums;
        this.rnd = new Random();
    }
    
    public int pick(int target) {
        int result = -1;
        int count = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] != target)
                continue;
            if (rnd.nextInt(++count) == 0)
                result = i;
        }
        
        return result;
    }
}
```

