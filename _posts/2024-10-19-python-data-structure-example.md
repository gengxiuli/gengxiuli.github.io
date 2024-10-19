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
#!/bin/python3

from string import Template
#text = 'this is a test text，it contain little words but for test ok'
#print(text)

def word_freq_count():
    words = []
    freqs  = []
    tops = []
    text = 'this is a test text，it contain little words but for test ok'
    #text = Template(str)
    #text.substitute(who = ',',what = ' ')
    text.replace(',',' ')
    print(text)
    s = 0
    e = 0
    m = 0
    for c in text:
        e = m
        if c.isspace():
            w = text[s:e]
            if w.isspace():
                m += 1
                s += 1
                continue
            p = words.count(w)
            if p > 0:
            #w = 'this'
                i = words.index(w)
                print(i)
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

word_freq_count()


```

Tuple 元组

Set 集合

Dict 字典
