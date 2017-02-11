Startup类
=================

.. contents:: Sections:
   :local:
   :depth: 2


简介
---------------

``Startup`` 类提供了应用程序的入口，而且在所有应用程序中都有 ``Startup`` 类。

* ASP.NET 会在主程序集中搜索名为 ``Startup`` 的类（在任何命名空间下）
* 可以指定一个其它程序集用于检索，只需使用 ``Hosting:Application`` 配置键
* ASP.NET 并不关心 ``Startup`` 类是不是定义为 ``public`` ，如果它符合命名规范，ASP.NET 将继续加载它
* 有多个 ``Startup`` 类，也不会触发异常，ASP.NET 将基于命名空间选择其中一个
    *  匹配项目的根命名空间优先，否则使用第一个按字母排列的命名空间中的类

Configure 方法
----------------------

``Configure`` 方法用于指定 ASP.NET 应用程序将如何响应每一个 HTTP 请求。

也就是说，通过 ``Configure`` 方法可以配置每个请求都接受相同的响应。

.. note:: 
   - 更复杂的管道配置可以封装于 ``中间件（middleware）``  之中，并通过扩展方法添加到 ``IApplicationBuilder`` 上。

``Configure`` 方法必须接受一个 ``IApplicationBuilder`` 参数。

一些额外服务，比如 ``IHostingEnvironment`` 或 ``ILoggerFactory`` 也可以被指定，如果在它们可用情况下，这些服务将会被服务器 注入 进来   

.. note::
   - ``UseMvc`` 扩展方法会把 **路由中间件** 加进请求管道，并把 MVC 配置为默认的处理器.


ConfigureServices 方法
------------------------

**可选地** 包含一个 ``ConfigureServices`` 方法用来配置用于应用程序内的服务,通过 **依赖注入（dependency injection）** 可将服务加入服务容器，使其在应用程序中可用

``ConfigureServices`` 方法是 ``Startup`` 类中的公开方法，通过参数获取一个 ``IServiceCollection`` 实例并 **可选地返回** ``IServiceProvider``.

``ConfigureServices`` 需要在 ``Configure`` 之前被调用 .

.. note::
   这是因为像 ASP.NET MVC 中的某些功能，需要从 ``ConfigureServices`` 中请求某些服务，而这些服务需要在接入请求管道之前先被加入 ``ConfigureServices`` 中


在启动时服务可用
----------------

ASP.NET Core 在应用程序启动期间提供了一些应用服务和对象。你可以非常简单地使用这些服务，只需要在在 ``Startup`` 类的构造函数或是它的 ``Configure`` 与 ``ConfigureServices`` 方法中的一个包含合适的接口即可.

IApplicationBuilder
""""""""""""""""""""""""

被用于构建应用程序的请求管道.

- 只可以在 ``Startup`` 中的 ``Configure`` 方法里使用

IApplicationEnvironment
"""""""""""""""""""""""""

提供了访问应用程序属性，类似于 ``ApplicationName`` 、``ApplicationVersion`` 以及 ``ApplicationBasePath`` 。

- 可以在 ``Startup`` 的构造函数和  ``Configure`` 方法中使用。

IHostingEnvironment
"""""""""""""""""""""

提供了当前的 ``EnvironmentName`` 、 ``WebRootPath`` 以及 Web 根文件提供者.

- 可以在 ``Startup`` 的构造函数和 ``Configure`` 方法中使用.

ILoggerFactory
""""""""""""""""""

提供了创建日志的机制.

- 可以在 ``Startup`` 的构造函数或 ``Configure`` 方法中使用.

IServiceCollection
""""""""""""""""""""""""

当前容器中各服务的配置集合.

- 只可在 ``ConfigureServices`` 方法中被使用，通过在该方法中配置可使服务在应用程序中可用.

.. note::
   总结 Startup类中每个函数参数可用的服务如下：
        - **Startup Constructor** - ``IApplicationEnvironment`` - ``IHostingEnvironment`` - ``ILoggerFactory``

        - **ConfigureServices** - ``IServiceCollection``
        
        - **Configure** - ``IApplicationBuilder`` - ``IApplicationEnvironment`` - ``IHostingEnvironment`` - ``ILoggerFactory``
