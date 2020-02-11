title: Python3 Counter类、all()

author: Zhiyuan
tags:

  - Python
categories:
  - 学习笔记
date: 2019-09-06 09:15:00

---

**Counter类(计数器)**

Counter类返回一个字典，统计每个元素出现次数，可以更新

```python
import collections
obj = collections.Counter(['11','22'])
obj.update(['22','55'])
print(obj)

#输出：Counter({'22': 2, '11': 1, '55': 1})
```

**all()** 函数用于判断给定的可迭代参数 iterable 中的所有元素是否都为 TRUE，如果是返回 True，否则返回 False。

```python
class Solution:
    def countCharacters(self, words: List[str], chars: str) -> int:
        count=collections.Counter(chars)
        sumlen=0
        for word in words:
            c=collections.Counter(word)
            #list类型作为参数
            if all([count[i]>=c[i] for i in word]):
                sumlen+=len(word)
        return sumlen              
```

**dict.item()**返回字典键值对组成的元组