WebService基本概念
====================

.. contents:: Sections:
  :local:
  :depth: 3


什么是 ``Web Service`` ?
----------------------------
``Web WebService`` 可以使应用程序成为Web应用程序，应用通过web进行发布、查找和使用，数据通过网络仅此传输和交换.

* 是 **应用程序组件**
* 使用开发协议( ``SOAP`` ) 进行通信
* 是独立的（self-contained）并可自我描述
* 可通过使用 ``UDDI`` 来发现
*  可被其他应用程序使用 
* ``XML`` 是 WebService的基础 


Web Service是如何工作的?
---------------------------

**Web Service 平台 = XML + HTTP**

* ``HTTP`` 协议是最常用的因特网协议
* ``XML`` 提供了一种可用于不同平台和编程语言

Web Service 平台的元素
-----------------------
* ``SOAP`` (简单对象访问协议)
* ``UDDI`` (用于描述、发现和整合)
* ``WSDL`` (Web Service 的描述语言)

什么是 SOAP？
^^^^^^^^^^^^^^^^
* ``SOAP`` 指简易对象访问协议
* ``SOAP`` 是一种通信协议
* ``SOAP`` 用于应用程序之间的通信
* ``SOAP`` 是一种用于发送消息的格式
* ``SOAP`` 被设计用来通过因特网进行通信
* ``SOAP`` 独立于平台
* ``SOAP`` 独立于语言
* ``SOAP`` 基于 XML
* ``SOAP`` 很简单并可扩展
* ``SOAP`` 允许您绕过防火墙
* ``SOAP`` 将作为 W3C 标准来发展

什么是 WSDL?
^^^^^^^^^^^^^^^^^^^

``WSDL`` 是基于 ``XML`` 的用于描述 Web Services 以及如何访问 Web Services 的 **语言**

* ``WSDL`` 指网络服务描述语言
* ``WSDL`` 使用 XML 编写
* ``WSDL`` 是一种 XML 文档
* ``WSDL`` 用于描述网络服务
* ``WSDL`` 也可用于定位网络服务
* ``WSDL`` 还不是 W3C 标准

什么是UDDI？
^^^^^^^^^^^^

``UDDI`` 是一种 **目录服务** ，通过它，企业可注册并搜索 ``Web services``

* ``UDDI`` 指通用的描述、发现以及整合（Universal Description, Discovery and Integration）。
* ``UDDI`` 是一种用于存储有关 web services 的信息的目录。
* ``UDDI`` 是一种由 WSDL 描述的网络服务接口目录。
* ``UDDI`` 经由 SOAP 进行通迅。
* ``UDDI`` 被构建于 Microsoft .NET 平台之中。

使用Web Service的原因
------------------------

* ``Web services`` 把 Web 应用程序提升到了另外一个层面
    
* 通过使用 ``Web services`` ，应用程序可向全世界发布功能或消息。``Web services`` 使用 ``XML`` 来编解码数据，并使用 ``SOAP`` 借由开放的协议来传输数据。

Web services 的应用类型
-------------------------

#. 可重复使用的应用程序组件（分布式应用）    
#. 连接现有的软件   