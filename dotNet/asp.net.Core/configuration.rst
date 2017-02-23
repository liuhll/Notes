配置
===========

.. contents:: Sections:
   :local:
   :depth: 2


概述
----------
ASP.NET Core 支持多种配置选项。
应用程序配置数据内建支持读取 ``JSON`` 、``XML`` 和 ``INI`` 格式的配置文件和环境变量


获取和设置配置
---------------

Asp.net Core 对.net 平台的配置架构重新进行了设计(老版本的.net 平台的配置是依赖于 ``System.Configuration`` 和 ``XML`` 配置文件, 如 ``Web.config`` );

* 新的配置模型提供了 **精简高效** 的通过 **检索多样化** 提供程序的获取 **基于键/值对** 配置的能力
* 应用程序和框架可以通过新的 **选择模式** 访问配置

配置的选择
^^^^^^^^^^^^^^^^^^^
建议在应用程序的 ``Startup`` 类中 **只实例化一个** ``Configuration`` 实例。然后使用 **选择模式** 来访问各自的设置


配置的原理
^^^^^^^^^^^
``Configuration`` 类 是一个提供了读写名/值对能力的 ``Providers`` **集合** 。

**至少需要配置一个提供程序** ，使得 ``Configuration`` 能正常工作

* ``Configuration`` 实例是通过 ``ConfigurationBuilder`` 的 ``Builder()`` 方法来构建的，
* 并且至少要求提供一个 **配置提供程序**

::

   var builder = new ConfigurationBuilder();
   builder.AddInMemoryCollection();
   var config = builder.Build();
   config["somekey"] = "somevalue";
   
   // do some other work
   
   var setting = config["somekey"]; // also returns "somevalue"


.. note::
   必须至少设置一个配置提供程序


获取配置值
^^^^^^^^^^^^^
可以使用以 ``:`` 符号分隔（从层次结构的根开始）的键来取回配置值

例如:

    应用程序使用 ``configuration`` 配置正确的连接字符串。    
    
    可以通过键 ``Data:DefaultConnection:ConnectionString`` 来访问 ``ConnectionString`` 的设置。

选择模式
^^^^^^^^^^^^^^^
应用程序所需要的设置和指定配置的机制（configuration 便是一例）都可通过使用 **选择模式** 解耦。

1. 创建自己的配置类（可以是几个不同的类，分别对应不同的配置组）

2. 而后通过选项服务注入到应用程序中。然后就可以通过配置或其它你所选择的机制来设置了。

內建的配置提供程序
---------------------

配置框架已内建支持 ``JSON`` 、``XML`` 和 ``INI`` 配置文件， **内存配置** （直接通过代码设置值），从 **环境变量和命令行参数** 中拉取配置
可以 **把多个配置提供程序组合在一起** ，就像是用从当前存在的另一个配置提供程序中获取配置值 **覆盖默认配置** 一样

* 扩展方法支持为配置添加额外的配置文件提供程序。这些方法能被独立的或链式的（如 fluent API）调用在 ``ConfigurationBuilder`` 实例之上

    ::
    
       // work with with a builder using multiple calls
       var builder = new ConfigurationBuilder();
       builder.SetBasePath(Directory.GetCurrentDirectory());
       builder.AddJsonFile("appsettings.json");
       var connectionStringConfig = builder.Build();
       
       // chain calls together as a fluent API
       var config = new ConfigurationBuilder()
           .SetBasePath(Directory.GetCurrentDirectory())
           .AddJsonFile("appsettings.json")
           .AddEntityFrameworkConfig(options =>
               options.UseSqlServer(connectionStringConfig.GetConnectionString("DefaultConnection"))
           )
           .Build();


* 指定配置提供程序的顺序非常重要，这将影响它们的设置 **被应用的优先级** （如果存在于多个位置）       
    * **最后指定的配置** 提供程序将“获胜”（如果该设置存在于至少两处位置）

* ASP.NET 团队建议最后指定环境变量，如此一来本地环境可以覆盖任何部署在配置文件中的设置。

* 对于指定环境的配置文件非常有用
    
    ::
    
       public Startup(IHostingEnvironment env)
       {
           var builder = new ConfigurationBuilder()
               .SetBasePath(env.ContentRootPath)
               .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
               .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true); //手动高亮
       
           if (env.IsDevelopment())
           {
               // For more details on using the user secret store see http://go.microsoft.com/fwlink/?LinkID=532709
               builder.AddUserSecrets();
           }
       
           builder.AddEnvironmentVariables();
           Configuration = builder.Build();
       }    


.. note::
   ``IHostingEnvironment`` 服务用于获取当前环境。
   
   在 ``Development`` 环境中，上例高亮行代码将寻找名为 ``appsettings.Development.json`` 的配置文件，并用其中的值覆盖当前存在的其它值       