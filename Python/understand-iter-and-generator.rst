理解 Python 迭代对象、迭代器、生成器
==========================================

.. contents:: Sections:
   :local:
   :depth: 2

``Python`` 的数据结构概念众多,下图显示了python各种数据结构之间的关系


.. image:: http://static.open-open.com/lib/uploadImg/20161215/20161215155425_891.png
   :width: 500
   :height: 340


python 数据结构的主要有以下：

容器( ``container`` )、可迭代对象( ``iterable`` )、迭代器( ``iterator`` )、生成器( ``generator`` )、列表/集合/字典推导式( ``list`` , ``set`` , ``dict`` comprehension)


容器(container)
-----------------------

``容器`` 是一种把多个元素组织在一起的数据结构，容器中的元素可以逐个地迭代获取，使用 ``in`` ,  ``not in`` 关键字判断元素是否包含在容器中

通常这类数据结构把所有的元素 **存储在内存** 中（也有一些特列并不是所有的元素都放在内存）

从技术角度来说，当它可以用来询问某个元素是否包含在其中时，那么这个对象就可以认为是一个容器
比如 ``list`` ，``set`` ，``tuples`` 都是容器对象

可以将容器类比为一个盒子、一栋房子、一个柜子，里面可以塞任何东西

常见的容器对象:

- list, deque, ....
- set, frozensets, ....
- dict, defaultdict, OrderedDict, Counter, ....
- tuple, namedtuple, …
- str


询问某元素是否在 ``dict`` 中用 ``dict`` 的中 ``key`` ,而不能通过 ``value`` 来判断

::

   >>> d = {1: 'foo', 2: 'bar', 3: 'qux'}
   >>> assert 1 in d
   >>> assert 'foo' not in d  # 'foo' 不是dict中的元素


尽管绝大多数容器都提供了某种方式来获取其中的每一个元素，但这并不是容器本身提供的能力，而是 ``可迭代对象`` 赋予了容器这种能力。

当然并不是所有的容器都是可迭代的，比如： ``Bloom filter`` ，虽然 ``Bloom filter`` 可以用来检测某个元素是否包含在容器中，但是并不能从容器中获取其中的每一个值，因为 ``Bloom filter`` 压根就没把元素存储在容器中，而是通过一个散列函数映射成一个值保存在数组中

可迭代对象(iterable)
---------------------

凡是可以返回一个 ``迭代器(iterator)`` 的对象都可称之为 ``可迭代对象(iterable)``

::

   >>> x = [1, 2, 3]
   >>> y = iter(x)
   >>> z = iter(x)
   >>> next(y)
   1
   >>> next(y)
   2
   >>> next(z)
   1
   >>> type(x)
   <class 'list'>
   >>> type(y)
   <class 'list_iterator'>

可迭代对象与迭代器对象的关联
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

迭代器有一种具体的迭代器类型,比如  ``list_iterator`` ， ``set_iterator`` 。

可迭代对象实现了  ``__iter__`` 和  ``__next__`` 方法（python2中是  ``next`` 方法，python3是  ``__next__`` 方法）
这两个方法对应内置函数  ``iter()`` 和  ``next()``

``__iter__`` 方法返回可迭代对象本身，这使得他既是一个可迭代对象同时也是一个迭代器

运行一个 ``list`` 的过程解析
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  x = [1, 2, 3]
  for elem in x:
     ...

实际执行情况是

.. image:: http://static.open-open.com/lib/uploadImg/20161215/20161215155425_967.png
   :width: 400
   :height: 150

迭代器(iterator)
-------------------------
``迭代器`` 是一个带状态的对象，他能在你调用 ``next()`` 方法的时候返回容器中的下一个值。

任何实现了  ``__next__()`` （python2中实现  ``next()`` ）方法的对象都是 ``迭代器`` ，至于它是如何实现的这并不重要

``迭代器`` 就是实现了工厂模式的对象，它在你每次你询问要下一个值的时候给你返回

``itertools`` 函数返回的都是迭代器对象

* 生成无限序列::

   >>> from itertools import count
   >>> counter = count(start=13)
   >>> next(counter)
   13
   >>> next(counter)
   14

* 从一个有限序列中生成无限序列::

    >>> from itertools import cycle
    >>> colors = cycle(['red', 'white', 'blue'])
    >>> next(colors)
    'red'
    >>> next(colors)
    'white'
    >>> next(colors)
    'blue'
    >>> next(colors)
    'red'  

* 从无限的序列中生成有限序列::

    >>> from itertools import islice
    >>> colors = cycle(['red', 'white', 'blue'])  # infinite
    >>> limited = islice(colors, 0, 4)            # finite
    >>> for x in limited:                        
    ...     print(x)
    red
    white
    blue
    red   

* 自定义一个迭代器，以斐波那契数列为例::

    class Fib:
       def __init__(self):
           self.prev = 0
           self.curr = 1
       def __iter__(self):
           return self
       def __next__(self):
           value = self.curr
           self.curr += self.prev
           self.prev = value
           return value
    >>> f = Fib()
    >>> list(islice(f, 0, 10))
    [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]

.. note:: 

   -  ``Fib`` 既是一个可迭代对象（因为它实现了 ``__iter__`` 方法）
   - 又是一个迭代器（因为实现了  ``__next__`` 方法）
   - 实例变量  ``prev`` 和  ``curr`` 用户维护迭代器内部的状态
   - 每次调用  ``next()`` 方法的时候做两件事
      #. 为下一次调用 ``next()`` 方法修改状态
      #. 为当前这次调用生成返回结果


``迭代器`` 就像一个懒加载的工厂，等到有人需要的时候才给它生成值返回，没调用的时候就处于休眠状态等待下一次调用   


生成器(generator)
-----------------------

``生成器`` 其实是一种特殊的迭代器，不过这种迭代器更加优雅,
它不需要再像上面的类一样写 ``__iter__()`` 和  ``__next__()`` 方法了，只需要一个  ``yiled`` 关键字。

``生成器`` 一定是 ``迭代器`` (反之不成立)，因此任何生成器也是以一种懒加载的模式生成值。


用生成器来实现斐波那契数列的例子::

   def fib():
      prev, curr = 0, 1
      while True:
          yield curr
          prev, curr = curr, curr + prev
   >>> f = fib()
   >>> list(islice(f, 0, 10))
   [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]

``fib`` 就是一个普通的python函数，它特需的地方在于函数体中没有  ``return`` 关键字，函数的返回值是一个生成器对象。

当执行  ``f=fib()`` 返回的是一个 **生成器对象** ，此时函数体中的代码并不会执行，只有显示或隐示地调用 ``next`` 的时候才会真正执行里面的代码。

生成器在Python中是一个非常强大的编程结构，可以用更少地中间变量写流式代码，此外，相比其它容器对象它更能节省内存和CPU，当然它可以用更少的代码来实现相似的功能。

但凡看到类似::

   def something():
      result = []
      for ... in ...:
          result.append(x)
      return result

都可以用生成器函数来替换::

   def iter_something():
      for ... in ...:
          yield x

生成器表达式(generator expression)
----------------------------------------

``生成器表达式`` 是 ``列表推倒式的生成器版本`` ，看起来像列表推导式，但是它返回的是一个生成器对象而不是列表对象

::

   >>> a = (x*x for x in range(10))
   >>> a
   <generator object <genexpr> at 0x401f08>
   >>> sum(a)
   285





