---
layout: post
title:  "Python 数据结构代码示例(一)"
date:   2024-10-19
category: programming
tags:   python
---

上一篇文章[SONiC 中的 Python 编程](https://gengxiuli.com/posts/python-programming-in-sonic/)中提到了 Python 的具体应用，为了打好基础，我们先复习一下 Python 内置的数据结构，并编写一些示例代码，展示一下如何使用他们。

List 列表, Tuple 元组和Range范围

List是一种序列类型(Sequence Types), 它包含List，Tuple和Range，序列类型支持一些通用操作：

| Operation | Result |
| ----------- | ----------- |
| x in s |                           `True` if an item of *s* is equal to *x*, else `False` |
| x not in s   |                  `False` if an item of *s* is equal to *x*, else `True` |
| s + t   |                            the concatenation of *s* and *t* |
| `s * n` or `n * s`  |     equivalent to adding *s* to itself *n* times |
| s[i]     |                            *i*th item of *s*, origin 0 |
| s[i:j]  |                            slice of *s* from *i* to *j* |
| s[i:j:k]  |                           slice of *s* from *i* to *j* with step *k* |
| len(s) |                            length of *s* |
| min(s)  |                         smallest item of *s* |
| max(s)  |                       largest item of *s* |
| s.index(x[, i[, j]]) |       index of the first occurrence of *x* in *s* (at or after index *i* and before index *j*) |
| s.count(x)   |                 total number of occurrences of *x* in *s* |

List是可修改序列类型(mutable Sequence Types)，而Tuple和Range是不可修改序列类型(Immutable Sequence Types)，不可修改意味着元素个数在定以后不可改变，元素内容也不可改变，所以Tuple和Range功能要比List少很多。

下表是List支持的一些操作

| Operation | Result |
| ----------- | ----------- |
| s[i] = x |  item i of s is replaced by x |
| s[i:j] = t | slice of s from i to j is replaced by the contents of the iterable t |
| del s[i:j] | same as s[i:j] = [] |
| s[i:j:k] = t | the elements of s[i:j:k] are replaced by those of t |
| del s[i:j:k] | removes the elements of s[i:j:k] from the list |
|s.append(x) | appends x to the end of the sequence (same as s[len(s):len(s)] = [x]) |
| s.clear() | removes all items from s (same as del s[:]) |
| s.copy() | creates a shallow copy of s (same as s[:]) |
| s.extend(t) or s += t | extends s with the contents of t (for the most part the same as s[len(s):len(s)] = t) |
| s *= n | updates s with its contents repeated n times |
| s.insert(i, x) | inserts x into s at the index given by i (same as s[i:i] = [x]) |
| s.pop() or s.pop(i) | retrieves the item at i and also removes it from s |
| s.remove(x) | removes the first item from s where s[i] is equal to x |
| s.reverse() | reverses the items of s in place |

List的初始化
fruit = ['apple', 'banana']

Tuple的初始化
month = ('Jan','Feb')

范围的初始化
r = range(0, 20, 2)

可以通过range给list初始化
list(range(10))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
list(range(0, 30, 5))
[0, 5, 10, 15, 20, 25]

一个示例程序：统计一段文本中单词个数，并输出频率最高的指定个数，比如 Top 10。

```python
#!/bin/python3

import string
#from string import Template
#text = 'this is a test text，it contain little words but for test ok'
#print(text)

#sym = [',','.','?','!']
sym = string.punctuation

def is_separator(c):
    if c.isspace():
        return True
    if c in sym:
        return True
    return False

def word_freq_count():
    words = []
    freqs  = []
    tops = []
    text = 'this is a test text,it contain little words but for test ok,what do you think about it?,it is a question.'
    #text = Template(str)
    #text.substitute(who = ',',what = ' ')
    #text.replace(',',' ')
    print(text)
    s = 0
    e = 0
    m = 0
    for c in text:
        e = m
        if is_separator(c):
            w = text[s:e]
            if is_separator(w):
                m += 1
                s += 1
                continue
            p = words.count(w)
            if p > 0:
            #w = 'this'
                i = words.index(w)
                #print(i)
            #if i > 0:
                freqs[i] += 1
                tops[i] += 1
            else:
                words.append(w)
                freqs.append(1)
                tops.append(1)
            s = e + 1
        m += 1
    print(words)
    print(freqs)
    print(tops)
    #top = freqs
    tops.sort(reverse=True)
    #print(freqs)
    #print(top)
    #for n in top:
    #print(top[0])
    i = freqs.index(tops[0])
    #i = freqs.index(2)
    #print(i)
    print(words[i])
    print(tops[0])

word_freq_count()
```

参考资料：
1. [https://docs.python.org/3/library/stdtypes.html](https://docs.python.org/3/library/stdtypes.html)
2. [https://docs.python.org/3/tutorial/datastructures.html](https://docs.python.org/3/tutorial/datastructures.html)
