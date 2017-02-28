依赖注入
============

.. contents:: Sections:
   :local:
   :depth: 3

依赖注入的概念
---------------

控制反转(IoC)
^^^^^^^^^^^^^^^^^
什么是控制反转(IoC)?
""""""""""""""""""""""

**控制反转** 即 ``IoC`` (Inversion of Control),它把传统上由程序代码直接操控的对象的调用权交给容器(即 ``IoC容器``)，通过容器来实现对象组件的装配和管理。

所谓的 **控制反转** 概念就是 **对组件对象控制权的转移** ，从程序代码本身转移到了 **外部容器** 。

控制反转的结构图

.. image::  http://zhangjunhd.blog.51cto.com/attachment/200901/200901141231944972691.jpg
   :width: 600
   :height: 250

IoC的实现方式
"""""""""""""""""

1. 依赖查找<Dependency Lookup>
  
   容器提供回调接口和上下文环境给组件

2. 依赖注入<Dependency Injection>
   
   组件不做定位查询，只提供普通的方法让容器去决定依赖关系     
      * 注入的方式
           1. 接口注入（Interface Injection）          
           2. 设值注入（Setter Injection）           
           3. 构造子注入（Constructor Injection）三种方式 


依赖注入
^^^^^^^^^^^^^
**依赖注入** （Dependency injection，DI）是一种实现对象及其合作者或依赖项之间松散耦合的技术。将类用来执行其操作（Action）的这些对象以某种方式提供给该类，而不是直接实例化合作者或使用静态引用.

当类的设计使用 ``DI`` 思想，它们 **耦合更加松散** ，因为它们没有对它们的合作者直接硬编码的依赖;

容器
^^^^^^^^^^
当系统被设计使用 ``DI`` ，很多类通过它们的 **构造函数**（或 **属性** ）请求其依赖关系，有一个类被用来创建这些类及其相关的依赖关系是很有帮助的,这个类就被称之为 **容器**，或是 **IOC容器** ,或是 **DI容器**

容器 **本质** 上是一个工厂，负责提供向它请求的类型实例;如果一个给定类型声明它具有依赖关系，并且容器已经被配置为提供依赖类型，它将把创建依赖关系作为创建请求实例的一部分;通过这种方式，可以向类型提供复杂的依赖关系而不需要任何硬编码的类型构造。

容器的作用：
""""""""""""""
1. 创建对象的依赖关系
2. 管理应用程序中对象的生命周期

ASP.NET Core的内置容器
----------------------
ASP.NET Core 包含了一个默认支持构造函数注入的简单内置容器（由 ``IServiceProvider`` 接口表示），并且 ASP.NET 使某些服务可以通过 ``DI`` 获取。

ASP.NET 的 **容器** 指的是它管理的类型为 ``services``;在应用程序 ``Startup`` 类的 ``ConfigureServices`` 方法中配置内置容器的服务

使用框架提供的服务
^^^^^^^^^^^^^^^^^^^

``IServiceCollection`` 只向 ``ConfigureServices`` 提供了几个服务定义。

使用一些扩展方法（如 ``AddDbContext`` ，``AddIdentity`` 和 ``AddMvc`` ）向容器中添加额外服务;

注册服务
^^^^^^^^^^^^^

::

   services.AddTransient<IEmailSender, AuthMessageSender>();
   services.AddTransient<ISmsSender, AuthMessageSender>();


第一个泛型类型表示将要从容器中请求的类型（通常是一个接口）。第二个泛型类型表示将由容器实例化并且用于完成这些请求的具体类型

生命周期
"""""""""""""

**瞬时**
- 瞬时（Transient）生命周期服务在它们每次请求时被创建。这一生命周期 **适合轻量级的，无状态的服务** 。

**作用域**
- 作用域（Scoped）生命周期服务在每次请求被创建一次。

**单例**
- 单例（Singleton）生命周期服务在它们第一次被请求时创建（或者如果你在 ConfigureServices
运行时指定一个实例）并且每个后续请求将使用相同的实例。
- 如果在应用程序需要单例行为，建议让服务容器管理服务的生命周期而不是在自己的类中实现单例模式和管理对象的生命周期

请求服务
-------------
来自 ``HttpContext`` 的一次 ASP.NET 请求中可用的服务通过 ``RequestServices`` 集合公开的.

**请求服务** (``RequestServices``) 将你配置的服务和请求描述为应用程序的一部分。当你的对象指定依赖关系，这些满足要求的对象通过查找  ``RequestServices`` 中对应的类型得到，而不是 ``ApplicationServices``

替换默认的服务容器
-------------------

开发人员可以很容易地使用他们的首选容器替换默认容器;

``ConfigureServices`` 方法通常返回 ``void`` ，但是如果改变它的签名返回 ``IServiceProvider`` ，可以 **配置并返回一个不同的容器**

配置 Autofac 作为服务容器
^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. 在 ``project.json`` 的 ``dependencies`` 属性中添加适当的容器包

   ::
   
      "dependencies" : {
        "Autofac": "4.0.0-rc2-237",
        "Autofac.Extensions.DependencyInjection": "4.0.0-rc2-200"
      },
   
2. 在 ``ConfigureServices`` 中配置容器并返回 ``IServiceProvider``

   ::
   
       public IServiceProvider ConfigureServices(IServiceCollection services)
       {
         services.AddMvc();
         // add other framework services
       
         // Add Autofac
         var containerBuilder = new ContainerBuilder();
         containerBuilder.RegisterModule<DefaultModule>();
         containerBuilder.Populate(services);
         var container = containerBuilder.Build();
         return container.Resolve<IServiceProvider>();
       } 

3. 在 ``DefaultModule`` 中配置 ``Autofac``

   ::
   
      public class DefaultModule : Module
      {
        protected override void Load(ContainerBuilder builder)
        {
          builder.RegisterType<CharacterRepository>().As<ICharacterRepository>();
        }
      }

.. note::

   当使用第三方 ``DI`` 容器时，你必须更改 ``ConfigureServices`` 让它返回 ``IServiceProvider`` 而不是 ``void`` 。      


使用依赖注入的建议
-----------------------

- ``DI`` 针对具有复杂依赖关系的对象。控制器，服务，适配器和仓储都是可能被添加到 ``DI`` 的对象的例子。

- 避免直接在 DI 中存储数据和配置。例如，用户的购物车通常不应该被添加到服务容器中。配置应该使用 Options Model。 同样, 避免 “数据持有者” 对象只是为了允许访问其他对象而存在。如果可能的话，最好是通过 DI 获取实际的项。

- 避免静态访问服务。

- 避免在应用程序代码中服务定位。

- 避免静态访问 ``HttpContext`` 。   