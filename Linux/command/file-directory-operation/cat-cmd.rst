cat命令
==========

.. contents:: Sections:
  :local:
  :depth: 2


概述
-----------

``cat`` 命令的用途是 **连接文件** 或 **标准输入并打印**。 

它常与重定向符号配合使用; 

.. note::
   当文件较大时，文本在屏幕上迅速闪过（滚屏），用户往往看不清所显示的内容。因此，一般用 ``more`` 或是 ``less`` 等命令分屏显示。为了控制滚屏，可以按 ``Ctrl+S`` 键，停止滚屏；按 ``Ctrl+Q`` 键可以恢复滚屏。按 ``Ctrl+C`` （中断）键可以终止该命令的执行，并且返回Shell提示符状态


命令格式
-------------

::

   cat [选项] [文件]...

命令功能
-------------
1. 一次显示整个文件:

   ::
   
      cat filename

2. 从键盘创建一个文件: 只能创建新文件,不能编辑已有文件    

    ::

        cat > filename

3. 将几个文件合并为一个文件:
   
   ::
   
       cat file1 file2 > file

命令参数
---------------

-A, --show-all           等价于 -vET

-b, --number-nonblank    对非空输出行编号

-e                       等价于 -vE

-E, --show-ends          在每行结束处显示 $

-n, --number     对输出的所有行编号,由1开始对所有输出的行数编号

-s, --squeeze-blank  有连续两行以上的空白行，就代换为一行的空白行 

-t                       与 -vT 等价

-T, --show-tabs          将跳格字符显示为 ^I

-u                       (被忽略)

-v, --show-nonprinting   使用 ^ 和 M- 引用，除了 LFD 和 TAB 之外       

使用实例
-------------

1. 把 log2012.log 的文件内容加上行号后输入 log2013.log 这个文件里

   ::
   
      cat -n log2012.log log2013.log

2. 把 log2012.log 和 log2013.log 的文件内容加上行号（空白行不加）之后将内容附加到 log.log 里   

   ::
   
      cat -b log2012.log log2013.log log.log
   
3. 是输入输出定向符号的结合使用生成文档 

   ::
   
      [root@localhost test]# cat >log.txt <<EOF
      > Hello
      > World
      > Linux
      > PWD=$(pwd)
      > EOF   
