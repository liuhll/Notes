托管
===============

.. contents:: Sections:
   :local:
   :depth: 3

什么是宿主？
-------------

ASP.NET Core 应用程序需要在宿主中执行。

* 为了运行 ASP.NET Core 应用程序，你需要使用 ``WebHostBuilder`` 配置和启动一个宿主。

* 宿主必须实现 ``IWebHost`` 接口，这个接口暴露了功能和服务的集合，以及 ``Start`` 方法.

* 宿主通常使用 ``WebHostBuilder`` 的实例进行创建，该实例构建并返回一个 ``WebHost`` 实例。``WebHost`` 引用服务器来处理请求。

宿主和服务器的不同之处是什么？
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
宿主:

* 宿主负责应用程序启动和生命周期管理
* 确保应用程序服务和服务器可用并正确配置也是宿主职责的一部分。


服务器:
* 服务器负责接受 HTTP 请求

.. note::
   可以把宿主看作是服务器的包装。宿主被配置为使用一个特定的服务器；服务器并不知道它的宿主。

设置宿主
-----------
使用 ``WebHostBuilder`` 实例创建一个宿主.

这通常是在你的应用程序入口点： ``public static void Main`` ，（在项目模板的 ``Program.cs`` 文件中）

::

  using System;
  using System.Collections.Generic;
  using System.IO;
  using System.Linq;
  using System.Threading.Tasks;
  using Microsoft.AspNetCore.Hosting;
  
  namespace WebApplication1
  {
      public class Program
      {
          public static void Main(string[] args)
          {
              var host = new WebHostBuilder()
                  .UseKestrel()
                  .UseContentRoot(Directory.GetCurrentDirectory())
                  .UseIISIntegration()
                  .UseStartup<Startup>()
                  .Build();
  
              host.Run();
          }
      }
  }


* ``WebHostBuilder`` 负责创建宿主，宿主会启动应用程序服务器.
* ``WebHostBuilder`` 需要你提供实现了 ``IServer`` （上面代码中的 ``UseKestrel`` ） 接口的服务器
   * ``UseKestrel`` 指定应用程序会使用 ``Kestrel`` 服务器

服务器的 *内容根 ( content root )* 决定它将在哪里搜索内容文件，比如 MVC 视图文件。
* 默认的内容根是应用程序运行的文件夹   

如果应用程序需要使用 ``IIS`` ，需要在构建宿主时调用 ``UseIISIntegration`` 方法

* 为了让 ASP.NET Core 使用 ``IIS`` ，你必须同时指定 ``UseKestrel`` 和 ``UseIISIntegration``
    * ``Kestrel`` 被设计为在代理后运行而不应该直接部署到互联网。
    * ``UseIISIntegration`` 指定 IIS 为反向代理服务器
    
.. note::
   * ``UseKestrel`` 与 ``UseIISIntegration`` 行为区别非常大。
   * IIS 只是作为一个反向代理。 ``UseKestrel`` 创建 Web 服务器并且对代码进行托管。        ``UseIISIntegration`` 指定 IIS 作为反向代理服务器。它同时也检查了 IIS/IISExpress 用的   境变量   并做出比如使用哪个动态端口，设置什么 Header 等决定。然而它不处理或者创建 ``IServer``   
     
配置一个最小的宿主服务
^^^^^^^^^^^^^^^^^^^^^^

::

   var host = new WebHostBuilder()
       .UseKestrel()
       .Configure(app =>
       {
           app.Run(async (context) => await context.Response.WriteAsync("Hi!"));
       })
       .Build();
   
   host.Run();

.. note::
    当设置一个宿主，你可以提供 ``Configure`` 和 ``ConfigureServices`` 方法，或者定义一个 ``Startup`` 类（也必须定义这些方法）。多次调用 ConfigureServices 会进行追加配置；多次调用 ``Configure`` 或者 ``UseStartup`` 会替换之前的设置 

配置宿主
-----------

``WebHostBuilder`` 提供了方法用于为宿主设置大多数可用的配置值，它也可以被设置为直接使用  ``UseSetting`` 以及相关的键.

例子：

::

  new WebHostBuilder()
      .UseSetting("applicationName", "MyApp")

宿主配置值
^^^^^^^^^^^^^^

应用程序名 string
"""""""""""""""""""

**键** : ``applicationName`` 。这个配置设定指定的值将从 ``IHostingEnvironment.ApplicationName``  返回

捕获启动异常 bool
""""""""""""""""""""

**键** ： ``captureStartupErrors`` 。默认是 ``false`` 。

* 当值为 ``false`` 时，在启动过程中的错误会导致宿主退出。
* 当值为 ``true`` 时，宿主会捕捉 ``Startup`` 类中的任何异常，并试图启动服务器。

可使用 ``CaptureStartupErrors`` 方法设置

::

  new WebHostBuilder()
      .CaptureStartupErrors(true)


内容根 string
"""""""""""""""""""

**Key** : ``contentRoot`` 
* 这个设置决定了 ASP.NET Core 从哪里开始搜索内容文件，比如 MVC 视图。
* **内容根** 同时被作为 Web 根设置 的基础路径使用
* 可使用 ``UseContentRoot`` 方法设置。
* 路径必须是存在的，否则宿主会启动失败

::

   new WebHostBuilder()
       .UseContentRoot("c:\\mywebsite")

详细错误 bool
"""""""""""""""""""
**键** ： ``detailedErrors``
* 默认是 ``false`` 。
* 当值是 ``true`` 时（或者当环境设置为 ``Development`` 时），应用程序会显示详细的启动错误信息，而不仅仅是一般的错误页
    * 当详细错误设置为 ``false`` 并且捕捉启动异常是 ``true`` 时，服务器在每个请求的（错误）响应中显示一般错误页
    * 当详细错误设置为 ``true`` 并且捕捉启动异常是 ``true`` 时，服务器在每个请求的（错误）响应中显示详细错误页。

* 可使用 ``UseSetting`` 设置

::

   new WebHostBuilder()
       .UseSetting("detailedErrors", "true")


环境 string
""""""""""""""
**键** ： ``environment``

* 默认是 ``Production``。可以设置为任何值
* 框架定义的值包含 ``Development`` ， ``Staging`` ，以及 ``Production``。 
* 值不区分大小写
* 可使用 ``UseEnvironment`` 方法设置

::

   new WebHostBuilder()
       .UseEnvironment("Development")

服务器 URLs string
""""""""""""""""""""
**键** ： ``urls``
* 设置分号（;）来分隔服务器应该响应的 ``URL`` 前缀
* 域名可以用 ``*`` 替换，表明服务器需要针对任何使用指定端口及协议的 IP 地址或域名监听请求       
* 前缀由配置好的服务器解释；服务器之间支持的格式会有所不同

::

  new WebHostBuilder()
      .UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002")

启动程序集 string
""""""""""""""""""
**键** : ``startupAssembly``

* 决定搜索 ``Startup`` 类的程序集
* 可使用 ``UseStartup`` 方法设置
* 可以使用 ``WebHostBuilder.UseStartup<StartupType>`` 指定特定的引用类型
* 如果调用多次 ``UseStartup`` 方法， **最后一个调用的生效** 

::

   new WebHostBuilder()
       .UseStartup("StartupAssemblyName")

Web 根 string
""""""""""""""""
**键** ： ``webroot``

* 如果不指定，默认是 ``(Content Root Path)\wwwroot`` ，如果该路径存在。       
* 如果这个路径不存在，则使用一个没有文件操作的提供器
* 可使用 ``UseWebRoot`` 方法设置。

::

   new WebHostBuilder()
       .UseWebRoot("public")

使用  ``configuration`` 来设置宿主的配置值 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
使用  ``UseConfiguration`` 指定  宿主的配置项，可以将所有的配置项写入到 ``hosting.json`` 文件中

::

   public static void Main(string[] args)
   {
     var config = new ConfigurationBuilder()
       .AddCommandLine(args)
       .AddJsonFile("hosting.json", optional: true)
       .Build();
   
     var host = new WebHostBuilder()
       .UseConfiguration(config)
       .UseKestrel()
       .Configure(app =>
       {
         app.Run(async (context) => await context.Response.WriteAsync("Hi!"));
       })
     .Build();
   
     host.Run();
   }      

启动宿主
^^^^^^^^^^^^

1. ``Run`` 方法启动 Web 应用程序并且阻止调用线程，直到宿主关闭   

::

   host.Run();

2. 调用宿主的 ``Start`` 方法来以非阻塞方式运行宿主

::

   using (host)
   {
     host.Start();
     Console.ReadLine();
   }

宿主配置项排序的重要性
----------------------

可以通过指定配置来重写任何环境变量（使用 ``UseConfiguration`` ）或者明确地设置值   

* **宿主会使用任何选项最后设置的值**

::

   var config = new ConfigurationBuilder()
   .AddCommandLine(args)
   .Build();
   
   var host = new WebHostBuilder()
       .UseUrls("http://*:1000") // default URL
       .UseConfiguration(config) // override from command line
       .UseKestrel()
       .Build();
   