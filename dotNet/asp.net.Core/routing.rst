路由
===========

.. contents:: Sections:
   :local:
   :depth: 3

路由的概念
----------------

什么是路由？
""""""""""""""""
路由是用来把请求映射到 **路由处理程序** 。应用程序一启动就配置了路由，并且可以从URL中提取值用于处理请求。它还负责使用 ASP.NET 应用程序中定义的路由来生成链接。

一个应用会有单个 **路由集合** 。这个集合会按顺序处理。请求会在这个路由集合里按照 *URL matching* 来查找匹配。响应使用路由生成 URLs


路由的作用
"""""""""""""
路由使用 ``routes`` 类 ( ``IRouter`` 的实现) 做到:

* 映射传入的请求到 **路由处理程序**
* 生成响应中使用的 URLs

URL 匹配
-----------

什么是路由匹配
"""""""""""""""
**路由匹配** 指的是路由调度请求到一个处理程序的过程.

这个过程通常是 **基于URL路径中的数据** ，但 **可以扩展到请求中的任何数据** .

调度请求到不同处理程序的能力是应用调节自身大小和复杂度的关键

如何进行路由匹配？
""""""""""""""""""""""
请求调用序列中每个路由的 **异步方法** 来进入路由中间件。 ``IRouter`` 实例通过设置 ``RouteContext Handler`` 为一个不为空的 ``RequestDelegate`` 来选择是否处理请求。

如果一个处理程序已经设置了路由，那么它就将被调用来处理这个请求，并且不会有其他的路由再去处理。如果所有的路由都执行了，请求还没有找到处理程序，那么中间件就会调用 ``next`` 方法，从而下一个在请求管道中的中间件就被调用了.


RouteAsync
"""""""""""""""
``RouteAsync`` 的主要输入是和当前请求关联的 ``RouteContext HttpContext`` 。在一个成功匹配之后， ``RouteContext.Handler`` 和   ``RouteContext RouteData`` 会作为输出.
                                      
在 ``RouteAsync`` 执行期间，一个成功匹配会基于已经完成的请求处理设置  ``RouteContext.RouteData`` 的属性为合适的值。当一个路由成功匹配了一个请求时， ``RouteContext.RouteData`` 包含了重要的关于匹配结果的状态信息.

RouteData Values
""""""""""""""""""
是一个从路由产生的 *路由值* 字典.

* 这些值通常 **由标记化的 URL 确定的** ，可以用来接收用户输入，或者用来在应用内部做更深层的调度决定.

RouteData DataTokens 
""""""""""""""""""""""
是相关的匹配路由 **附加数据的属性包** .

提供 ``数据令牌`` 支持与每个路由相关的状态数据，这样应用基于匹配的路由 **可以迟点做出决定** .

* 这些数据是开发人员定义的，不会影响路由的行为。
* 而且，数据令牌中的值可以是任何类型，对比路由值，它可以很容易的转成字符串

RouteData Routers 
"""""""""""""""""""""""
是一个成功匹配请求的路由列表.

* 路由可以彼此嵌套，而且 ``Routers`` 属性反映了请求通过路由逻辑树导致匹配的路径
* 一般来说， ``Routers`` 中的 **第一项** 就是一个路由集合,而且应该用来生成URL。 
* ``Routers`` 中的 **最后一项** 就是已匹配路由。

URL 生成
----------------

**Url生成** 指的路由基于一系列的路由值创建一个URL路径的过程.

* 这允许你的处理程序和能访问它们的URL直接有一个逻辑分离

Url生成的过程
""""""""""""""
路由生成遵循一个类似的迭代过程，但 **开始于** 用户或框架代码调用到路由集合的 ``GetVirtualPath`` 方法时.

每个路由的 ``GetVirtualPath`` 方法都会被调用，直到返回一个 **不为空** 的 ``VirtualPathData``.

``GetVirtualPath`` 的输入:  
^^^^^^^^^^^^^^^^^^^^^^^^^^^

* VirtualPathContext HttpContext
    * 提供 ``HttpContext`` 是因为路由可能需要获取服务或当前上下文相关的数据
* VirtualPathContext Values
    * 是用于指定如何生成当前操作所需的URL的路由值
* VirtualPathContext AmbientValues
    * 是随着路由系统匹配当前请求而产生的一系列路由值

.. note::
路由主要使用 ``Values`` 和 ``AmbientValues`` 提供的路由值来决定在哪儿生成一个 URL 以及包含什么值。

GetVirtualPath 的输出:
^^^^^^^^^^^^^^^^^^^^^^^^^^

是一个 ``VirtualPathDath`` ；

``VirtualPathData`` 是一个并行的 ``RouteData`` ；它包含了输出 URL 的虚拟路径以及应该由路由设置的一些额外的属性.

VirtualPathData VirtualPath 属性：
* 包含了路由生成的虚拟路径，根据需求，可能需要进一步处理；

VirtualPathData Router ：
* 是一个成功生成URL路由的参考。

VirtualPathData DataTokens 属性：

是一个关联到生成URL的路由的附加数据的字典集合，这个和 RouteData.DataTokens 是并行的。

创建路由
--------------
路由提供了 ``Route`` 类作为 ``IRouter`` 的标准实现。

* 当调用 ``RouteAsync`` 方法时， ``Route`` 使用 **路由模板** 语法定义匹配URL路径的模式。
* 当调用 ``GetVirtualPath`` 方法时， ``Route`` 会使用相同的路由模板生成 URL

如何创建路由?
""""""""""""""""

大多数的应用会通过调用 ``MapRoute`` 方法或者定义在 ``IRouteBuilder`` 接口上的一个类似的扩展方法来创建路由。所有的这些方法会创建一个 ``Route`` 实例并添加到路由集合中.

例子：

::

   routes.MapRoute(
       name: "default",
       template: "{controller=Home}/{action=Index}/{id?}");

增加路由约束和数据令牌的示例:

::

   routes.MapRoute(
       name: "us_english_products",
       template: "en-US/Products/{id}",
       defaults: new { controller = "Products", action = "Details" },
       constraints: new { id = new IntRouteConstraint() },
       dataTokens: new { locale = "en-US" });

使用路由中间件
-----------------

1. 在使用路由前，需要将其添加到 ``project.json`` 的 **依赖性** 中       

   ::
   
       "Microsoft.AspNetCore.Routing": <current version>
   
2. 在 ``Startup.cs`` 中添加路由到服务容器

   ::
   
      public void ConfigureServices(IServiceCollection services)
      {
          services.AddRouting();
      }
   
3. 路由必须在 ``Startup`` 类的 ``Configure`` 方法中配置   

   ::
   
      public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
      {
          var trackPackageRouteHandler = new RouteHandler(context =>
          {
              var routeValues = context.GetRouteData().Values;
              return context.Response.WriteAsync(
                  $"Hello! Route values: {string.Join(", ", routeValues)}");
          });
      
          var routeBuilder = new RouteBuilder(app, trackPackageRouteHandler);
      
          routeBuilder.MapRoute(
              "Track Package Route",
              "package/{operation:regex(^track|create|detonate$)}/{id:int}");
      
          routeBuilder.MapGet("hello/{name}", context =>
          {
              var name = context.GetRouteValue("name");
              // This is the route handler when HTTP GET "hello/<anything>"  matches
              // To match HTTP GET "hello/<anything>/<anything>, 
              // use routeBuilder.MapGet("hello/{*name}"
              return context.Response.WriteAsync($"Hi, {name}!");
          });            
      
          var routes = routeBuilder.Build();
          app.UseRouter(routes);
      }
   

框架提供了一系列的创建路由的扩展方法:

* MapRoute
* MapGet
* MapPost
* MapPut
* MapDelete
* MapVerb

请求委托 RequestDelegate
""""""""""""""""""""""""""""""
在路由匹配的时候将被用做 **路由处理程序**

路由模板参考
------------------

令牌内的大括号( ``{ }``  )定义了路由值 **参数的边界**

* 使用 ``*`` 号作为一个路由参数的前缀去绑定其余的 URI - 这被叫做 **全捕获参数**
* 路由参数可以有默认值，定义的方式是在参数名称后定义默认值，用 ``=`` 号分开
* 路由参数也可以有约束，必须匹配从URL绑定的路由值
   * 通过在路由参数名后增加一个冒号 ``:`` 和约束名来定义一个 **内联约束**

路由约束
---------------
当一个路由已经匹配到了传入的URL并标记了URL中的路由值时路由约束就会执行。它一般会检查和路由模板相关的路由的值而且会做出一个关于这个值是否是可接受的简单决定。有些路由约束使用路由值之外的数据来决定该请求是否可以进行路由。

下表展示了一些路由约束和他们的预期行为。

.. list-table:: Inline Route Constraints
  :header-rows: 1

  * - constraint
    - Example
    - Example Match
    - Notes
  * - ``int``
    - {id:int}
    - 123
    - Matches any integer
  * - ``bool``
    - {active:bool}
    - true
    - Matches ``true`` or ``false``
  * - ``datetime``
    - {dob:datetime}
    - 2016-01-01
    - Matches a valid ``DateTime`` value (in the invariant culture - see `options <http://msdn.microsoft.com/en-us/library/aszyst2c(v=vs.110).aspx>`_)
  * - ``decimal``
    - {price:decimal}
    - 49.99
    - Matches a valid ``decimal`` value
  * - ``double``
    - {weight:double}
    - 4.234
    - Matches a valid ``double`` value
  * - ``float``
    - {weight:float}
    - 3.14
    - Matches a valid ``float`` value
  * - ``guid``
    - {id:guid}
    - 7342570B-<snip>
    - Matches a valid ``Guid`` value
  * - ``long``
    - {ticks:long}
    - 123456789
    - Matches a valid ``long`` value
  * - ``minlength(value)``
    - {username:minlength(5)}
    - steve
    - String must be at least 5 characters long.
  * - ``maxlength(value)``
    - {filename:maxlength(8)}
    - somefile
    - String must be no more than 8 characters long.
  * - ``length(min,max)``
    - {filename:length(4,16)}
    - Somefile.txt
    - String must be at least 8 and no more than 16 characters long.
  * - ``min(value)``
    - {age:min(18)}
    - 19
    - Value must be at least 18.
  * - ``max(value)``
    - {age:max(120)}
    - 91
    - Value must be no more than 120.
  * - ``range(min,max)``
    - {age:range(18,120)}
    - 91
    - Value must be at least 18 but no more than 120.
  * - ``alpha``
    - {name:alpha}
    - Steve
    - String must consist of alphabetical characters.
  * - ``regex(expression)``
    - {ssn:regex(^\d{3}-\d{2}-\d{4}$)}
    - 123-45-6789
    - String must match the provided regular expression.
  * - ``required``
    - {name:required}
    - Steve
    - Used to enforce that a non-parameter value is present during URL generation.



  * - 约束
    - 示例
    - 匹配示例
    - 注释
  * - ``int``
    - {id:int}
    - 123
    - 匹配所有整型
  * - ``bool``
    - {active:bool}
    - true
    - 匹配 ``true`` 或 ``false``
  * - ``datetime``
    - {dob:datetime}
    - 2016-01-01
    - 匹配一个合法的 ``DateTime`` 值 (固定区域性 - 请看 `options <http://msdn.microsoft.com/en-us/library/aszyst2c(v=vs.110).aspx>`_)
  * - ``decimal``
    - {price:decimal}
    - 49.99
    - 匹配一个合法的 ``decimal`` 值
  * - ``double``
    - {weight:double}
    - 4.234
    - 匹配一个合法的 ``double`` 值
  * - ``float``
    - {weight:float}
    - 3.14
    - 匹配一个合法的 ``float`` 值
  * - ``guid``
    - {id:guid}
    - 7342570B-<snip>
    - 匹配一个合法的 ``Guid`` 值
  * - ``long``
    - {ticks:long}
    - 123456789
    - 匹配一个合法的 ``long`` 值
  * - ``minlength(value)``
    - {username:minlength(5)}
    - steve
    - 至少5个字符串长.
  * - ``maxlength(value)``
    - {filename:maxlength(8)}
    - somefile
    - 字符串不能超过8个字符长.
  * - ``length(min,max)``
    - {filename:length(4,16)}
    - Somefile.txt
    - 字符串至少8个长度且不超过16个字符长度.
  * - ``min(value)``
    - {age:min(18)}
    - 19
    - 值至少是18.
  * - ``max(value)``
    - {age:max(120)}
    - 91
    - 值不能超过120.
  * - ``range(min,max)``
    - {age:range(18,120)}
    - 91
    - 值必须介于18和120之间.
  * - ``alpha``
    - {name:alpha}
    - Steve
    - 字符串必须是由字母字符组成.
  * - ``regex(expression)``
    - {ssn:regex(\d{3}-\d{2}-\d{4})}
    - 123-45-6789
    - 字符串必须匹配提供的正则表达式.
  * - ``required``
    - {name:required}
    - Steve
    - 用于在URL生成时强制必须存在值.   

URL生成参考
--------------
