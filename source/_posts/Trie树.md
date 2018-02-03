---
title: Trie树 - 字典树
date: 2018-01-28 22:17:00
tags: [leetcode,tree]
---

## 简介

Trie树又称单词查找树，字典树，是一种树形结构，是一种哈希树的变种。典型应用是用于统计，排序和保存大量的字符串（但不仅限于字符串），所以经常被搜索引擎系统用于文本词频统计。它的优点是：利用字符串的公共前缀来减少查询时间，最大限度地减少无谓的字符串比较，查询效率比哈希树高。

## 性质

1. 根节点不包含字符，除根节点外每一个节点都只包含一个字符
2. 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串
3. 每个节点的所有子节点包含的字符都不相同

## 实现思路

字典树显然是一个树结构，在保证上述三个性质的基础上，增加一个位用于表示该节点是否为单词的终止处即可

## Python实现

```python
class Trie(object):
    
    def __init__(self):
        """
        Initialize your data structure here.
        """
        self.root = {
            'isEnd': True,
            'neighbors': {}
        }
        

    def insert(self, word):
        """
        Inserts a word into the trie.
        :type word: str
        :rtype: void
        """
        ptr = self.root
        for ch in word:
            if ch in ptr['neighbors']:
                ptr = ptr['neighbors'][ch]
            else:
                ptr['neighbors'][ch] = {
                    'isEnd': False,
                    'neighbors': {}
                }
                ptr = ptr['neighbors'][ch]
        ptr['isEnd'] = True
        return
            

    def search(self, word):
        """
        Returns if the word is in the trie.
        :type word: str
        :rtype: bool
        """
        ptr = self.root
        for ch in word:
            if ch not in ptr['neighbors']:
                return False
            ptr = ptr['neighbors'][ch]
        
        return ptr['isEnd']
            
        

    def startsWith(self, prefix):
        """
        Returns if there is any word in the trie that starts with the given prefix.
        :type prefix: str
        :rtype: bool
        """
        ptr = self.root
        for ch in prefix:
            if ch not in ptr['neighbors']:
                return False
            ptr = ptr['neighbors'][ch]
        
        return True
```

相关题目：Leetcode 208
