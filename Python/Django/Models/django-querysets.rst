查询集
=============

.. contents::
   :local:
   :depth: 3

一旦你建立好数据模型，Django 会自动为你生成一套 **数据库抽象的API** ,可以让你 *创建* 、*检索* 、 *更新* 和 *删除* 对象

该文档基于以下模型::

   # 一个基础博客应用的模型
   from django.db import models
   
   class Blog(models.Model):
       name = models.CharField(max_length=100)
       tagline = models.TextField()
   
       def __str__(self):              # __unicode__ on Python 2
           return self.name
   
   class Author(models.Model):
       name = models.CharField(max_length=50)
       email = models.EmailField()
   
       def __str__(self):              # __unicode__ on Python 2
           return self.name
   
   class Entry(models.Model):
       blog = models.ForeignKey(Blog)
       headline = models.CharField(max_length=255)
       body_text = models.TextField()
       pub_date = models.DateField()
       mod_date = models.DateField()
       authors = models.ManyToManyField(Author)
       n_comments = models.IntegerField()
       n_pingbacks = models.IntegerField()
       rating = models.IntegerField()
   
       def __str__(self):              # __unicode__ on Python 2
           return self.headline    


创建和保存对象
---------------
一个模型类代表数据库中的一个表，一个模型类的实例代表这个数据库表中的一条特定的记录

使用 ``关键字参数`` 实例化模型实例来创建一个对象，然后调用 ``save()`` 把它保存到数据库中

* 在你显式调用 ``save()`` 之前，Django 不会访问数据库(这与模型管理器的 ``create()`` 方法不同， ``create()`` 将创建并将数据插入到数据库中去)

* ``save()`` 方法没有返回值

保存对象的改动
^^^^^^^^^^^^^^^^
如果数据已经存在了某个实例的记录，那么该实例调用 ``save()`` 的时候，在背后执行SQL 的 ``UPDATE`` 语句。

保存ForeignKey和ManyToManyField字段
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
更新 ``ForeignKey`` 字段的方式和保存普通字段相同 

* 只要把一个正确类型的对象赋值给该字段    
    ::
    
       >>> from blog.models import Entry
       >>> entry = Entry.objects.get(pk=1)
       >>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
       >>> entry.blog = cheese_blog
       >>> entry.save()
   
* 为了在一条语句中，向 ``ManyToManyField`` 添加多条记录，可以在调用 ``add()`` 方法时传入多个参数 
   ::

      >>> john = Author.objects.create(name="John")
      >>> paul = Author.objects.create(name="Paul")
      >>> george = Author.objects.create(name="George")
      >>> ringo = Author.objects.create(name="Ringo")
      >>> entry.authors.add(john, paul, george, ringo)


获取对象
-------------------

通过模型中的 **管理器** 构造一个 **查询集** ，来从你的数据库中获取对象,每个模型都至少有一个管理器，它默认命名为 ``objects`` 。通过 ``模型类`` 来直接访问它

.. note::
   **管理器** 只可以通过模型的类访问，而不可以通过模型的实例访问，目的是为了强制区分“表级别”的操作和“记录级别”的操作。

**查询集** 表示从数据库中取出来的对象的集合。它可以含有零个、一个或者多个 **过滤器** 。过滤器基于所给的参数限制查询的结果。 从SQL 的角度，查询集和 ``SELECT`` 语句等价，过滤器是像 ``WHERE`` 和 ``LIMIT`` 一样的限制子句

获取所有对象
^^^^^^^^^^^^^
使用管理器的 ``all()`` 方法::

   >>> all_entries = Entry.objects.all()

使用过滤器获取特定对象
^^^^^^^^^^^^^^^^^^^^^^

通常情况下，我们期望得到的是通过条件筛选特定的子集。这需要在原始的的查询集上增加一些过滤条件

两个最普遍的途径:

#. filter(**kwargs)
   
   返回一个新的查询集，它包含 **满足** 查询参数的对象

#. exclude(**kwargs)
   
   返回一个新的查询集，它包含 **不满足** 查询参数的对象

``查询参数`` （上面函数定义中的 ``**kwargs`` ）需要满足特定的格式


链式过滤
"""""""""""""""

**查询集** 的筛选结果本身还是 **查询集** ，所以可以将筛选语句链接在一起


过滤后的查询集是独立的
""""""""""""""""""""""

每次你筛选一个 **查询集** ，得到的都是全新的另一个 **查询集** ，它和之前的 **查询集** 之间 **没有任何绑定关系** 

每次筛选都会创建一个独立的查询集，它可以被存储及反复使用


查询集是惰性执行的
""""""""""""""""""""""

查询集 是 **惰性执行** 的 —— **创建查询集不会带来任何数据库的访问**,
直到查询集 需要求值时，Django 才会真正运行这个查询


只有在 **请求** 查询集 的结果时才会到数据库中去获取它们

::

   >>> q = Entry.objects.filter(headline__startswith="What")
   >>> q = q.filter(pub_date__lte=datetime.date.today())
   >>> q = q.exclude(body_text__icontains="food")
   # 只有在调用下面语句时才访问一次数据库
   >>> print(q)

通过get 获取一个单一的对象
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
只有一个对象满足你的查询，你可以使用管理器的 ``get()`` 方法，它直接返回该对象

::

   >>> one_entry = Entry.objects.get(pk=1)

对 ``get()`` 使用任何查询表达式，和 ``filter()`` 一样

.. note::

    * 使用 ``get()`` 和使用 ``filter()`` 的切片 ``[0]`` 有一点区别
         * 如果没有结果满足查询， ``get()`` 将引发一个 ``DoesNotExist``  异常。这个异常是正在查询的模型类  的一   个属性 —— 所以在上面的代码中，如果没有主键为1 的Entry 对象，Django 将引发一个      ``Entry.DoesNotExist``         
         * 如果有多条记录满足 ``get()`` 的查询条件，Django 也将报错
    * 更多关于查询的请参考 `查询集 API <http://python.usyiyi.cn/documents/django_182/ref/models/querysets.html#queryset-api>`_     

限制查询集
^^^^^^^^^^^^^^^^

使用Python 的 ``切片`` 语法来限制查询集记录的数目,它等同于SQL 的 ``LIMIT`` 和 ``OFFSET`` 子句

查询集 的切片返回一个新的查询集 —— 它不会执行查询,如果你使用Python 切片语法中 ``step`` 参数,它将真实执行查询

::

   >>> Entry.objects.all()[:10:2]

若要获取一个单一的对象而不是一个列表（例如，``SELECT foo FROM bar LIMIT 1`` ），可以简单地使用一个索引而不是切片

::

   >>> Entry.objects.order_by('headline')[0]
   # 它大体等同于
   >>> Entry.objects.order_by('headline')[0:1].get()

.. note::

   如果没有对象满足给定的条件，第一条语句将引发 ``IndexError`` 而第二条语句将引发 ``DoesNotExist``


字段查询
--------------
字段查询是指如何指定SQL ``WHERE`` 子句的内容,它们通过查询集方法 ``filter()`` 、``exclude()``  和 ``get()`` 的关键字参数指定。

查询的 ``关键字参数`` 的基本形式是 ``field__lookuptype=value``

::

   >>> Entry.objects.filter(pub_date__lte='2006-01-01')
   # 翻译成SQL（大体）是：
   SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';

查询条件中指定的字段 **必须是模型字段的名称**, 但有一个例外，对于 ``ForeignKey`` 你可以使用字段名加上 ``_id`` 后缀,在这种情况下，该参数的值应该是外键的原始值。

::

   >>> Entry.objects.filter(blog_id=4)

常见查询
^^^^^^^^^^^^^^^^
* exact
   
  “精确”匹配

* iexact   

  大小写不敏感的匹配

* contains
  
  大小写敏感的包含关系测试

* startswith, endswith

  分别表示以 ``XXX开头`` 和以 ``XXX结尾`` 。当然还有大小写不敏感的版本，叫做 ``istartswith`` 和 ``iendswith`` 。

跨关联关系的查询
--------------------

Django 提供一种强大而又直观的方式来“处理”查询中的关联关系，它在后台自动帮你处理 ``JOIN``

若要跨越关联关系，只需使用关联的模型字段的名称，并使用 ``双下划线`` 分隔，直至你想要的字段

这种跨越可以是 **任意的深度**

::

   # 获取所有Blog 的name 为'Beatles Blog' 的Entry 对象 
   >>> Entry.objects.filter(blog__name='Beatles Blog')


还可以反向工作。若要引用一个“反向”的关系，只需要使用该模型的小写的名称   

::

   #示例获取所有的Blog 对象，它们至少有一个Entry 的headline 包含'Lennon'
   >>> Blog.objects.filter(entry__headline__contains='Lennon')


跨越多值的关联关系
^^^^^^^^^^^^^^^^^^
当你基于 ``ManyToManyField`` 或 ``反向的ForeignKey`` 来过滤一个对象时，有两种不同种类的过滤器



Filter 可以引用模型的字段
^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果你想将模型的一个字段与同一个模型的另外一个字段进行比较该怎么办？

Django 提供 ``F 表达式`` 来允许这样的比较。

``F()`` 返回的实例 **用作查询内部对模型字段的引用** 。 **这些引用** 可以用于查询的 ``filter`` 中来比较相同模型实例上 **不同字段之间值的比较** 。


::

   # 查找comments 数目多于pingbacks 的Entry，我们将构造一个F() 对象来引用pingback 数   目，并在查询中使用该F() 对象
   >>> from django.db.models import F
   >>> Entry.objects.filter(n_comments__gt=F('n_pingbacks'))

Django 支持对 ``F()`` 对象使用 *加法* 、 *减法* 、 *乘法* 、 *除法* 、*取模* 以及 *幂计算* 等算术操作，两个操作数可以都是常数和其它 ``F()`` 对象


还可以在 ``F()`` 对象中使用 **双下划线标记** 来跨越关联关系

::

   #要获取author 的名字与blog 名字相同的Entry，我们可以这样查询
   >>> Entry.objects.filter(authors__name=F('blog__name'))


对于 ``date`` 和 ``date/time`` 字段，你可以给它们加上或减去一个 ``timedelta`` 对象   

::

   >>> from datetime import timedelta
   >>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))

``F()`` 对象支持 ``.bitand()`` 和 ``.bitor()`` 两种位操作

::

   >>> F('somefield').bitand(16)   


查询的快捷方式pk
------------------------

查询快捷方式 ``pk`` ，它表示 ``primary key`` 的意思

任何查询类型都可以与 ``pk`` 结合来完成一个模型上对主键的查询

::

   # Get blogs entries with id 1, 4 and 7
   >>> Blog.objects.filter(pk__in=[1,4,7])
   
   # Get all blog entries with id > 14
   >>> Blog.objects.filter(pk__gt=14)


``pk`` 查询在 ``join`` 中也可以工作   

::

   # 这三个语句是等同的
   >>> Entry.objects.filter(blog__id__exact=3) # Explicit form
   >>> Entry.objects.filter(blog__id=3)        # __exact is implied
   >>> Entry.objects.filter(blog__pk=3)        # __pk implies __id__exact

转义LIKE 语句中的百分号和下划线
--------------------------------------

与LIKE SQL 语句等同的字段查询（ ``iexact`` 、 ``contains`` 、``icontains`` 、 ``startswith`` 、 ``istartswith`` 、 ``endswith``  和 ``iendswith`` ）将自动转义在LIKE 语句中使用的两个特殊的字符 —— 百分号和下划线 

::

   >>> Entry.objects.filter(headline__contains='%') 
   #生成的SQL 看上去会是这样：
   SELECT ... WHERE headline LIKE '%\%%'; 

缓存和查询集
------------------

每个 **查询集** 都包含一个　**缓存** 来最小化对数据库的访问   

在一个新创建的 **查询集** 中，**缓存为空** 。首次对查询集进行求值 —— **同时发生** 数据库查询 ——Django 将保存查询的结果到查询集的缓存中并返回明确请求的结果（例如，如果正在迭代查询集，则返回下一个结果）。接下来对该查询集 的求值将重用缓存的结果

对查询集使用不当的话,会增加对性能的损耗

::

   #相同的数据库查询将执行两次，显然倍增了你的数据库负载。同时，还有可能两个结果列表并不   包含相同的数据库记录，因为在两次请求期间有可能有Entry被添加进来或删除掉
   >>> print([e.headline for e in Entry.objects.all()])
   >>> print([e.pub_date for e in Entry.objects.all()])
   
   #为了避免这个问题，只需保存查询集并重新使用它
   >>> queryset = Entry.objects.all()
   >>> print([p.headline for p in queryset]) # Evaluate the query set.
   >>> print([p.pub_date for p in queryset]) # Re-use the cache from the    evaluation.

何时查询集不会被缓存
^^^^^^^^^^^^^^^^^^^^^^^^^
**查询集** **不会永远** 缓存它们的结果。当只对查询集的部分进行求值时会检查缓存， 但是如果这个部分不在缓存中，那么接下来查询返回的记录都将不会被缓存。

这意味着使用 ``切片`` 或 ``索引`` 来限制查询集将 **不会** 填充缓存

简单地打印查询集 **不会** 填充缓存

::

   #重复获取查询集对象中一个特定的索引将每次都查询数据库
   >>> queryset = Entry.objects.all()
   >>> print queryset[5] # Queries the database
   >>> print queryset[5] # Queries the database again

::

   #如果已经对全部查询集求值过，则将检查缓存：
   >>> queryset = Entry.objects.all()
   >>> [entry for entry in queryset] # Queries the database
   >>> print queryset[5] # Uses cache
   >>> print queryset[5] # Uses cache


使用Q 对象进行复杂的查询
---------------------------

``filter()`` 等方法中的关键字参数查询都是一起进行 ``AND`` 的

如何执行更复杂的查询？(例如 ``OR`` 语句)

* 使用Q 对象

**Q 对象** ( ``django.db.models.Q`` ) 对象用于封装一组关键字参数。这些关键字参数就是上文 :ref:`字段查询` 中所提及的那些。

::

   #Q 对象封装一个LIKE 查询
   from django.db.models import Q
   Q(question__startswith='What')

** Q对象** 可以使用 ``&`` 和 ``|`` 操作符组合起来。当一个操作符在两个 **Q对象** 上使用时，它产生一个新的 **Q对象**, **Q对象** 可以使用 ``~`` 操作符取反，这允许组合正常的查询和 **取反( ``NOT`` ) 查询**

每个接受关键字参数的查询函数（例如 ``filter()`` 、 ``exclude()`` 、 ``get()`` ）都可以传递一个或多个Q 对象作为位置（不带名的）参数


::

   Q(question__startswith='Who') | Q(question__startswith='What')
   #它等同于下面的SQL WHERE 子句：   
   WHERE question LIKE 'Who%' OR question LIKE 'What%'


::

   Poll.objects.get(
       Q(question__startswith='Who'),
       Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
   )
   # 大体上可以翻译成这个SQL
   SELECT * from polls WHERE question LIKE 'Who%'
       AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')


查询函数 **可以混合使用Q 对象和关键字参数** 。所有提供给查询函数的参数（关键字参数或Q 对象）都将 ``AND`` 在一起。

.. note::

    如果混合使用Q 对象和关键字参数,Q 对象必须位于所有关键字参数的前面       


比较对象
-----------------

使用标准的Python 比较操作符，即双等于符号：``==``

比较两个模型 **主键的值**

删除对象
---------------

删除对象的方法是 ``delete()``, 这个方法将 **立即删除对象且没有返回值**

::

   e.delete()

可以批量删除对象。

* 每个查询集 都有一个  ``delete()`` 方法，它将删除该查询集中的所有成员.
   ::

      Entry.objects.filter(pub_date__year=2005).delete()


当Django 删除一个对象时，它默认使用 ``SQL ON DELETE CASCADE`` 约束 —— 换句话讲， **任何有外键指向要删除对象的对象将一起删除**

::

   b = Blog.objects.get(pk=1)
   # This will delete the Blog and all of its Entry objects.
   b.delete()

确实想删除所有的对象，你必须明确地请求一个完全的查询集::

   Entry.objects.all().delete()  

.. note:: 

   ``delete()`` 是唯一没有在管理器 上暴露出来的查询集方法。这是一个安全机制来防止你意外地请求 ``Entry.objects.delete()``
  
拷贝模型实例
-----------------

最简单的方法是，只需要将 ``pk`` 设置为 ``None``

::

   blog = Blog(name='My blog', tagline='Blogging is easy')
   blog.save() # blog.pk == 1
   
   blog.pk = None

如果是继承的模型实例，则必须设置 ``pk`` 和 ``id`` 都为 ``None``

::

   # 这个过程不会拷贝关联的对象
   django_blog.pk = None
   django_blog.id = None
   django_blog.save() # django_blog.pk == 4

一次更新多个对象
---------------------

使用 ``update()`` 方法, **查询集** 提供了 ``update()`` 方法可以一次性更新查询集内所有实体的字段值

对 ``update`` 的调用也可以使用 **F 表达式** 来根据模型中的一个字段更新另外一个字段
。这对于在当前值的基础上加上一个值特别有用

在 ``update`` 中你不可以使用 ``F()`` 对象引入 ``join`` —— 你只可以引用正在更新的模型的字段，如果你尝试使用 ``F()`` 对象引入一个 ``join`` ，将引发一个 ``FieldError``


关联的对象
---------------

在一个模型中定义一个关联关系时（例如， ``ForeignKey`` 、 ``OneToOneField`` 或 ``ManyToManyField`` ），该模型的实例将带有一个方便的API 来访问关联的对象

处理关联对象的其它方法
^^^^^^^^^^^^^^^^^^^^^^

* add(obj1, obj2, ...)
   * 添加一指定的模型对象到关联的对象集中。
* create(**kwargs)
   * 创建一个新的对象，将它保存并放在关联的对象集中。返回新创建的对象。
* remove(obj1, obj2, ...)
   * 从关联的对象集中删除指定的模型对象。
* clear()
   * 从关联的对象集中删除所有的对象。
