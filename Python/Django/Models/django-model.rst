模型
============

.. contents:: Sections:
   :local:
   :depth: 2
   
模型的概念
--------------

**模型** 是数据的唯一的、权威的信息源。它包含你所储存数据的必要 ``字段`` 和 ``行为`` 。
通常， **每个模型对应数据库中唯一的一张表** 。

基础

#. 每个模型都是 ``django.db.models.Model`` 的一个Python 子类
#. 模型的每个属性都表示为数据库中的一个字段
#. Django 提供一套自动生成的用于数据库访问的API(通过模型的管理器)

如何使用模型
---------------
定义好模型之后，接下来你需要告诉Django 使用这些模型。

#. 修改配置文件中的 ``INSTALLED_APPS`` 设置，在其中添加 ``models.py`` 所在应用的名称

   例如::
   
      INSTALLED_APPS = (
      #...
      'myapp',
      #...
      )

#. 使用 ``manage.py makemigrations`` 给应用生成迁移脚本 

#. 运行命令 ``manage.py migrate`` ,将模型迁移到数据库 

.. note:: 
   * 关于数据库迁移，请查看 :doc:`./django-model-migrate`  


字段
---------------   

**字段** 由 ``models`` 类属性指定。要注意选择的字段名称不要和 模型API 冲突，比如 ``clean`` 、``save`` 或者 ``delete``。
详见 `模型字段参考 <http://python.usyiyi.cn/documents/django_182/ref/models/fields.html#field-types>`_  ;


字段类型
^^^^^^^^^^

模型中的每个字段都是 ``Field`` 子类的某个实例。

Django 根据字段类的类型确定以下信息：

* 数据库当中的列类型 (比如: ``INTEGER`` , ``VARCHAR`` )
* 渲染表单时使用的默认 HTML 部件（例如，``<input type="text">`` , ``<select>`` ）
* 最低限度的 *验证需求* ，它被用在 Django 管理站点和自动生成的表单中

字段选项
^^^^^^^^^^^^

每个字段有一些特有的参数，详见 `模型字段参考 <http://python.usyiyi.cn/documents/django_182/ref/models/fields.html#field-options>`_  ;
有一些适用于所有字段的通用参数。

最常用的字段选项：
"""""""""""""""""""

* null
    如果为 ``True`` ，Django 将用 ``NULL`` 来在数据库中存储空值。 默认值是 ``False``

* blank  
    如果为 ``True`` ，该字段允许不填。默认为 ``False``    
       .. note::
          * 与 ``null`` 不同。``null`` 纯粹是数据库范畴,指数据库中字段内容是否允许为空，
          * 而 ``blank`` 是表单数据输入验证范畴的。如果一个字段的 ``blank=True`` ，表单的验证将允许该字段是空值。
          * 如果字段的 ``blank=False`` ，该字段就是必填的      

* choices
      * 由二项元组构成的一个 ``可迭代对象`` （例如，列表或元组），用来给字段提供选择项
      * 如果设置了 ``choices``  ，默认的表单将是一个 **选择框** 而不是标准的文本框，而且这个选择框的选项就是 ``choices`` 中的选项


* default
   字段的默认值。
   
   可以是一个值或者可调用对象。如果是可调用对象 ，每有新对象被创建它都会被调用

   .. note::
      Python中有一个有趣的语法，只要定义类型的时候，实现__call__函数，这个类型就成为可调用的
   
* primary_key
   如果为 ``True`` ，那么这个字段就是模型的 **主键**
      .. note::
         * 没有指定任何一个字段的 ``primary_key=True`` ，Django 就会自动添加一个 ``IntegerField``  字段做为主键;
         * 如果想覆盖默认的主键行为，否则没必要设置任何一个字段的 ``primary_key=True``

* unique 
   如果该值设置为 ``True`` , 这个数据字段的值在整张表中必须是唯一的  

自增主键字段
^^^^^^^^^^^^^^^^
默认情况下，Django 会给每个模型添加下面这个字段::

   id = models.AutoField(primary_key=True)

如果需要自定义主键:

* 只要在某个字段上指定 ``primary_key=True`` 即可;如果设置了 ``Field.primary_key`` ，Django就不会自动添加 ``id`` 列

.. warning:: 每个模型只能有一个字段指定primary_key=True（无论是显式声明还是自动添加）

字段的自述名
^^^^^^^^^^^^^^^^

除 ``ForeignKey`` 、``ManyToManyField`` 和 ``OneToOneField`` (关系键)之外，每个字段类型都接受一个可选的位置参数 —— **字段的自述名**

如果 *没有给定自述名* ，Django 将根据字段的属性名称自动创建自述名 —— **将属性名称的下划线替换成空格**

键的自述名
^^^^^^^^^^^^^^^^

``ForeignKey`` 、 ``ManyToManyField`` 和 ``OneToOneField`` 都要求第一个参数是一个模型类，所以要使用 ``verbose_name`` 关键字参数才能指定自述名
   ::
   
      poll = models.ForeignKey(Poll, verbose_name="the related poll")
      sites = models.ManyToManyField(Site, verbose_name="list of sites")
      place = models.OneToOneField(Place, verbose_name="related place")


关系
----------------

关系数据库的威力体现在表之间的相互关联,Django 提供了三种最常见的数据库关系： **多对一(many-to-one)** ， **多对多(many-to-many)** ， **一对一(one-to-one)**

多对一关系
^^^^^^^^^^^^
Django 使用 ``django.db.models.ForeignKey`` 定义多对一关系(在模型当中把它做为一个类属性包含进来)

``ForeignKey`` 需要一个位置参数：与该模型关联的类

::

   from django.db import models
   
   class Manufacturer(models.Model):
      # ...
      pass
   
   class Car(models.Model):
      manufacturer = models.ForeignKey(Manufacturer)
      # ...


.. tip::
   建议使用被关联的模型的小写名称做为 ``ForeignKey`` 字段的名字,如上例所示，但也可以使用其他名称

多对多关系
^^^^^^^^^^^^^
``ManyToManyField`` 用来定义多对多关系，用法和其他 ``Field`` 字段类型一样： **在模型中做为一个类属性包含进来**

建议以被关联模型名称的复数形式做为 ``ManyToManyField`` 的名字   

在哪个模型中设置 ``ManyToManyField`` 并不重要，在两个模型中任选一个即可 

.. note::

   不要两个模型都设置

通常， ``ManyToManyField`` 实例应该位于可以编辑的表单中。

::

   from django.db import models
   
   class Topping(models.Model):
       # ...
       pass
   
   class Pizza(models.Model):
       # 因为设想一个Pizza 有多种Topping 比一个Topping 位于多个Pizza 上要更加自然。
       # 在Pizza 的表单中将允许用户选择不同的Toppings。
       toppings = models.ManyToManyField(Topping)   


多对多关系中的其他字段
""""""""""""""""""""""""""
如何解决需要关联数据到两个模型之间的关系上？

Django 允许你指定一个 ``中介模型`` 来定义多对多关系，可以将其他字段放在中介模型里面，源模型的 ``ManyToManyField`` 字段将使用 ``through`` 参数指向 ``中介模型``

在设置 ``中介模型`` 时，要显式地指定外键并关联到多对多关系涉及的模型。

中介模型有一些限制：
* 中介模型必须有且只有一个外键到源模型，或者你必须使用 ``ManyToManyField.through_fields``  显式指定Django 应该在关系中使用的外键
* 对于通过 **中介模型与自己进行多对多关联的模型** ，允许存在到同一个模型的两个外键，但它们将被当做多对多关联中一个关系的两边; *如果你的模型中存在不止一个外键，并且through_fields没有指定，将会触发一个无效的错误*
* 使用中介模型定义与自身的多对多关系时，你必须设置 ``symmetrical=False``
* 与普通的多对多字段不同，你不能使用 ``add`` 、 ``create`` 和赋值语句（比如，``beatles.members = [...]`` ）来创建关系,也不能使用 ``remove()`` 来移除关系
* 但是 ``clear()`` 方法却是可用的。它可以清空某个实例所有的多对多关系


::

   # 例子：音乐家以及他加入的音乐小组
   from django.db import models
   
   class Person(models.Model):
       name = models.CharField(max_length=128)
   
       def __str__(self):              # __unicode__ on Python 2
           return self.name
   
   class Group(models.Model):
       name = models.CharField(max_length=128)
       members = models.ManyToManyField(Person, through='Membership')
   
       def __str__(self):              # __unicode__ on Python 2
           return self.name
   
   # 更多成员关系的细节，比如成员是何时加入小组的(中介模型)
   class Membership(models.Model):
       # 需要在中介模型中，要显示的指定外键
       person = models.ForeignKey(Person)
       group = models.ForeignKey(Group)
       date_joined = models.DateField()
       invite_reason = models.CharField(max_length=64)      


创建中介模型的实例::

   # 创建个人1
   >>> ringo = Person.objects.create(name="Ringo Starr")
   # 创建个人2
   >>> paul = Person.objects.create(name="Paul McCartney")
   # 创建一个组1
   >>> beatles = Group.objects.create(name="The Beatles")
   # 创建一个关系，将个人1和组1作为参数传入构造器中
   >>> m1 = Membership(person=ringo, group=beatles,
   ...     date_joined=date(1962, 8, 16),
   ...     invite_reason="Needed a new drummer.")
   #保存关系
   >>> m1.save()
   >>> beatles.members.all()
   [<Person: Ringo Starr>]
   >>> ringo.group_set.all()
   [<Group: The Beatles>]
   # 使用管理器的create方法创建一个关系
   >>> m2 = Membership.objects.create(person=paul, group=beatles,
   ...     date_joined=date(1960, 8, 1),
   ...     invite_reason="Wanted to form a band.")
   >>> beatles.members.all()
   [<Person: Ringo Starr>, <Person: Paul McCartney>]       


多对对的查询
""""""""""""""""

可以直接使用被关联模型的属性进行查询::

   # Find all the groups with a member whose name starts with 'Paul'
   >>> Group.objects.filter(members__name__startswith='Paul')
   [<Group: The Beatles>]

可以利用中介模型的属性进行查询::

   # Find all the members of the Beatles that joined after 1 Jan 1961
   >>> Person.objects.filter(
   ...     group__name='The Beatles',
   ...     membership__date_joined__gt=date(1961,1,1))
   [<Person: Ringo Starr]

可以直接获取Membership模型::

   >>> ringos_membership = Membership.objects.get(group=beatles, person=ringo)
   >>> ringos_membership.date_joined
   datetime.date(1962, 8, 16)
   >>> ringos_membership.invite_reason
   'Needed a new drummer.'

一对一关系
^^^^^^^^^^^^^^
``OneToOneField`` 用来定义一对一关系。 用法和其他字段类型一样： **在模型里面做为类属性包含进来**

一对一的关系一般用于什么场景？
**当某个对象想扩展自另一个对象时** ，最常用的方式就是在这个对象的主键上添加一对一关系,(类似于一种继承的关系,某一个对象扩展了另外一个对象的属性)

**OneToOneField** 要一个位置参数： 与模型关联的类

跨文件的模型
-----------------

在文件顶部你定义模型的地方，导入相关的模型来实现它。然后，无论在哪里需要的话，都可以引用它。

::

   from django.db import models
   from geography.models import ZipCode
   
   class Restaurant(models.Model):
       # ...
       zip_code = models.ForeignKey(ZipCode)
   
字段命名的限制
-----------------

#. 字段的名称 **不能是Python 保留的关键字** ，因为这将导致一个Python 语法错误
#. 由于Django 查询语法的工作方式， **字段名称中连续的下划线不能超过一个**

::

   class Example(models.Model):
       foo__bar = models.IntegerField() # 'foo__bar' has two underscores!

.. note::
   SQL 的保留字例如 ``join`` 、 ``where``  和 ``select``  ，可以用作模型的字段名，因为Django 会对底层的SQL 查询语句中的数据库表名和列名进行转义

模型元数据 Meta(元数据)
---------------------------

使用内部的 ``class Meta`` 定义模型的 **元数据**

什么是模型元数据?
**任何不是字段的数据** ，比如排序选项（ ``ordering`` ），数据库表名（ ``db_table`` ）或者人类可读的单复数名称（ ``verbose_name`` 和 ``verbose_name_plural``)

在模型中添加 ``class Meta`` 是完全 **可选** 的，所有选项都不是必须的

::

   from django.db import models
   
   class Ox(models.Model):
       horn_length = models.IntegerField()
   
       class Meta:
           ordering = ["horn_length"]
           verbose_name_plural = "oxen"

`模型元选项 <http://python.usyiyi.cn/documents/django_182/ref/models/options.html>`_  


模型的属性
---------------

``模型管理器`` 是Django 模型进行数据库查询操作的接口，并用于从数据库获取实例;

如果没有自定义 ``Manager`` ，则默认的名称为 ``objects``

``Managers`` **只能通过模型类访问，而不能通过模型实例访问**

模型的方法
------------------

可以在模型上 **定义自定义的方法** 来给你的对象添加自定义的“底层”功能

``Manager`` 方法用于 **表范围** 的事务

**模型的方法** 应该着眼于特定的模型实例,这是一个非常有价值的技术，让业务逻辑位于同一个地方 —— 模型中

.. tip::
   这体现了设计模式的 `单一职责原则` 

模型的自定义方法例子::

   from django.db import models
   
   class Person(models.Model):
       first_name = models.CharField(max_length=50)
       last_name = models.CharField(max_length=50)
       birth_date = models.DateField()
   
       def baby_boomer_status(self):
           "Returns the person's baby-boomer status."
           import datetime
           if self.birth_date < datetime.date(1945, 8, 1):
               return "Pre-boomer"
           elif self.birth_date < datetime.date(1965, 1, 1):
               return "Baby boomer"
           else:
               return "Post-boomer"
   
       def _get_full_name(self):
           "Returns the person's full name."
           return '%s %s' % (self.first_name, self.last_name)
       # 这是一个属性
       full_name = property(_get_full_name)

一般情况下，需要重新定义模型的一些预定义方法： ``__str__()`` ， ``__unicode__()`` ,  ``get_absolute_url()`` ...

重写预定义的模型方法
-----------------------

根据项目的实际业务情形，可能需要对模型的预定义方法进行重写；可以自由覆盖这些方法（和其它任何模型方法）来改变它们的行为

在开发过程中，可能需要经常改变 ``save()`` 和 ``delete()`` 的工作方式

例如::

    from django.db import models
    
    class Blog(models.Model):
        name = models.CharField(max_length=100)
        tagline = models.TextField()
    
        def save(self, *args, **kwargs):
            do_something()
            super(Blog, self).save(*args, **kwargs) # Call the "real" save() method.
            do_something_else()
            
或者阻止保存::

   from django.db import models
   
   class Blog(models.Model):
       name = models.CharField(max_length=100)
       tagline = models.TextField()
   
       def save(self, *args, **kwargs):
           if self.name == "Yoko Ono's blog":
               return # Yoko shall never have her own blog!
           else:
               super(Blog, self).save(*args, **kwargs) # Call the "real" save() method.


.. note::
   1. 记住调用超类的方法—— ``super(Blog, self).save(*args, **kwargs)``  —— 来确保对象被保存到数据库中。如果你忘记调用超类的这个方法，默认的行为将不会发生且数据库不会有任何改变
   2. 批量操作中被覆盖的模型方法不会被调用
   3. 当使用查询集批量删除对象时，将不会为每个对象调用 ``delete()`` 方法。为确保自定义的删除逻辑得到执行，你可以使用 ``pre_delete`` 和/或 ``post_delete`` 信号。


模型继承
---------------------

自定义的模型类应该继承 ``django.db.models.Model``

在Django 中有3种风格的继承:

#. 你只想使用父类来持有一些信息，你不想在每个子模型中都敲一遍。这个类永远不会单独使用，所以你要使用 ``抽象基类`` (元数据的 ``abstract=True``)
#. 继承一个已经存在的模型且想让每个模型具有它自己的数据库表，那么应该使用 ``多表继承``
#. 如果你只是想改变一个模块Python 级别的行为，而不用修改模型的字段，你可以使用 ``代理模型``

抽象基类
^^^^^^^^^^^^^^^^^^^^

将一些 **共有信息** 放进其它一些 ``model`` 的时候，抽象化类是十分有用的

在 ``Meta类`` 中设置 ``abstract=True`` ，这个模型就不会被用来创建任何数据表

抽象积累无法生成一张数据表或者拥有一个管理器，并且不能实例化或者直接储存

::

   from django.db import models
   
   class CommonInfo(models.Model):
       name = models.CharField(max_length=100)
       age = models.PositiveIntegerField()
   
       class Meta:
           abstract = True
   
   class Student(CommonInfo):
       home_group = models.CharField(max_length=5)
 
.. warning:: 
   如果抽象基类和它的子类有相同的字段名，那么将会出现 ``error`` （并且Django将抛出一个 ``exception`` ）


元继承
""""""""""""""""
如果子类想要扩展父类的 ``Meta类`` ，它可以作为其子类   

::

   from django.db import models
   
   class CommonInfo(models.Model):
       # ...
       class Meta:
           abstract = True
           ordering = ['name']
   
   class Student(CommonInfo):
       # ...
       class Meta(CommonInfo.Meta):
           db_table = 'student_info'

.. note::
   Django 会对抽象基类的 ``Meta类`` 做一个调整：在设置 ``Meta属性`` 之前，Django 会设置 ``abstract=False``  (抽象基类的子类本身不会自动变成抽象)         

小心使用 related_name
"""""""""""""""""""""""""

多表继承
^^^^^^^^^^^^^^
使用这种继承方式时，每一个层级下的每个 ``model`` 都是一个真正意义上完整的 ``model`` 。

每个 ``model`` 都有专属的数据表，都可以查询和创建数据表,继承关系在子 ``model`` 和它的每个父类之间都添加一个链接 (通过一个自动创建的 ``OneToOneField`` 来实现)
(使用隐含的 ``OneToOneField`` 来链接子类与父类)

::

   from django.db import models
   
   class Place(models.Model):
       name = models.CharField(max_length=50)
       address = models.CharField(max_length=80)
   
   class Restaurant(Place):
       serves_hot_dogs = models.BooleanField(default=False)
       serves_pizza = models.BooleanField(default=False)

多表继承中的Meta
"""""""""""""""""""
子类继承父类的 ``Meta类`` 是 **没什么意义的**

但是在某些受限的情况下，子类可以从父类继承某些 ``Meta`` ：如果子类没有指定 ``ordering`` 属性 或  ``get_latest_by`` 属性，它就会从父类中继承这些属性

如果父类有了排序设置，而你并不想让子类有任何排序设置，你就可以显式地禁用排序::

   class ChildModel(ParentModel):
       # ...
       class Meta:
           # Remove parent's ordering effect
           ordering = []

继承与反向关联
""""""""""""""""""""

如果你与该父类的另一个子类做多对一或是多对多关系，你就必须在每个多对一和多对多字段上强制指定 ``related_name`` 。如果你没这么做，Django 就会在你运行 验证( ``validation`` )  时抛出异常

代理继承
^^^^^^^^^^^^^^

如果我们只想更改 ``model`` 在 Python 层的行为实现。比如：更改默认的 ``manager`` ，或是添加一个新方法,并不想在数据库中创建一张新的数据库表，那么我们就使用代理集成。

-- 为原始模型创建一个代理。
#. 可以创建，删除，更新代理 ``model`` 的实例，而且所有的数据都可以像使用原始 ``model`` 一样被保存
#. 可以在 ``代理model`` 中改变默认的排序设置和默认的 ``manager`` ，更不会对原始 ``model`` 产生影响

例子::

   from django.db import models
   
   class Person(models.Model):
       first_name = models.CharField(max_length=30)
       last_name = models.CharField(max_length=30)
   
   class MyPerson(Person):
       class Meta:
           proxy = True
   
       def do_something(self):
           # ...
           pass

MyPerson类和它的父类 Person 操作同一个数据表。特别的是，Person 的任何实例也可以通过 MyPerson访问，反之亦然::

   >>> p = Person.objects.create(first_name="foobar")
   >>> MyPerson.objects.get(first_name="foobar")
   <MyPerson: foobar>           

查询集始终返回请求的模型
""""""""""""""""""""""""
它会使用依赖于原生Person的代码，而你可以使用你添加进来的扩展对象（它不会依赖其它任何代码）。而并不是将Person模型（或者其它）在所有地方替换为其它你自己创建的模型

基类的限制
""""""""""""""

代理模型 **必须** 继承自一个非抽象基类。 你不能继承自多个非抽象基类，这是因为一个代理 ``model`` 不能连接不同的数据表

代理 ``model`` 也可以继承任意多个抽象基类，但前提是它们 **没有 定义任何 ``model`` 字段**   

代理模型的管理器
"""""""""""""""""

如果你没有在代理 模型中定义任何管理器 ，代理模型就会从父类中继承管理器 。

如果你在代理模型中定义了一个管理器 ，它就会 **变成默认的管理器** ，不过定义在父类中的管理器 **仍然有效**

如果你想要向代理中添加新的管理器，而不是替换现有的默认管理器，你可以使用自定义管理器管理器文档中描述的技巧：

* 创建一个含有新的管理器的基类，并继承时把他放在主基类的后面
   
  ::

      # Create an abstract class for the new manager.
      class ExtraManagers(models.Model):
          secondary = NewManager()
      
          class Meta:
              abstract = True
      
      class MyPerson(Person, ExtraManagers):
          class Meta:
              proxy = True

代理模型与非托管模型之间的差异
""""""""""""""""""""""""""""""
``代理model``  继承看上去和使用 ``Meta类``中的   ``managed`` 属性的非托管 ``model`` 非常相似。但两者并不相同，使用时选用哪种方案是一个值得考虑的问题              

1. 如果你要借鉴一个已有的 模型或数据表，且不想涉及所有的原始数据表的列，那就令 ``Meta.managed=False``

2. 如果你想对 ``model`` 做 Python 层级的改动，又想保留字段不变，那就令 ``Meta.proxy=True`` 。因此在数据保存时，代理 model 相当于完全复制了原始 模型的存储结构

多重继承
-------------
django的模型可以继承自多个父类模型

一般来说， **你并不需要继承多个父类** 。多重继承主要对 ``mix-in`` 类有用：向每个继承 ``mix-in`` 的类添加一个特定的、额外的字段或者方法。你应该尝试将你的继承关系保持得尽可能简洁和直接，这样你就不必费很大力气来弄清楚某段特定的信息来自哪里。

