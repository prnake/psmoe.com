---
title: Python 常用技巧
mathjax: true
url: python-skills.html
categories:
	- Coding
tags:
	- python
date: 2020/7/27 12:40
---
在 Leetcode 上划水写写 Python3 ，主要追求程序的简洁和实现的精巧。

<!--more-->

<!-- toc -->

## Zen of Python

> **生活的最好状态是冷冷清清的风风火火。 ——木心**

```python
>>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

## lru_cache

`lru_cache` 属于 `functools` ，是一个为函数提供缓存功能的装饰器，利用的是 [最久未使用算法 (Least recently used)](https://en.wikipedia.org/wiki/Cache_algorithms#Examples) 进行缓存。适合在**需要重用之前计算结果，且函数参数可哈希**的情况下使用。比如**动态规划、记忆化搜索**，可以减少存储计算结果的代码。

该函数的默认调用参数为`@functools.lru_cache(maxsize=128, typed=False)`。特别的，如果 *maxsize* 设为 `None`，LRU 特性将被禁用且缓存可无限增长。

### 509. 斐波那契数

```python
class Solution:
    @lru_cache(None)
    def fib(self, N: int) -> int:
        if N < 2:
            return N
        return self.fib(N-1) + self.fib(N-2)
```

### 329. 矩阵中的最长递增路径

>给定一个整数矩阵，找出最长递增路径的长度。
>
>对于每个单元格，你可以往上，下，左，右四个方向移动。 你不能在对角线方向上移动或移动到边界外（即不允许环绕）。

直接使用 `dfs` + `lru_cache` ：

```python
class Solution:
    def longestIncreasingPath(self, matrix: List[List[int]]) -> int:
        @lru_cache(None)
        def dfs(i,j):
            res = 1
            if i-1>=0 and matrix[i-1][j]>matrix[i][j]:
                res=max(res,dfs(i-1,j)+1)
            if i+1<len(matrix) and matrix[i+1][j]>matrix[i][j]:
                res=max(res,dfs(i+1,j)+1)
            if j-1>=0 and matrix[i][j-1]>matrix[i][j]:
                res=max(res,dfs(i,j-1)+1)
            if j+1<len(matrix[0]) and matrix[i][j+1]>matrix[i][j]:
                res=max(res,dfs(i,j+1)+1)
            return res
        res = 0
        for i in range(len(matrix)):
            for j in range(len(matrix[0])):
                res = max(res,dfs(i,j))
        return res
```

### functools

> [`functools`](https://docs.python.org/zh-cn/3/library/functools.html#module-functools) 模块应用于高阶函数，即参数或（和）返回值为其他函数的函数。 通常来说，此模块的功能适用于所有可调用对象。

看起来还有很多的功能可以挖掘。

## 使用异常抛出机制解决问题

### 什么是异常？

- 异常即是一个事件，该事件会在程序执行过程中发生，影响了程序的正常执行。
- 一般情况下，在 Python 无法正常处理程序时就会发生一个异常。
- 异常是 Python 对象，表示一个错误。
- 当 Python 脚本发生异常时我们需要捕获处理它，否则程序会终止执行。

### 异常处理

捕捉异常可以使用 try/except 语句。

try/except 语句用来检测 try 语句块中的错误，从而让 except 语句捕获异常信息并处理。

如果你不想在异常发生时结束你的程序，只需在 try 里捕获它。

以下为简单的语法：

```python
try:
<语句>        #运行别的代码
except <名字>：
<语句>        #如果在 try 部份引发了'name'异常
except <名字>，<数据>:
<语句>        #如果引发了'name'异常，获得附加的数据
else:
<语句>        #如果没有异常发生
```

try 的工作原理是，当开始一个 try 语句后，python 就在当前程序的上下文中作标记，这样当异常出现时就可以回到这里，try 子句先执行，接下来会发生什么依赖于执行时是否出现异常。

- 如果当 try 后的语句执行时发生异常，python 就跳回到 try 并执行第一个匹配该异常的 except 子句，异常处理完毕，控制流就通过整个 try 语句（除非在处理异常时又引发新的异常）。
- 如果在 try 后的语句里发生了异常，却没有匹配的 except 子句，异常将被递交到上层的 try，或者到程序的最上层（这样将结束程序，并打印默认的出错信息）。
- 如果在 try 子句执行时没有发生异常，python 将执行 else 语句后的语句（如果有 else 的话），然后控制流通过整个 try 语句。

### 巧用异常抛出机制

当遇到**索引访问越界或者不被数据结构认可的处理方式**时，可以采用 `try & except` 语句减少合法性判断的代码。

### 350. 两个数组的交集 II

> 给定两个数组，编写一个函数来计算它们的交集。

```python
class Solution:
    def intersect(self, nums1: List[int], nums2: List[int]) -> List[int]:
        res = []
        for num in nums1:
            try:
                nums2.remove(num)
                res.append(num)
            except:
                continue
        
        return res
```

## 迭代器 iter

迭代器的好处是 `for in` 遍历时，迭代器自身也会移动，适合处理只需要使用一次的数据。

### 392. 判断子序列

> 给定字符串 s 和 t ，判断 s 是否为 t 的子序列。
>
> 你可以认为 s 和 t 中仅包含英文小写字母。字符串 t 可能会很长（长度 ~= 500,000），而 s 是个短字符串（长度 <=100）。
>
> 字符串的一个子序列是原始字符串删除一些（也可以不删除）字符而不改变剩余字符相对位置形成的新字符串。（例如，"ace"是"abcde"的一个子序列，而"aec"不是）。
>

```python
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        t = iter(t)
        return all(c in t for c in s) #t 在 c in t 的查找过程中不断右移
```
