Soap 协议
==============

.. contents:: Sections:
  :local:
  :depth: 3

什么是 Soap?
---------------

``SOAP`` （Simple Object Access Protocol ）: 中文名称为: **简单对象访问协议** , 是在分散或分布式的环境中交换信息的简单的协议，是一个基于XML的协议;可使应用程序在 ``HTTP`` 之上进行信息交换;

更简单地说： ``SOAP`` 是用于 **访问网络服务的协议**

* ``SOAP`` 指简易 **对象访问协议**
* ``SOAP`` 是一种 **通信协议**
* ``SOAP`` 用于 **应用程序之间** 的通信
* ``SOAP`` 是一种用于 **发送消息的格式**
* ``SOAP`` 被设计用来 **通过因特网** 进行通信
* ``SOAP`` 独立于平台
* ``SOAP`` 独立于语言
* ``SOAP`` 基于 XML
* ``SOAP`` 很简单并 **可扩展** 
* ``SOAP`` 允许您绕过防火墙
* ``SOAP`` 将被作为 W3C 标准来发展

为什么使用 SOAP？
-----------------

对于应用程序开发来说， **使程序之间进行因特网通信是很重要的** ;

通过 ``HTTP`` 在应用程序间通信是更好的方法，因为 ``HTTP`` 得到了所有的因特网浏览器及服务器的支持。``SOAP`` 就是被创造出来完成这个任务的。

``SOAP`` 提供了一种标准的方法， **使得运行在不同的操作系统并使用不同的技术和编程语言的应用程序可以互相进行通信** 。

SOAP 语法
-----------

Soap消息的基本结构
^^^^^^^^^^^^^^^^^^^^^^

一条 ``SOAP`` 消息就是一个普通的 ``XML`` 文档，包含下列元素:

* **必需的** ``Envelope`` 元素，可把此 XML 文档标识为一条 SOAP 消息
* **可选的** ``Header`` 元素，包含 **头部** 信息
* **必需的** ``Body`` 元素，包含所有的调用和响应信息 (消息的正文)
* **可选的** ``Fault`` 元素，提供有关在处理此消息所发生错误的信息

SOAP 消息的基本结构::
   
   <?xml version="1.0"?>
   <soap:Envelope
   xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
   soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
   
   <soap:Header>
   ...
   </soap:Header>
   
   <soap:Body>
   ...
     <soap:Fault>
     ...
     </soap:Fault>
   </soap:Body>
   
   </soap:Envelope>

语法规则
^^^^^^^^^^

* ``SOAP`` 消息 *必须* 用 ``XML`` 来编码
* ``SOAP`` 消息 *必须* 使用 ``SOAP Envelope`` 命名空间
* ``SOAP`` 消息 *必须* 使用 ``SOAP Encoding`` 命名空间
* ``SOAP`` 消息 *不能* 包含 DTD 引用
* ``SOAP`` 消息 *不能* 包含 XML 处理指令

SOAP Envelope 元素
---------------------

* ``Envelope`` 被要求强制使用的元素

* ``Envelope`` 元素是 SOAP 消息的 **根元素** 。
 
* ``Envelope`` 把 XML 文档定义为 SOAP 消息 。

实例::

   <?xml version="1.0"?>
   <soap:Envelope
   xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
   soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
     ...
     Message information goes here
     ...
   </soap:Envelope>

xmlns:soap 命名空间
^^^^^^^^^^^^^^^^^^^^
SOAP 消息必须拥有与命名空间 ``http://www.w3.org/2001/12/soap-envelope`` 相关联的一个 ``Envelope`` 元素。

如果使用了不同的命名空间，应用程序会发生错误，并抛弃此消息。   

encodingStyle 属性
^^^^^^^^^^^^^^^^^^^^
SOAP 的 ``encodingStyle`` 属性用于定义在文档中使用的数据类型。
此属性可出现在任何 SOAP 元素中，并会 **被应用到元素的内容及元素的所有子元素上** 。

语法::

   soap:encodingStyle="URI"

实例::

   <?xml version="1.0"?>
   <soap:Envelope
   xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
   soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
     ...
     Message information goes here
     ...
   </soap:Envelope>   

SOAP Header 元素
-------------------

* ``Header`` 元素是可选的,
* 包含头部信息(有关 SOAP 消息的应用程序专用信息 比如认证、支付等)

.. note::

   * 如果 ``Header`` 元素被提供，则它必须是 ``Envelope`` 元素的第一个子元素
   * 所有 ``Header`` 元素的直接子元素必须是合格的命名空间

mustUnderstand 属性
^^^^^^^^^^^^^^^^^^^^^

标识标题项对于要对其进行处理的接收者来说是强制的还是可选的   

语法::

   soap:mustUnderstand="0|1"

actor 属性
^^^^^^^^^^^^^

被用于将 ``Header`` 元素 **寻址到一个特定的端点**

语法::

   soap:actor="URI"   

encodingStyle 属性
^^^^^^^^^^^^^^^^^^^^

Body 元素
-------------

* 该元素要求 **强制使用** .
* 包含打算传送到消息最终端点的实际 ``SOAP`` 消息 .
* ``Body`` 元素的直接子元素可以是合格的命名空间

实例::

   <?xml version="1.0"?>
   <soap:Envelope
   xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
   soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
   
   <soap:Body>
     <m:GetPrice xmlns:m="http://www.w3schools.com/prices">
       <m:Item>Apples</m:Item>
     </m:GetPrice>
   </soap:Body>
   
   </soap:Envelope>

Fault 元素
-----------------

* 可选的元素
* 用于存留 SOAP 消息的错误和状态信息
* 如果已提供了 ``Fault`` 元素，则它必须是 ``Body`` 元素的子元素
* 在一条 SOAP 消息中，``Fault`` 元素只能出现一次

Fault 可拥有的子元素
^^^^^^^^^^^^^^^^^^^^
.. list-table::
   :header-rows: 1

   * - 子元素
     - 描述
   * - <faultcode>	
     - 供识别故障的代码
   * - <faultstring>	
     - 可供人阅读的有关故障的说明
   * - <faultactor>	
     - 有关是谁引发故障的信息
   * - <detail>	
     - 存留涉及 Body 元素的应用程序专用错误信息