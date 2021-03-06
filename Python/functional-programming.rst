Python函数式编程入门以及lambda/map/reduce/filter使用
=====================================================

.. contents:: Sections:
   :local:
   :depth: 2


函数式编程的概念
-----------------------

``函数式编程`` 是一种"编程范式"（programming paradigm），也就是如何编写程序的方法论

``函数式编程`` 是一种抽象程度很高的编程范式，纯粹的函数式编程语言编写的函数没有变量，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。而允许使用变量的程序设计语言，由于函数内部的变量状态不确定，同样的输入，可能得到不同的输出，因此，这种函数是有副作用的 

函数式编程的特点
^^^^^^^^^^^^^^^^

``函数式编程`` 的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数

1. 函数是 **第一等公民**

   所谓 **第一等公民（first class）** ，指的是函数与其他数据类型一样，处于平等地位，可以赋值给其他变量，也可以作为参数，传入另一个函数，或者作为别的函数的返回值


2. 只用"表达式"，不用"语句"
   
   "表达式"（expression）是一个单纯的运算过程，总是有返回值

   "语句"（statement）是执行某种操作，没有返回值

   函数式编程要求，只使用表达式，不使用语句,也就是说，每一步都是单纯的运算，而且都有返回值。 

3. 没有"副作用"

   "副作用"（side effect），指的是 **函数内部与外部互动**，产生运算以外的其他结果
   
   函数式编程强调没有"副作用"，意味着函数要保持独立，所有功能就是返回一个新的值，没有其他行为，尤其是不得修改外部变量的值

4. 不修改状态

   函数式编程只是返回新的值，不修改系统变量。因此， **不修改变量** ，也是它的一个重要特点

   函数式编程 **使用参数保存状态** ，最好的例子就是递归

   
5. 引用透明

   引用透明（Referential transparency），指的是函数的运行不依赖于外部变量或"状态"，只依赖于输入的参数，任何时候只要参数相同，引用函数所得到的返回值总是相同的


.. note:: 
   相比面向对象编程，函数式编程的一大优势就是 **Immutable Data** (数据不可变)，就是 不依赖于外部的数据，而且也不改变外部数据的值 ，这种思想可以大大减少我们代码的Bug，而且函数式编程也支持我们像 使用变量一样使用函数

函数式编程的意义
^^^^^^^^^^^^^^^^^^
1. 代码简洁，开发快速

2. 接近自然语言，易于理解

3. 更方便的代码管理
   
   函数式编程不依赖、也不会改变外界的状态，只要给定输入参数，返回的结果必定相同。

4. 易于"并发编程"

   函数式编程不需要考虑 "死锁"（deadlock），因为它不修改变量，所以根本不存在"锁"线程的问题。不必担心一个线程的数据，被另一个线程修改，所以可以很放心地把工作分摊到多个线程，部署"并发编程"（concurrency）。

5. 代码的热升级

   函数式编程没有副作用，只要保证接口不变，内部实现是外部无关的。所以， **可以在运行状态下直接升级代码，不需要重启，也不需要停机**


Python函数式编程
----------------------

lambda的使用
^^^^^^^^^^^^^^^^^
``lambda`` 即匿名函数，合理地使用lambda不仅可以减少我们的代码量，而且也可以更好地描绘代码逻辑

::

   # lambda后面的x表示lambda函数要接收的参数，x + x表示lambda函数所要返回的值
   >>> f = lambda x: x + x
   
   # 可以看到f现在也是一个函数对象
   >>> f
   <function __main__.<lambda>>
   
   # 调用lambda函数
   >>> f(2)
   4

map的使用
^^^^^^^^^^^^^^^^^

``map( function , iterable )`` 接收两个参数，第一个参数代表的是 接收一个函数 ，第二个参数代表的是 接收一个 ``iteralbe`` 类型的对象，比如 ``list``   

map函数的作用
"""""""""""""""
事实上map函数把运算规则抽象了,把 ``iterable``对象中的每一项都通过 ``function`` 进行加工


map函数的原理：
""""""""""""""""""
1. 每次从 ``iterable`` 中取出一个参数
2. 将这个参数传递给我们的函数
3. 然后函数返回的值加入一个 ``list`` ( 这种说法不准确，只是为了帮助理解)，``map``函数返回的是一个map对象



例子1
::

   # 还是用我们上面那个lambda的例子
   >>> function = lambda x: x + x
   
   # 定义一个iterable对象list(列表)
   >>> iterable = [1, 2, 3, 4, 5, 6, 7, 8, 9]
   
   # 函数fucntion每次从iterable中取出一个参数x，然后function返回x + x的值，
   # 并将返回值加入一个新建的list，等将iterable遍历完，map就将这个新建的list返回。
   >>> v = map(function, iterable)
   
   # 注意上面的说法并不准确，只是为了帮助大家理解，其实map返回的是一个map对象，并不是   list
   >>> v
   <map at 0x7fcb56231588>
   
   # 但是我们可以调用内建的list函数将map转换成一个list来得到我们想要的结果
   >>> list(v)
   [2, 4, 6, 8, 10, 12, 14, 16, 18]
   
对于 ``map`` 的第二个参数，我们也可以传递一组函数列表进去，也就是说列表中间包含多个函数对象。   

例子2

::

   >>> multiply = lambda x: x * x
   
   >>> add = lambda x: x + x
   
   >>> funcs = [multiply, add]
   
   >>> list(map(lambda f: f(1), funcs))
   [1, 2]


reduce的使用
^^^^^^^^^^^^^^^

``reduce( function , iterable )`` 也接收两个参数，第一个参数代表的是接收一个函数，第二个参数代表的是接收一个 ``iteralbe类型`` 的对象，比如 ``list``, ``reduce``  函数必须要接收两个参数  

reduce函数的作用
""""""""""""""""""
``reduce`` 把 ``function`` 每次运行的结果继续和序列的下一个元素做累积计算

求一个 ``list`` (列表)累加和的例子

::

   from functools import reduce
   
   # 使用lambda定义一个函数，函数的作用是接收两个参数，然后返回两个参数之和
   
   >>> function = lambada x, y: x+y
   
   >>> iterable = [1, 2, 3, 4, 5, 6, 7, 8, 9]
   
   # 函数function每次接收两个参数，除第一次外每次从iterable中取一个元素作为一个参数
   # 另外一个参数取自上一次function返回的值
   >>> reduce(function,  iterable)
   45


filter的使用
^^^^^^^^^^^^^^^^^

``filter( function , iterable )`` 一次也接收两个参数，一个参数是函数，另外一个参数是 ``iterable对象``

filter的作用
""""""""""""""

从名字也可以看出, ``filter`` 用于过滤 ``iterble`` 对象，比如说 ``list`` (列表)


filter的原理
"""""""""""""""

它的原理是每次从 ``iterable`` 对象中取出一个元素作用于我们的 ``function`` ，如果 ``function`` 返回 ``True`` 就保留该元素，如果返回 ``False`` 就删除该元素

例子1

::

   # 定义一个函数，如果接收的字符s为空，那么返回False，如果为非空，那么返回True
   >>> function = lambda s : s and s.strip()
   
   >>> iterable = ['AJ', ' ', 'Stussy', '', 'CLOT', 'FCB', None]
   
   >>> filter(function, iterable)
   <filter at 0x7fcb562319b0>
   
   >>> list(filter(function, iterable))
   ['AJ', 'Stussy', 'CLOT', 'FCB']