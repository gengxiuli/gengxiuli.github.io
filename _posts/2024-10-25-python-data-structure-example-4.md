---
layout: post
title:  "Python 数据结构代码示例(四)"
date:   2024-10-25
category: programming
tags:   python counter deque defaultdict
---

除了之前介绍的几种常用数据结构外，Python 的[collections](https://docs.python.org/3/library/collections.html)模块，还支持了很多容器数据类型(Container datatypes)。研究了一下，其中counter，deque和defaultdict 对目前的一些场景比较有用，特此记录一下。

### 1.counter

Counter是Dict的一个子类(subclass),主要用于可散列对象的计数场景。具体数据结构实现是Dict，其中key存储需要计数的对象，而value则是该对象对应的计数，可以认为是一种特殊的Dict字典。

Counter的value可以是任何整型值包括0和负数，这里的负数有点意思，什么场景统计需要负数呢？从数学角度说，负数和正数只是方向的差异，从财务角度负数就是欠账，总之这种设计可以覆盖更多的使用场景。

统计值0和负值都只是value的不同数值，对应的key还是存在的，也就是Dict中有一条记录。另外，如果使用c[key]的方式访问Counter，如果key不存在则返回也是0，但是和上面的0其实是不一样的，后面这种Dict中是没有对应的记录的。

因为Counter中的value固定是整型数字，所以还可以使用total()方法直接计算所有value总和，使用most_common(n)方法获取最大n个记录，这对于一些统计场景非常有用。

示例代码

```python

# Tally occurrences of words in a list
cnt = Counter()
for word in ['red', 'blue', 'red', 'green', 'blue', 'blue']:
    cnt[word] += 1

cnt
Counter({'blue': 3, 'red': 2, 'green': 1})

# Find the ten most common words in Hamlet
import re
words = re.findall(r'\w+', open('hamlet.txt').read().lower())
Counter(words).most_common(10)
[('the', 1143), ('and', 966), ('to', 762), ('of', 669), ('i', 631),
 ('you', 554),  ('a', 546), ('my', 514), 

```

### 2.deque

deque("double-ended queue") 双端队列，顾名思义，在头部或者尾部添加或者删除元素都可以做到O(1)时间复杂度，但是在队列中间添加或者删除元素，则平均时间复杂度会达到O(n)。

Python是怎么做到的呢，通过阅读deque的[代码实现](https://github.com/python/cpython/blob/main/Modules/_collectionsmodule.c)可知，其内部还是使用了数组来存储数据，但是并不像 List那样是连续的逻辑地址，而是通过Block来存储数据，一个Block目前是64。然后用双向链表的方式将这些Block连接起来，这样可以实现对于头部和尾部的快速添加删除，因为添加和删除并不会移动其他元素。

对于添加来说，只有当前的Block没有剩余空间了，才申请一块新的Blcok,除了新添加的元素，剩余空间就可以下次直接使用了。同理，对于删除来说，当元素释放后，只有整个Block都没有元素了，才将整个Block释放掉。当然，为了这些Block可以重复使用，deque内部还维护了空闲Block链表，这样申请和释放可以更高效。

列表实现实现头端添加和删除的缺陷是，元素添加后无法申请新的空间，元素删除后空间又不能直接回收，其实如果你在列表上把这两个问题解决了，上面deque的实现就是一种自然而然的解法了。

Deque除了支持List的一些方法外，还支持外左侧(left)的一些操作，比如
appendleft(),extendleft(),popleft(),这些方法可以方便地在左侧添加和删除元素。

示例代码

```python

from collections import deque
d = deque('ghi')                 # make a new deque with three items
for elem in d:                   # iterate over the deque's elements
    print(elem.upper())
G
H
I

d.append('j')                    # add a new entry to the right side
d.appendleft('f')                # add a new entry to the left side
d                                # show the representation of the deque
deque(['f', 'g', 'h', 'i', 'j'])

d.pop()                          # return and remove the rightmost item
'j'
d.popleft()                      # return and remove the leftmost item
'f'
list(d)                          # list the contents of the deque
['g', 'h', 'i']
d[0]                             # peek at leftmost item
'g'
d[-1]                            # peek at rightmost item
'i'

list(reversed(d))                # list the contents of a deque in reverse
['i', 'h', 'g']
'h' in d                         # search the deque
True
d.extend('jkl')                  # add multiple elements at once
d
deque(['g', 'h', 'i', 'j', 'k', 'l'])
d.rotate(1)                      # right rotation
d
deque(['l', 'g', 'h', 'i', 'j', 'k'])
d.rotate(-1)                     # left rotation
d
deque(['g', 'h', 'i', 'j', 'k', 'l'])

deque(reversed(d))               # make a new deque in reverse order
deque(['l', 'k', 'j', 'i', 'h', 'g'])
d.clear()                        # empty the deque
d.pop()                          # cannot pop from an empty deque
Traceback (most recent call last):
    File "<pyshell#6>", line 1, in -toplevel-
        d.pop()
IndexError: pop from an empty deque

d.extendleft('abc')              # extendleft() reverses the input order
d
deque(['c', 'b', 'a'])

```

### 3.defaultdict

Defaultdict也是Dict从继承的一个子类，因此他拥有Dict的所有方法，同时他的创建参数中多了一个default_factory属性，默认是none，顾名思义，这是一种生产默认值的工厂属性。

这个属性在内部会被Defaultdict的__missing__(key)方法调用，如果default_factory为none则会触发KeyError异常，如果default_factory不为none，则会为Defaultdict的key生成一个默认值，而__missing__()会被Dict中的__getitem__()调用，后者在访问字典中不存在的key时被调用。

简单来说就是，Defaultdict可以提供一种自定义工厂方法入口，来为不存在的key创建一个默认值。这在创建Dict字典的时候很有用，因为你需要处理key不存在和key存在两种情况，当default_factory属性是list时，就是动态创建了空列表，当属性是int就是动态创建了0整型值。

```python

s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]
d = defaultdict(list)
for k, v in s:
    d[k].append(v)

sorted(d.items())
[('blue', [2, 4]), ('red', [1]), ('yellow', [1, 3])]

s = 'mississippi'
d = defaultdict(int)
for k in s:
    d[k] += 1

sorted(d.items())
[('i', 4), ('m', 1), ('p', 2), ('s', 4)]

```

参考资料
1. <https://docs.python.org/3/library/collections.html>
2. <https://docs.python.org/3/library/functions.html#sorted>
3. <https://github.com/python/cpython/blob/3.13/Lib/collections/__init__.py>
4. <https://github.com/python/cpython/blob/main/Modules/_collectionsmodule.c>
5. <https://docs.python.org/3/library/operator.html>
