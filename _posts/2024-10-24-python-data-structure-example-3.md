---
layout: post
title:  "Python 数据结构代码示例(三)"
date:   2024-10-24
category: programming
tags:   python C语言 C++
---

前两篇文章我们已经介绍了List列表，Tuple元组，Range范围，Set集合，Dict字典这几种数据类型，以及它们支持的一些操作。

如果之前一直使用C语言编程，会感觉Python的数据类型有很多，其实上面只是一小部分，还有很多我们还没有介绍。C语言本身支持的数据类型很少，数组和结构体是两个基本且常用的类型了，再加上强大的指针可以实现各种数据结构了，比如Python官方的解释器Cpython就是用C语言实现的。

C语言之后的C++增加了很多内置类型，比如vector、list、deque、set、map等。这些容器类提供了丰富的方法来管理数据集合，包括添加、删除、搜索和排序数据。

不管是Python还是C++，其内置的数据类型都是为了方便我们编程使用，C++通过模板和迭代器，扩充了数据结构的能力边界，Python则针对各种不同应用，定义了丰富的数据结构，相对于C语言来说，确实易用性增加了很多。但是这些数据结构自有其特定使用场景，选择得当能提高程序的性能，选择不当则事半功倍还不如不用，所以我们还得了解其适用范围，同时在实际编程中具体体会。

举几个例子说明一下:

### 1.List列表的索引访问和值访问

```python
    nums = [3,1,4,1,5,9,2,6,5,3,5,8,9,7,9,3,2]
    print(nums[5]) #(1)打印索引是5的元素
    9
    print(nums.index(5)) #(2)打印第一个值是5的元素的索引
    4
    print(nums.count(5)) #(3)打印值是5的元素的个数
    3
```

上面的例子中(1)是通过索引访问，而(2)和(3)是通过元素值访问，因为列表的内部实现是可变长数组，所以(1)直接可以找到元素，时间复杂度是O(1)，而(2)和(3)因为要遍历比较值的大小，时间复杂度是O(n)。

总结：List列表因为内部是可变长数组实现，所以适用于通过索引的访问，通过值访问时间开销大，对于时间要求不高的场景可以，对于时间要求高的场景需要使用其他数据结构。

### 2.Stack栈结构的实现方法

```python
    stack = [3, 4, 5]
    stack.append(6)
    stack.append(7)
    stack
    [3, 4, 5, 6, 7]
    stack.pop()
    7
    stack
    [3, 4, 5, 6]
    stack.pop()
    6
    stack.pop()
    5
    stack
    [3, 4]
 ```   

上面是python doc中的一个例子，使用List实现Stack栈结构，append相当于push压栈，pop就是弹出。因为Stack是LIFO(后进先出)结构，在尾部插入或者删除，根据索引就可以实现，List可以很好的支持这种操作。

### 3.Queue结构的实现方法

```python
    from collections import deque
    queue = deque(["Eric", "John", "Michael"])
    queue.append("Terry")           # Terry arrives
    queue.append("Graham")          # Graham arrives
    queue.popleft()                 # The first to arrive now leaves
    'Eric'
    queue.popleft()                 # The second to arrive now leaves
    'John'
    queue                           # Remaining queue in order of arrival
    deque(['Michael', 'Terry', 'Graham'])
 ```

上面也是python doc的一个例子，使用deque实现Queue结构。Queue队列需要支持FIFO(先进先出)，虽然List也可以通过insert和remove支持，但是因为List的insert和remove在内部的实现是，移动第一个元素所有后面元素向右或者向左，所以时间复杂度其实是O(n)，并不适合Queue的高效实现。Python中collections的deque结构专门为Queue应用而设计，支持在两端快速插入和删除操作。

### 4.索引遍历的实现方法

```python
    count = 100
    start = 0
    end = start + count
    for x in range(start,end):
        print(x)
```

上述代码实现x从start到end的编译，操作是简单打印x的值。这里range动态生成范围序列，实际值存放了start和end两个变量，中间范围根据计算动态生成。
    
### 5.Key/Value的实现方法

在前两篇文章中，实现了一个统计一段文本中单词个数，并输出频率最高的指定个数的程序。分别用List和Dict的方式实现了key/value结构，其中List方法使用了两个List,一个存放key一个存放vlue，同步添加和删除。

Dict则直接提供了key/value的数据结构，其中key是必须保证唯一。在生成完Dict后还需要根据value聚合成新的Dict，也就是将前一个Dict中的value对应的key合并成List，这样新的Dict的key就是出现次数，而value就是包含的所有单词。注意新的Dict中的value其实不是普通值，而是一个List列表。这种数据结构中嵌套其他数据结构的方法，在Python会经常用到，也是实现很多复杂功能的强大工具。当然，通过C语言也可以实现类似的效果，但是要编写很多代码，这也体现了Python丰富数据结构的优势。

### 6.元素是否允许重复的实现方法

通过for x in y来判断y中是否包含x,注意这里的y可以是List也可以是Set和Dict，如果是Dict则查找的是key中是否包含x。注意如果y是List，因为列表本质是按索引查找，如果查找值则时间复杂度为O(n)，所以遇到查找元素值的场景，尽量使用Set或者Dict数据结构，因为它们内部通过hash算法实现了高效的查找，时间复杂度一般是O(1)。