title: Python3 while-else、//
author: Zhiyuan
tags:

  - Python
categories:
  - 学习笔记
date: 2019-09-08 20:43:00

---

在LeetCode-自除数答案中学到的几个语法和技巧，

**While-else:**

1、在Python中，else 可以和 while 循环搭配使用，当 while 循环正常执行完的情况下，执行 else 输出；    

2、如果当 while 循环中执行了跳出循环的语句，比如 break，将不执行 else 代码块的内容。

**//**:向下取整

```python
class Solution:
    def selfDividingNumbers(self, left: int, right: int) -> List[int]:
        ans = []
        for num in range(left,right + 1):
            copy = num
            while copy > 0:
                div, copy = copy % 10, copy // 10
                if div == 0 or num % div != 0: break
            else: ans.append(num) # while … else 在循环条件为 false 时执行 else 语句块
        return ans
```





 