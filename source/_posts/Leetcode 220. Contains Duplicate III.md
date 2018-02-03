---
title: Leetcode 220 Contains Duplicate III
date: 2018-02-01 20:16:00
tags: [leetcode,sort]
---

Given an array of integers, find out whether there are two distinct indices *i* and *j* in the array such that the **absolute** difference between **nums[i]** and **nums[j]** is at most *t* and the **absolute** difference between *i* and *j* is at most *k*.

要求寻找数组中是否存在这样的一堆数字：它们的值之差小于等于 t 且 位置相距小于等于 k

利用桶排序算法，把元素按照类似于哈希的方式分到桶里，使相差为 t 的元素一定会映射到同一或相邻的桶中。选取桶大小为 t+1 避免考虑 t=0 的情况，使每个元素被映射到桶 n/(t+1)  中。这样我们只需要判断两个元素是否在同一桶中，或两元素是否在相邻桶中，且差小于等于 t。当位移多于 k 时，应该删除最早的那个桶。

**需要注意的是，这里我们是用 / 而不是 % 作为“哈希”的依据。**

```python
class Solution(object):
    def containsNearbyAlmostDuplicate(self, nums, k, t):
        """
        :type nums: List[int]
        :type k: int
        :type t: int
        :rtype: bool
        """
        if t < 0:
            return False
        bucket = {}
        n = len(nums)
        
        for i in range(n):
            m = nums[i] / (t + 1)
            if m in bucket or (m - 1 in bucket and abs(nums[i] - bucket[m - 1]) < (t + 1)) or (m + 1 in bucket and abs(nums[i] - bucket[m + 1]) < (t + 1)):
                return True
            bucket[m] = nums[i]
            # delete first added num
            if i >= k:
                del bucket[nums[i - k] / (t + 1)]
        
        return False
```

