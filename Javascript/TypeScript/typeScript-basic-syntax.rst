TypeScript基础语法
=======================

.. contents:: Sections:
  :local:
  :depth: 2

基础类型
---------------------

布尔值
^^^^^^^^^^^^^^
``boolean`` 类型

值： ``true`` / ``false`` 

数字
^^^^^^^^^^^
``number``

字符串
^^^^^^^^^^^^^
``string`` 表示文本数据类型

模版字符串
""""""""""""""""
被反引号包围（  ````` ），并且以 ``${ expr }`` 这种形式嵌入表达式

数组
----------------
有两种方式可以定义数组

1. 在元素类型后面接上 ``[]`` ，表示由此类型元素组成的一个数组。
   ::
   
      let list: number[] = [1, 2, 3];
   

2. 使用数组泛型，``Array<元素类型>``
   ::
   
      let list: Array<number> = [1, 2, 3];  
   

元组 Tuple
--------------------
表示一个已知元素数量和类型的数组，各元素的类型不必相同
::

    // Declare a tuple type
    let x: [string, number];
    // Initialize it
    x = ['hello', 10]; // OK
    // Initialize it incorrectly
    x = [10, 'hello']; // Error


根据索引访问元组的元素时，会得元素相应的类型
::

   console.log(x[0].substr(1)); // OK
   console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'


访问一个越界的元素，会使用联合类型替代
::

   x[3] = 'world'; // OK, 字符串可以赋值给(string | number)类型
   
   console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString
   
   x[6] = true; // Error, 布尔不是(string | number)类型

枚举
-----------------
``enum`` 类型是对JavaScript标准数据类型的一个补充

从 ``0`` 开始为元素编号。 也可以手动的指定成员的数值

枚举类型提供的一个便利是,可以由枚举的值得到它的名字

::

  enum Color {Red = 1, Green, Blue};
  let colorName: string = Color[2];
  
  alert(colorName);

任意值
--------------
不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用 ``any`` 类型来标记这些变量
   
::

   let notSure: any = 4;
   notSure = "maybe a string instead";
   notSure = false; // okay, definitely a boolean

变量声明
---------------------
