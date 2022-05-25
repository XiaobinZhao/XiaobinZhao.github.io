---
title: yield
description: yield 解释
date: 2020-10-20 09:49:01
tag: 
- python
- yield
categories:
- python
---


# yield
为了解释yiled需要理解几个概念：
- 可迭代对象
- 迭代器
- 生成器

## 可迭代对象
创建一个列表（list）时，你可以逐个地读取里面的每一项元素，这个过程称之为迭代（iteration）。

```
>>> for i in mylist:
...    print(i)
1
2
3
```
mylist是一个可迭代对象。当使用列表推导式（list comprehension）创建了一个列表时，它就是一个可迭代对象：
```
>>> mylist = [x*x for x in range(3)]
>>> for i in mylist:
...    print(i)
0
1
4
```
任何可以使用在`for...in...`语句中的对象都可以叫做**可迭代对象**，例如：lists，strings，files等等。这些可迭代对象使用非常方便因为它能如你所愿的尽可能读取其中的元素，但是你不得不把所有的值存储在内存中，当它有大量元素的时候这并不一定总是你想要的。

**注**：dict对象以及任何**实现了__iter__()或者__getitem__()方法的类都是可迭代对象**，此外，可迭代对象还可以用在zip,map等函数中，当一个可迭代对象作为参数传递给内建函数iter()时，它会返回一个迭代器对象。通常没必要自己来处理迭代器本身或者手动调用iter()，for语句会自动调用iter()，它会创建一个临时的未命名的变量来持有这个迭代器用于循环期间。 

## 迭代器（ITERATOR）
迭代器代表一个数据流对象，不断重复调用迭代器的next()方法可以逐次地返回数据流中的每一项，当没有更多数据可用时，next()方法会抛出异常StopIteration。此时迭代器对象已经枯竭了，之后调用next()方法都会抛出异常StopIteration。迭代器需要有一个__iter()__方法用来返回迭代器本身。因此它也是一个可迭代的对象。
## 生成器

### 1. 生成器定义

在Python中，一边循环一边计算的机制，称为生成器：`generator`。
生成器也是一个迭代器，但是你只可以迭代他们一次，不能重复迭代，因为它并没有把所有值存储在内存中，而是实时地生成值


### 2. 为什么要有生成器

列表所有数据都在内存中，如果有海量数据的话将会非常耗内存。

如：仅仅需要访问前面几个元素，那后面绝大多数元素占用的空间都白白浪费了。

如果列表元素按照某种算法推算出来，那我们就可以在循环的过程中不断推算出后续的元素，这样就不必创建完整的list，从而节省大量的空间。

简单一句话：**我又想要得到庞大的数据，又想让它占用空间少，那就用生成器！**



### 3. 如何创建生成器

- 通过()生成，

  只要把一个列表生成式的`[]`改成`()`，就创建了一个generator：

```python
>>> L = [x * x for x in range(10)]
>>> L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x1022ef630>
```

创建`L`和`g`的区别仅在于最外层的`[]`和`()`，`L`是一个list，而`g`是一个generator。

- 使用`yield`关键字生成

如果一个函数中包含`yield`关键字，那么这个函数就不再是一个普通函数，而是一个generator。调用函数就是创建了一个生成器（generator）对象。如果我们想定义一个自己的生成器函数怎么办？用return好像不行。没关系，python有yield的关键词。其作用和return的功能差不多，就是返回一个值给调用者，只不过有yield的函数返回值后函数依然保持调用yield时的状态，当下次调用的时候，在原先的基础上继续执行代码，直到遇到下一个yield或者满足结束条件结束函数为止。

```python
>>> def test():
...     yield 1
...     yield 2
...     yield 3
>>> t = test()
>>> t
<generator object test at 0x7f3da9d00e08>
```



### 4. 生成器的调用

1. for循环：

   ```python
   for item in gen:
       print(item)
   ```

2. next：

   ```python
   print(next(gen))#output:0
   print(next(gen))#output:1
   print(next(gen))#output:2
   print(next(gen))#output:3
   print(next(gen))#output:4
   print(next(gen))#output:Traceback (most recent call last):StopIteration
   ```

从第一个用for的调用方式我们可以知道生成器是可迭代的。更准确的说法是他就是个迭代器。
我们可以验证一下：

```python
from collections import Iterable, Iterator
print(isinstance(gen, Iterable))#output:True
print(isinstance(gen, Iterator))#output:True
```



### 4. 生成器的工作原理

（1）生成器(generator)能够迭代的关键是它有一个`next()`方法，

　　工作原理就是通过**重复调用next()**方法，直到捕获一个异常。

（2）带有`yield`的函数不再是一个普通函数，而是一个生成器`generator`。

　　可用`next()`调用生成器对象来取值。next 两种方式 `t.__next__()` | `next(t)`。

　　可用`for`循环获取返回值（每执行一次，取生成器里面一个值）

　　（基本上不会用`next()`来获取下一个返回值，而是直接使用`for`循环来迭代）。

（3）`yield`相当于`return`返回一个值，并且记住这个返回的位置，下次迭代时，代码从yield的下一条语句开始执行。

（4）`.send()`和`next()`一样，都能让生成器继续往下走一步（下次遇到yield停），但`send()`能传一个值，这个值作为`yield`表达式整体的结果

　　——换句话说，就是send可以强行修改上一个yield表达式值。比如函数中有一个yield赋值，a = yield 5，第一次迭代到这里会返回5，a还没有赋值。第二次迭代时，使用.send(10)，那么，就是强行修改yield 5表达式的值为10，那么a=10

```python
>>> def test():
...     i = 0
...     while i < 5:
...         temp = yield i
...         print(temp)
...         i += 1
        
>>> a = test()
>>> a.__next__()
0
>>> a.__next__()
None
1
>>> a.__next__()
None
2
>>> a.send("10")
10
3
```
关于为何第二次a.__next__()有None输出的问题，可以简单的理解为：a.__next__()和a.send(None)是等价的，或者这么说： `return i`这个表达式的返回值是None, temp = yiled i可以理解为： temp = return i； 因为程序中遇到return，方法就结束，所以temp = return i = None. 当然这样描述是不准确的，只为理解。

生成器仅仅保存了一套生成数值的算法，并且没有让这个算法现在就开始执行，而是我什么时候调它，它什么时候开始计算一个新的值，并给你返回。



> 参考 https://blog.csdn.net/SL_World/article/details/86507872