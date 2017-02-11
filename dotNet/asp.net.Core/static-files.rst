静态资源
================

.. contents:: Sections:
   :local:
   :depth: 2


什么是静态资源?
--------------------
直接把相应文件发送到客户端的文件都是 **静态资源** .

诸如 ``HTML`` 、``CSS`` 、``图片`` 和 ``JavaScript`` 之类的资源被 ASP.NET Core 应用直接提供给客户端,就是静态资源.

静态文件服务
------------

**静态文件** 通常位于 ``web root`` （ ``<content-root>/wwwroot`` ）文件夹下.

.. note::
   通常会把项目的当前目录设置为 ``Content root`` ，这样项目的 ``web root`` 就可以在开发阶段被明确


::

   public static void Main(string[] args)
   {
       var host = new WebHostBuilder()
           .UseKestrel()
           .UseContentRoot(Directory.GetCurrentDirectory()) //手工高亮
           .UseIISIntegration()
           .UseStartup<Startup>()
           .Build();
   
       host.Run();
   }


静态文件能够被保存在网站根目录下的 **任意文件夹** 内，并通过相对根的路径来访问.

* ``http://<app>/images/<imageFileName>``
* ``http://localhost:9189/images/banner3.svg``

静态文件中间件
""""""""""""""""""

想要启用静态文件服务，就必须配置中间件（ ``middleware`` ），把静态文件中间件加入到管道内.
  
#. 增加 ``Microsoft.AspNetCore.StaticFiles`` 包依赖，
#. 然后从 ``Startup.Configure`` 调用 ``UseStaticFiles`` 扩展方法

::

   public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
   {
       app.UseStaticFiles(); //手工高亮
   }

.. note::
   ``web root`` 的默认目录是 ``wwwroot`` ，但你可以通过 ``UseWebRoot`` 来设置 web root   

如何配置静态文件的位于外部的 web root    
-----------------------------------------
项目结构如下：

* wwwroot
   * css
   * images
   * ...
* MyStaticFiles
   * test.png

对于访问 test.png 的请求，可以如此配置静态文件中间件、

::


   public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
   {
       app.UseStaticFiles();
   
       app.UseStaticFiles(new StaticFileOptions()                                      //手工高亮
       {                                                                               //手工高亮
           FileProvider = new PhysicalFileProvider(                                    //手工高亮
               Path.Combine(Directory.GetCurrentDirectory(), @"MyStaticFiles")),       //手工高亮
           RequestPath = new PathString("/StaticFiles")                                //手工高亮
       });                                                                             //手工高亮
   }

静态文件授权
---------------
静态文件模块并 **不** 提供授权检查。任何通过该模块提供访问的文件，包括位于 ``wwwroot`` 下的文件都是公开的   

为了给文件提供授权:

* 将文件保存在 ``wwwroot`` 之外并将目录设置为可被静态文件中间件访问到，同时——
* 通过一个控制器的 ``Action`` 来访问它们，通过授权后返回 ``FileResult``

允许直接浏览目录
-----------------

什么是目录浏览
"""""""""""""""""
**目录浏览** 允许网站用户看到指定目录下的目录和文件列表。

基于安全考虑，默认情况下是禁用目录访问功能的

怎样开启目录浏览
""""""""""""""""

#. 在 ``Startup.Configure`` 中调用 ``UseDirectoryBrowser`` 扩展方法可以开启网络应用目录浏览.

   ::
   
      public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory    loggerFactory)
      {
          app.UseStaticFiles(); // For the wwwroot folder
      
          app.UseStaticFiles(new StaticFileOptions()
          {
              FileProvider = new PhysicalFileProvider(
                  Path.Combine(Directory.GetCurrentDirectory(), @"wwwroot\images")),
              RequestPath = new PathString("/MyImages")
          });
      
          app.UseDirectoryBrowser(new DirectoryBrowserOptions()
          {
              FileProvider = new PhysicalFileProvider(
                  Path.Combine(Directory.GetCurrentDirectory(), @"wwwroot\images")),
              RequestPath = new PathString("/MyImages")
          });
      }

#. 并且通过从 ``Startup.ConfigureServices`` 调用 ``AddDirectoryBrowser`` 扩展方法来增加所需服务

   ::
   
      public void ConfigureServices(IServiceCollection services)
      {
          services.AddDirectoryBrowser();
      }

默认文档服务
----------------

设置默认首页能给你的站点的每个访问者提供一个起始页.

避免用户输入完整 URI，须在 ``Startup.Configure`` 中调用 ``UseDefaultFiles`` 扩展方法      

::

   public void Configure(IApplicationBuilder app)
   {
       app.UseDefaultFiles();                                                          //手工高亮
       app.UseStaticFiles();
   }

.. note::
   ``UseDefaultFiles`` 必须在 ``UseStaticFiles`` 之前调用。 ``UseDefaultFiles`` 只是重写了 URL，而不是真的提供了这样一个文件。你必须开启静态文件中间件（ ``UseStaticFiles`` ）来提供这个文件.

通过 ``UseDefaultFiles`` ，请求文件夹的时候将检索以下文件:
* default.htm
* default.html
* index.htm
* index.html

修改默认文件
""""""""""""""""
::

   public void Configure(IApplicationBuilder app)
   {
       // Serve my app-specific default file, if present. 
       DefaultFilesOptions options = new DefaultFilesOptions(); 
       options.DefaultFileNames.Clear(); 
       options.DefaultFileNames.Add("mydefault.html");
       app.UseDefaultFiles(options);
       app.UseStaticFiles();
   }

UseFileServer
-----------------

``seFileServer`` 包含了 ``UseStaticFiles`` 、 ``UseDefaultFiles``  和 ``UseDirectoryBrowser`` 的功能。

* 启用了静态文件和默认文件，但不允许直接访问目录

  ::
  
  app.UseFileServer();

* 启用了静态文件、默认文件和目录浏览功能

  ::
  
  app.UseFileServer(enableDirectoryBrowsing: true);

FileExtensionContentTypeProvider
""""""""""""""""""""""""""""""""""""

``FileExtensionContentTypeProvider`` 类内包含一个将文件扩展名映射到 MIME 内容类型的集合


非标准的内容类型
-----------------

ASP.NET 静态文件中间件能够支持超过 400 种已知文件内容类型。

如果用户请求一个未知的文件类型，静态文件中间件将返回 HTTP 404（未找到）响应。

如果启用目录浏览，该文件的链接将会被显示，但 URI 会返回一个 HTTP 404 错误。  


