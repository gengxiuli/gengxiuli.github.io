---
layout: post
title:  "Python 数据结构代码示例"
date:   2024-08-24
category: programming
tags:   python
---

上一篇文章[SONiC 中的 Python 编程](https://gengxiuli.com/posts/python-programming-in-sonic/)中提到了 Python 的具体应用，为了打好基础，我们先复习一下 Python 内置的数据结构，并编写一些示例代码，展示一下如何使用他们。

List 列表

一个示例程序：统计一段文本中单词个数，并输出频率最高的指定个数，比如 Top 10。

```python
text = 'this is a test text，it contain little words but for test ok'

words = []
freqs  = []

def word_feq_count()
    s = 0
    e = 0
    for c in text
        if c is ' '
            i = words.index(w)
            if i > 0
                freqs[i] += 1
            else
                words.append(text[s,e])
                freqs.append(1)
        s = 0
        e = 0
    top = freqs
    top.sort()
    for n in top
        i = freqs(n)
        print(word[i])

```

Tuple 元组

Set 集合

Dict 字典