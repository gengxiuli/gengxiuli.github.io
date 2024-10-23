---
layout: post
title:  "Python 数据结构代码示例(二)"
date:   2024-10-22
category: programming
tags:   python set dict
---

上一篇文章展示了 List 列表, Tuple 元组和Range范围的一些定义和用法，这篇文章继续介绍 Set 集合以及 Dict 字典。

Set是一种包含无序成员(unordered collection)的数据类型, 它包含可散列的不相同的成员对象(distinct [hashable](https://docs.python.org/3/glossary.html#term-hashable) objects)，这里的可散列的含义是，对象成员是不可修改的，且可以相互比较的，原文如下:

> An object is *hashable* if it has a hash value which never changes during its lifetime (it needs a [`__hash__()`](https://docs.python.org/3/reference/datamodel.html#object.__hash__ "object.__hash__") method), and can be compared to other objects (it needs an [`__eq__()`](https://docs.python.org/3/reference/datamodel.html#object.__eq__ "object.__eq__") method). Hashable objects which compare equal must have the same hash value.

Python中的很多内置不可修改对象都是可散列的，比如int, str，也就是可以作为Set的成员。

Set可以通过{}符号初始化，也可以通过Set()定义，如果创建空的Set则必须用Set()，因为{}表示创建一个空的Dict字典。

下面是一些示例代码：

```python
basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
print(basket)                      # show that duplicates have been removed
{'orange', 'banana', 'pear', 'apple'}
'orange' in basket                 # fast membership testing
True
'crabgrass' in basket
False

# Demonstrate set operations on unique letters from two words

a = set('abracadabra')
b = set('alacazam')
a                                  # unique letters in a
{'a', 'r', 'b', 'c', 'd'}
a - b                              # letters in a but not in b
{'r', 'd', 'b'}
a | b                              # letters in a or b or both
{'a', 'c', 'r', 'd', 'b', 'm', 'z', 'l'}
a & b                              # letters in both a and b
{'a', 'c'}
a ^ b                              # letters in a or b but not both
{'r', 'd', 'b', 'm', 'z', 'l'}
```

Set支持的一些操作：

| Operation     | Result                                                                                    |
| ------------- | ----------------------------------------------------------------------------------------- |
| x in s        | `True` if an item of *s* is equal to *x*, else `False`                                    |
| x not in s    | `False` if an item of *s* is equal to *x*, else `True`                                    |
| s + t         | the concatenation of *s* and *t*                                                          |
| len(s)        | length of *s*                                                                             |
| add(elem)     | Add element elem to the set.                                                              |
| remove(elem)  | Remove element elem from the set. Raises KeyError if elem is not contained in the set.    |
| discard(elem) | emove element elem from the set if it is present.                                         |
| pop()         | Remove and return an arbitrary element from the set. Raises KeyError if the set is empty. |
| clear()       | Remove all elements from the set                                                          |

如果把上表和List列表的的做对比，可以发现少了很多根据索引而执行的操作，这是因为Set内部使用了散列算法将对象放到合适的位置，便于快速查找，这通常只需要O(1)时间，代价就是不支持通过索引查找元素，事实上你可以用对象的__hash__()查看其hash值，还可以在自定义对象中实现__hash__(),进而支持自定义对象作为Set的成员。

Dict是一种key/value键值对映射类型(Mapping Types)，key类似于Set中的成员，也就是要求可散列，不重复，value和key是一一对应的，但是value是可以重复的，也就是多个不同的key可以有相同的value。

下表是Dict支持的一些操作

| Operation              | Result                                                                                                                                                            |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| list(d)                | Return a list of all the keys used in the dictionary d.                                                                                                           |
| len(d)                 | Return the number of items in the dictionary d.                                                                                                                   |
| d[key]                 | Return the item of d with key key. Raises a KeyError if key is not in the map.                                                                                    |
| d[key] = value         | Set d[key] to value.                                                                                                                                              |
| del d[key]             | Remove d[key] from d. Raises a KeyError if key is not in the map.                                                                                                 |
| key in d               | Return True if d has a key key, else False.                                                                                                                       |
| key not in d           | Equivalent to not key in d.                                                                                                                                       |
| iter(d)                | Return an iterator over the keys of the dictionary. This is a shortcut for iter(d.keys()).                                                                        |
| clear()                | Remove all items from the dictionary.                                                                                                                             |
| copy()                 | Return a shallow copy of the dictionary.                                                                                                                          |
| get(key, default=None) | Return the value for key if key is in the dictionary, else default. If default is not given, it defaults to None, so that this method never raises a KeyError.    |
| items()                | Return a new view of the dictionary’s items ((key, value) pairs). See the documentation of view objects.                                                          |
| keys()                 | Return a new view of the dictionary’s keys. See the documentation of view objects.                                                                                |
| pop(key[, default])    | If key is in the dictionary, remove it and return its value, else return default. If default is not given and key is not in the dictionary, a KeyError is raised. |
| values()               | Return a new view of the dictionary’s values. See the documentation of view objects.                                                                              |

Dict的初始化也使用{}，但是要求key/value结构，有如下一些示例方法：

```python
a = dict(one=1, two=2, three=3)
b = {'one': 1, 'two': 2, 'three': 3}
c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
d = dict([('two', 2), ('one', 1), ('three', 3)])
e = dict({'three': 3, 'one': 1, 'two': 2})
f = dict({'one': 1, 'three': 3}, two=2)
a == b == c == d == e == f
True

tel = {'jack': 4098, 'sape': 4139}
tel['guido'] = 4127
tel
{'jack': 4098, 'sape': 4139, 'guido': 4127}
tel['jack']
4098
del tel['sape']
tel['irv'] = 4127
tel
{'jack': 4098, 'guido': 4127, 'irv': 4127}
list(tel)
['jack', 'guido', 'irv']
sorted(tel)
['guido', 'irv', 'jack']
'guido' in tel
True
'jack' not in tel
False
```

Dict会保留插入的顺序，更新key对应的value不会改变顺序，但是删除后重新添加则会加入到最后，示例代码如下：

```python
d = {"one": 1, "two": 2, "three": 3, "four": 4}
d
{'one': 1, 'two': 2, 'three': 3, 'four': 4}
list(d)
['one', 'two', 'three', 'four']
list(d.values())
[1, 2, 3, 4]
d["one"] = 42
d
{'one': 42, 'two': 2, 'three': 3, 'four': 4}
del d["two"]
d["two"] = None
d
{'one': 42, 'three': 3, 'four': 4, 'two': None}
```

Dict的遍历和其他数据类型也有所不同，因为它的成员是key/value结构，一种方法是使用items()方法：

```python
knights = {'gallahad': 'the pure', 'robin': 'the brave'}
for k, v in knights.items():
    print(k, v)

gallahad the pure
robin the brave
```

一个示例程序：统计一段文本中单词个数，并输出频率最高的指定个数，比如 Top 10。

这个程序用Dict和Set重新实现了一下：

```python
#!/bin/python3

import string

sym = string.punctuation

def is_separator(c):
    if c.isspace():
        return True
    if c in sym:
        return True
    return False

def word_freq_count():
    words = {}
    text = 'this is a test text,it contain little words but for test ok,what do you think about it?,it is a question?'
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
            if w in words:
                words[w] += 1
            else:
                words[w] = 1
            s = e + 1
        m += 1
    print(words)

    freqs = {}
    for k,v in words.items():
        print(k,v)
        if v in freqs:
            freqs[v].append(k)
        else:
            freqs[v] = list(k)

    for k,v in freqs.items():
        print(k,v)

word_freq_count()

```

参考资料：

1. [https://docs.python.org/3/library/stdtypes.html](https://docs.python.org/3/library/stdtypes.html)
2. [https://docs.python.org/3/tutorial/datastructures.html](https://docs.python.org/3/tutorial/datastructures.html)
3. [https://docs.python.org/3/glossary.html](https://docs.python.org/3/glossary.html)
