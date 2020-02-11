title: LeetCode-质数排列
author: Zhiyuan
tags:
  - 数论
categories:
  - LeetCode
date: 2019-09-02 14:34:00

---

**质数**：质数是大于1的，且不能用小于它的两个正整数乘积表示。

首先求1-n中质数的个数

```python
def getnum(self,n:int) ->int:
        num=0
        for i in range(2,n+1):
            flag=True
            for j in range(2,int(i**0.5)+1):
                if i%j==0:
                    flag=False
                    break
            if flag==True:
                num+=1
        return num
```

总的方案数就是质数排列数和非质数排列数的乘积

```python
    def jiecheng(self,n:int) ->int:
        sums=1
        for i in range(1,n+1):
            sums*=i
        return sums%1000000007
    def numPrimeArrangements(self, n: int) -> int:
        if n==1 or n==2:
            return 1
        count=self.getnum(n)
        #调用类中的函数要使用self.
        num=self.jiecheng(count)*self.jiecheng(n-count)
        return num%1000000007
```

