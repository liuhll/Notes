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
----------------
``ManyToManyField`` 用来定义多对多关系，用法和其他 ``Field`` 字段类型一样：在模型中做为一个类属性包含进来

   

