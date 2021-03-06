touch 命令
===================

.. contents:: Sections:
  :local:
  :depth: 2

概述
----------

linux的 ``touch`` 命令不常用，一般在使用make的时候可能会用到;
可以用来修改文件的时间戳,或是创建一个新的空文件.

命令格式
-------------
::

   touch [选项]... 文件...  

命令参数
-------------

``-a``   或--time=atime或--time=access或--time=use 　只更改存取时间。

``-c``   或--no-create 　不建立任何文档。

``-d`` 　使用指定的日期时间，而非现在的时间。

``-f`` 　此参数将忽略不予处理，仅负责解决BSD版本touch指令的兼容性问题。

``-m``  或--time=mtime或--time=modify 　只更改变动时间。

``-r`` 　把指定文档或目录的日期时间，统统设成和参考文档或目录的日期时间相同。

``-t`` 　使用指定的日期时间，而非现在的时间。   


命令功能
----------
1. 用于把已存在文件的时间标签更新为系统当前的时间（默认方式），它们的数据将原封不动地保留下来；

2. 用来创建新的空文件。

使用范例
----------

创建不存在的文件
^^^^^^^^^^^^^^^^^^^^^^^^^

::

   touch log2012.log log2013.log

更新log.log的时间和log2012.log时间戳相同
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

   touch -r log.log log2012.log   

设定文件的时间戳
^^^^^^^^^^^^^^^^^^^

::

   touch -t 201211142234.50 log.log   

.. note::
   ``-t  time`` 使用指定的时间值 ``time`` 作为指定文件相应时间戳记的新值．此处的 time规定为如下形式的十进制数:      
       
   - [[CC]YY]MMDDhhmm[.SS]    
   