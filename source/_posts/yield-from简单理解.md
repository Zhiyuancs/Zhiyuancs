title: yield from简单理解
author: Zhiyuan
tags:

  - Python
categories:
  - 学习笔记
date: 2019-09-05 09:45:00

---

### yield

**yield**简单理解可以看成**return**，但是函数执行**yield**后会返回值但是不会终止。

```python
# yield返回值, 生成器
def gen():
    for x in ["a", "b", "c"]:
        yield x

for i in gen():
    print(i)

# a b c
```

**在函数外部不能使用yield from（yield也不行）。**

### yield from

**yield from**用来调用生成器，可以用于递归函数中，或者调用的函数中包含**yield语句**

```python
#中序遍历树
class Solution:
    def increasingBST(self, root):
        def inorder(node):
            if node:
                yield from inorder(node.left)
                yield node.val
                yield from inorder(node.right)

        ans = cur = TreeNode(None)
        for v in inorder(root):
            cur.right = TreeNode(v)
            cur = cur.right
        return ans.right
```

上例函数为生成器函数，生成器对象是一个可迭代对象，可以存储遍历结果

