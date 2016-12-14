reStructuredText学习笔记
=========================

.. contents:: Sections:
  :local:
  :depth: 2

段落 Paragraphs
------------------
段落是 `reST` 文章中最常见的文本块. 段落是由一个或多个 **空白分隔** 的文本块, 所有同级段落,必须 **左对齐** ,使用 **同级缩进** 

行内标记 Inline markup
----------------------
* 单星号: *文本* 得 强调 (斜体 对中文一般效果不好) ,
* 双星号: **文本** 得 加重 (加黑),
* 反引号: ``文本`` 得 代码引用.

如果有 **星号** 或 **反引号** 出现在引用的文本, 就可能会弄乱内联标记分隔符,这时,可以用 ``反斜杠`` 来转义

.. note:: 
   - 1. 不可叠用
   - 2. 前后不能用空格: * text* 这样会出错 
   - 3. 必须和周围文字用非单词隔离, 一般使用转义空白来完成: thisis\ *one*\ word



文本诠释规则
^^^^^^^^^^^^^^^   
任意由指定字符封闭的文本都可以用特定的方式来诠释

* 一般语法形如: 
   * :规则名:`内容`

标准 ``reST`` 提供以下规则
"""""""""""""""""""""""""""""""
- emphasis – *emphasis* 的替代拼写
   - ``:emphasis:`斜体``` 
- strong – **strong** 的替代拼写
   - ``:strong:`加粗``` 
- literal – ``literal`` 的替代拼写
   - ``:literal:`代码引用```
- subscript – 下标
   - ``:subscript:`下标```
- superscript – 上标
   - ``:superscript:`上标```
- title-reference – 书籍/期刊/及其他材料的标题
   - ``:title-reference:`书籍/期刊/及其他材料的标题```

列表和引用块
-------------------
1. 只要自然的在段落的开始放置一个星号 ``*`` 并正确缩进
2. 使用 ``#`` 签署自动编号

线块 ``|`` 是保留换行符的一种方法::

| These lines are
| broken exactly like in
| the source file.


源代码 Source Code
---------------------
代码文本块 (参考) 由末尾是特殊标记 ``::`` 的段落引发. 整个代码文本块 **必须缩进** (同所有的段落一样,使用空白行和周围文本完成分隔)

``::`` 标记是智能处置的:
  - 如果作为一个 **独立段落** 出现,则和其它文本完全隔离
  - 如果它 **紧跟有空格** ,则将被删除 **不起作用**
  - 如果它 **在非空白字符之前** ,则替换为 **普通的单一冒号** 

.. note::
  - ``::`` 与代码块之间有空一行，空的这一行无缩进
  - 代码块与``::``使用缩进来进行标识

表格 Tables
-------------------
网格表
^^^^^^^^^^^^^^
需要绘制表格的边框，如下所示::

    +------------------------+------------+----------+----------+
    | Header row, column 1   | Header 2   | Header 3 | Header 4 |
    | (header rows optional) |            |          |          |
    +========================+============+==========+==========+
    | body row 1, column 1   | column 2   | column 3 | column 4 |
    +------------------------+------------+----------+----------+
    | body row 2             | ...        | ...      |          |
    +------------------------+------------+----------+----------+       


简单表 
^^^^^^^^^^
**有限制** :至少要有一列,而且,第一行不能包含多行文本::

     =====  =====  =======
     A      B      A and B
     =====  =====  =======
     False  False  False
     True   False  False
     False  True   False
     True   True   True
     =====  =====  =======

超链接 Hyperlinks
--------------------

外部链接 External links
^^^^^^^^^^^^^^^^^^^^^^^^^
1. 使用 ``Link text <http://example.com/>`_`` 定义外部链接,例如 `百度 <http://www.baidu.com>`_

2. 也可以单独定义链接目标用引用
例如::

   This is a paragraph that contains `a link`_.
   
   .. _a link: http://example.com/

内部链接 Internal links
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
使用特殊 ``reST`` 规则支持内部链接,请 `参考 <http://sphinx-doc-zh.readthedocs.io/en/latest/markup/inline.html#ref-role>`_。   
一般地，会使用下面三种指令实现内部链接

1. ``:ref:``

2. ``:doc:``

3. ``::download:``

章节 Sections
-------------------
章节头部 (参考) 由下线(也可有上线)和包含标点的标题 组合创建, 其中 **下线要至少等于标准文本的长度**
  
通常没有对标题的层级指定明确的标线字符. 不过,可以使用如下约定
  * ``#``  有上标线, 用以部分
  * ``*``  有上标线, 用以章节
  * ``=`` , 用以小节
  * ``-`` , 用以子节
  * ``^`` , 用以子节的子节
  * ``"`` , 用以段落

直解标记 Explicit Markup
----------------------------
用以 ``reST`` 中需要特殊处理的各种内容, 如脚注,特殊高亮段落,注释,以及通用指令

直解标记块由 ``..`` 开始,紧后跟空格以及跟随的同缩进的文本块

指令 Directives
----------------------
**指令(ref)** 就是一个标准的 ``明确标记(Explicit Markup)块`` ,它是 ``reST`` 的又一个扩展机制

支持以下指令:
  - 警示 Admonitions: ``attention`` , ``caution`` , ``danger``, ``error``, ``hint``, ``important``, ``note``, ``tip``, ``warning`` 等等
  - 图像 Images:
      - ``image``
      - ``figure``
  - 其它行文元素
      - ``contents`` (对诸如 本地文件 的内容表单)
      - ``container`` (配有定制 class 的容器,以便生成HTML 中的 <div> )
      - ``rubric`` (没有到相对段落关系的标题 a heading without relation to the document sectioning)
      - ``topic``, ``sidebar`` (特殊高亮的正文元素 special highlighted body elements)
      - ``parsed-literal`` (支持内嵌标记的文本块)
      - ``epigraph`` (有可选归属行的引用文本块)
      - ``highlights``, ``pull-quote`` (有他们自己class属性的文本块)
      - ``compound`` (复合段落)    
  - 特殊表格 Special tables
  - 特殊指令 Special directives

指令的用法
^^^^^^^^^^^^^
基本上一个指令由 **名称** , **参数** , **选项** 和 **内容** 组成::

  .. function:: foo(x)
              foo(y, z)
   :module: some.module.name
   
   Return a line of text input from the user.

图片 Images   
--------------------

使用方法::

   .. image:: gnu.png
    (options)

图片尺寸的解释选项 ( ``width`` 和 ``height``)有如下规约: 如果大小没给任何单位或单位是像素, 输出通道优先使用像素(换言之,非LaTeX输出). 其他单位(如 pt 或是 点) 将被用于HTML和LaTeX输出    

脚注 Footnotes
----------------------

使用 ``[#name]_`` 来标记位置, 并在文章底部 “Footnotes” 专栏之后追加脚注内容

用法::

  Lorem ipsum [#f1]_ dolor sit amet ... [#f2]_
  
  .. rubric:: Footnotes
  
  .. [#f1] Text of the first footnote.
  .. [#f2] Text of the second footnote.


引证 Citations
------------------
引证能从所有文件来引用，[引证]_ 的例子

用法::

  Lorem ipsum [Ref]_ dolor sit amet.
  
  .. [Ref] Book or article reference, URL or whatever.



.. [引证] 这是一个引证的例子  

替换 Substitutions
-----------------------

以 ``|name|`` 形式来定义替换的文本或是标记对象

用法::

  .. |name| replace:: replacement *text*

或是::

  .. |caution| image:: warning.png
               :alt: Warning!

注释 Comments
----------------------
没有有效标记(如脚注)的直解标记文本块就是注释::

  .. This is a comment.

可以用缩进文本来进行多行注释::

  ..
     This whole indented block
     is a comment.
  
     Still in the comment.               