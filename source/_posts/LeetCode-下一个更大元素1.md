title: LeetCode-下一个更大元素1
author: Zhiyuan
tags:
categories:

  - LeetCode
date: 2019-09-04 15:17:00

---

[题目地址](https://leetcode-cn.com/problems/next-greater-element-i/)

数据结构：栈、哈希表

思路：遍历nums2，哈希表存储每个元素后第一个大于它的元素。栈为递减栈，当遇到比栈顶元素大的元素，依次弹出元素，存入哈希表。最后遍历nums1，hash[nums[i]]组成的列表即为所求。

如 nums1 = [4,1,2], nums2 = [1,3,4,2]. 

stack=[1]  hash[1]=3  hash[3]=-1(第一次出现该元素，hash值为-1)

stack=[3]  hash[3]=4 hash[4]=-1 stack=[4]  stack=[4,2] hash[2]=-1

```python
class Solution(object):
    def nextGreaterElement(self, nums1, nums2):
        """
        :type nums1: List[int]
        :type nums2: List[int]
        :rtype: List[int]
        """
        stack=[]
        hashmap={}
        for num in nums2:
            while  len(stack)>0 and stack[-1]<num :
                hashmap[stack.pop()]=num
            hashmap[num]=-1
            stack.append(num)
        return [hashmap[i] for i in nums1]
```

