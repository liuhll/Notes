应用程序状态的管理
======================

.. contents:: Sections:
   :local:
   :depth: 2

在 ASP.NET Core 中，有多种途径可以对应用程序的状态进行管理， **取决于检索状态的时机和方式** 。

什么是应用程序状态
---------------------

``应用程序状态`` 指的是用于描述应用程序 **当前状况的任意数据** 。包括的数据有:

-  **全局的** 

- **用户特有**  

.. note::
    ASP.NET Core 中， ``Application`` 已经没有了；可以用 ``Caching`` 的实现来代替 ``Application`` 的功能，从而把之前版本的 ASP.NET 应用程序升级到 ASP.NET Core

存储数据状态的依据
-------------------
- 数据需要储存多久？
- 数据有多大？
- 数据的格式是什么？
- 数据是否可以序列化？
- 数据有多敏感？能不能保存在客户端？

根据这些问题的答案，可以选择不同的方式储存

应用程序状态的可选方式
-------------------------

HttpContext.Items
^^^^^^^^^^^^^^^^^^^^^^

当数据仅用于一个请求之中时，用 ``Items`` 集合储存是最好的方式。

数据将 **在每个请求结束之后被丢弃** 。它可以作为组件和中间件在一个请求期间的不同时间点进行互相通讯的最佳手段。

QueryString 和 Post
^^^^^^^^^^^^^^^^^^^^^^^^^^
在查询字符串（ ``QueryString`` ）中添加数值、或利用 ``POST`` 发送数据， **可以将一个请求的状态数据提供给另一个请求**。

这种技术 **不应该用于敏感数据** ，因为这需要将数据发送到客户端，然后再发送回服务器。这种方法也最好用。

这种方法也最好用于少量的数据。查询字符串对于持久地保留状态特别有用，可以将状态嵌入链接通过电子邮件或社交网络发出去，以备日后使用。

Cookies
^^^^^^^^^^^^
与状态有关的非常小量的数据可以储存在 ``Cookies`` 中。他们会随每次请求被发送，所以应该保持在最小的尺寸。

理想情况下，应该只使用一个标识符，而真正的数据储存在服务器端的某处， **键值与这个标识符关联** 。


Session
^^^^^^^^^^^^^

会话（ ``Session`` ）储存依靠一个基于 ``Cookie`` 的标识符来访问与给定浏览器（来自一个特定机器和特定浏览器的一系列访问请求）会话相关的数据。不能假设一个会话只限定给了一个用户，因此要慎重考虑在会话中储存哪些信息。这是用来储存那种针对具体会话，但又不要求永久保持的（或者说，需要的时候可以再从持久储存中重新获取的）应用程序状态的好地方

Cache
^^^^^^^

缓存（ ``Caching`` ）提供了一种方法，用开发者自定义的键对应用程序数据进行储存和快速检索。它提供了一套基于时间和其他因素来使缓存项目过期的规则


Configuration
^^^^^^^^^^^^^^^^^^^

配置（ ``Configuration`` ）可以被认为是应用程序状态储存的另外一种形式，不过通常它在程序运行的时候是 **只读** 的

其他持久化
^^^^^^^^^^^^^^

任何其他形式的持久化储存，无论是 Entity Framework 和数据库还是类似 Azure Table Storage 的东西，都可以被用来储存应用程序状态

HttpContext.Items的使用
-------------------------

``HttpContext`` 抽象提供了一个简单的 ``IDictionary<object, object>`` 类型的字典集合，叫作 ``Items`` 。在每个请求中，这个集合从 ``HttpRequest`` 开始起就可以使用，直到请求结束后被丢弃。要存取集合，你可以直接给键控项赋值，或根据给定键查询值。 

例子
^^^^^^^

在 ``Items`` 中增加一个配置项

::

    app.Use(async (context, next) =>
       {
         // perform some verification
         context.Items["isVerified"] = true;
         await next.Invoke();
       });

在之后的管道中，其他的中间件就可以访问到这些内容了

::

   app.Run(async (context) =>
   {
     await context.Response.WriteAsync("Verified request? "
       + context.Items["isVerified"]);
   });

.. note::

    ``Items`` 的键名是简单的字符串，所以如果你是在开发跨越多个应用程序工作的中间件，你可能要用一个唯一标识符作为前缀以避免键名冲突。（如：采用"MyComponent.isVerified"，而非简单的"isVerified"）

Session的使用
---------------

安装和配置 Session
^^^^^^^^^^^^^^^^^^^^^^

1. ASP.NET Core 发布了一个关于会话的程序包，里面提供了用于管理会话状态的中间件。你可以在 ``project.json`` 中加入对 ``Microsoft.AspNetCore.Session`` 的引用来安装这个程序包：

2. 当安装好程序包后，必须在你的应用程序的 ``Startup`` 类中对 ``Session`` 进行配置。

   ``Session`` 是基于 ``IDistributedCache`` 构建的，因此你也必须把它配置好，否则会得到一个错误

.. note::
   如果你一个 ``IDistributedCache`` 的实现都没有配置，则会得到一个异常，说“在尝试激活 'Microsoft.AspNetCore.Session.DistributedSessionStore' 的时候，无法找到类型为 'Microsoft.Extensions.Caching.Distributed.IDistributedCache' 的服务。”

IDistributedCache 的实现
"""""""""""""""""""""""""""

ASP.NET 提供了 ``IDistributedCache`` 的多种实现， ``in-memory`` 是其中之一（ **仅用于开发期间和测试** ）。要配置会话采用 ``in-memory`` ，需将 ``Microsoft.Extensions.Caching.Memory`` 依赖项加入你的 ``project.json`` 文件，然后再把以下代码添加到 ``ConfigureServices``

::

   services.AddDistributedMemoryCache();
   services.AddSession();

3. 使用 ``Session`` 中间件

::

   app.UseSession();

ISession
^^^^^^^^^^^^

一旦 ``Session`` 安装和配置完成，你就可以通过 ``HttpContext`` 的一个名为 ``Session`` ，类型为 ``ISession`` 的属性来引用会话了。   

::

   public interface ISession
   {
     bool IsAvailable { get; }
     string Id { get; }
     IEnumerable<string> Keys { get; }
     Task LoadAsync();
     Task CommitAsync();
     bool TryGetValue(string key, out byte[] value);
     void Set(string key, byte[] value);
     void Remove(string key);
     void Clear();
     IEnumerable<string> Keys { get; }
   }

``Session`` 是建立在 ``IDistributedCache`` 之上的，所以总是需要序列化被储存的对象实例   

