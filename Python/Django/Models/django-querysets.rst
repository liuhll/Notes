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


跨越多值的关联关系
^^^^^^^^^^^^^^^^^^

Filter 可以引用模型的字段
-----------------------------
