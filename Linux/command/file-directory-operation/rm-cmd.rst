rm命令
===========

.. contents:: Sections:
  :local:
  :depth: 2

概述
---------

``rm`` 是常用的命令，该命令的功能为删除一个目录中的一个或多个文件或目录，它也可以将某个目录及其下的所有文件及子目录均删除。对于链接文件，只是删除了链接，原有文件均保持不变  

``rm`` 是一个危险的命令，因为一旦删除了一个文件，就无法再恢复它,使用的时候要特别当心，尤其对于新手，否则整个系统就会毁在这个命令（比如在 ``/``（根目录）下执行 ``rm * -rf`` ）

命令格式
---------

::

   rm (选项)(参数)

命令功能
----------
删除一个目录中的一个或多个文件或目录，如果没有使用 ``-r`` 选项，则 ``rm`` 不会删除目录

命令参数
-------------

-d：直接把欲删除的目录的硬连接数据删除成0，删除该目录；

-f：强制删除文件或目录；

-i：删除已有文件或目录之前先询问用户；

-r, -R, --recursive：递归处理，将指定目录下的所有文件与子目录一并处理； 

--preserve-root：不对根目录进行递归操作； 

-v：显示指令的详细执行过程。

实例
--------

自定义回收站功能
^^^^^^^^^^^^^^^^
::

   myrm(){ D=/tmp/$(date +%Y%m%d%H%M%S); mkdir -p $D; mv "$@" $D && echo "moved to $D ok"; }


使用例子

::

   [root@localhost test]# myrm(){ D=/tmp/$(date +%Y%m%d%H%M%S); mkdir -p    $D;  mv "$@" $D && echo "moved to $D ok"; }
   [root@localhost test]# alias rm='myrm'
   [root@localhost test]# touch 1.log 2.log 3.log
   [root@localhost test]# ll
   总计 16
   -rw-r--r-- 1 root root    0 10-26 15:08 1.log
   -rw-r--r-- 1 root root    0 10-26 15:08 2.log
   -rw-r--r-- 1 root root    0 10-26 15:08 3.log
   drwxr-xr-x 7 root root 4096 10-25 18:07 scf
   drwxrwxrwx 2 root root 4096 10-25 17:46 test3
   drwxr-xr-x 2 root root 4096 10-25 17:56 test4
   drwxr-xr-x 3 root root 4096 10-25 17:56 test5
   [root@localhost test]# rm [123].log
   moved to /tmp/20121026150901 ok
   [root@localhost test]# ll
   总计 16drwxr-xr-x 7 root root 4096 10-25 18:07 scf
   drwxrwxrwx 2 root root 4096 10-25 17:46 test3
   drwxr-xr-x 2 root root 4096 10-25 17:56 test4
   drwxr-xr-x 3 root root 4096 10-25 17:56 test5
   [root@localhost test]# ls /tmp/20121026150901/
   1.log  2.log  3.log
   [root@localhost test]#

.. note::
   
   上面的操作过程模拟了回收站的效果，即删除文件的时候只是把文件放到一个临时目录中，这样在需要的时候还可以恢复过来。