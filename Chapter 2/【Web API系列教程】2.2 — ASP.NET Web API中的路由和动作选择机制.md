这篇文章描述了ASP.NET Web API如何将HTTP请求路由到控制器上的特定动作。

备注：想要了解关于路由的高层次概述，请查看Routing in ASP.NET Web API。

这篇文章侧重于路由过程的细节。如果你创建了一个Web API项目并且发现一些请求并没有按你预期得到相应的路由，希望这篇文章有所帮助。

路由有以下三个主要阶段：

1.	将URI匹配到路由模板
2.	选择一个控制器
3.	选择一个动作

你可以用自己的习惯行为来替换其中一些过程。在本文中，我会描述默认行为。在结尾，我会指出你可以自定义行为的地方。

**路由模板（Route Templates）**
---------------------

路由模板看起来和URI路径非常相似，但是它能包含用大括号指明的占位符。

```
"api/{controller}/public/{category}/{id}"
```

当你创建了一个路由，你为一些或全部占位符提供默认的值：

```
defaults: new { category = "all" }
```

你也可以提供一些约束（constraints），它限制了URI字段如何才能匹配一个占位符：

```
constraints: new { id = @"\d+" }   // Only matches if "id" is one or more digits.
```

框架会尽力将URI路径中的字段匹配到模板中。模板中的文字必须准确匹配。一个占位符可以匹配多个变量，除非你指定了约束。框架不会匹配URI的其他部分，比如主机名或查询参数。框架仅仅在用于匹配URI的路由表中选择第一个路由。

这里有两个特殊的占位符：”{controller}“和“{action}”。

- “{controller}“提供了控制器的名称。
- “{action}“提供了动作的名称。在Web API中，通过会忽略“{action}”。

**Defaults**
--------

如果你提供了默认的API，路由将会匹配缺少这些的URI。例如：

```
routes.MapHttpRoute(
    name: "DefaultApi",
    routeTemplate: "api/{controller}/{category}",
    defaults: new { category = "all" }
);

```
对于URI *http: //localhost/api/products* 将会匹配这个路由。*{category}* 字段会被分配默认值 *all*。

**路由字典（Route Dictionary）**
----------------------

如果框架发现了URI的一个匹配，它会创建一个包含了每个占位符适用的值的字典集合。键是不包含大括号的占位符名称。值是提取自URI路径或者默认表单。该字典被存储在IHttpRouteData对象中。

在路由匹配阶段，“{controller}“和”{action}“占位符会被像其他占位符一样对待。它们被同其他值一起简单地存储在字典中。

对于defaults，它可以有一个特殊值RouteParameter.Optional。如果一个占位符被分配到这个值，那么这个值不会被添加到路由字典中。例如：


```
routes.MapHttpRoute(
    name: "DefaultApi",
    routeTemplate: "api/{controller}/{category}/{id}",
    defaults: new { category = "all", id = RouteParameter.Optional }
);

```

对于URI路径“api/products”，路由字典将会包含：

1.	controller：“products”
2.	category：“all”

然而对于“api/products/toys/123”，路由字典将会包含：

1.	controller：“products”
2.	category：“all”
3. id：“123“


对于defaults，它同样也会包含一个没有在路由模板中任何地方出现的值。如果路由匹配了，这个值会被存储在字典中。例如：

```
routes.MapHttpRoute(
    name: "Root",
    routeTemplate: "api/root/{id}",
    defaults: new { controller = "customers", id = RouteParameter.Optional }
);

```

如果URI路径“api/root/8”，字典将会包含两个值：

- controller:”customers“
- id：“8”

**选择控制器（Selecting a Controller）**
-----------------------------

控制器的选择由IHttpControllerSelector.SelectController方法来处理。这个方法需要传入一个HttpRequestMessage实例并返回HttpControllerDescriptor对象。默认的实现是由DefaultHttpControllerSelector类来实现的。这个类使用了一个简单的算法：

1.	在路由字典中查找键”controller“。
2.	提取出这个键对应的值，并添加字符串“Controller”以得到控制器的类型名
3.	用这个类型名来查找一个Web API控制器

例如，如果路由字典包含键值对“controller”=“products”，那么控制器类型就是“ProductsController”。如果这里不存在匹配的类型，或存在多个匹配，那么框架就会向客户端发送一个错误。

对于步骤3，DefaultHttpControllerSelector会使用IHttpControllerTypeResolver接口来得到Web API控制器类型的列表。IHttpControllerTypeResolver的默认实现会返回（a）实现IHttpController，（b）不是抽象的，（c）名称以“Controller“结尾的所有公共的类。

**动作选择**
----


在选择控制器之后，框架会通过调用IHttpActionSelector.SelectAction方法来选择动作。这个方法需要传入一个HttpControllerContext参数以及返回一个HttpActionDescriptor对象。

默认的实现由ApiControllerActionSelector类来提供。为了选择一个动作，它会按以下要求来查找：
1）	请求的HTTP方法
2）	路由模板中的“{action}“占位符（如果存在）
3）	控制器中动作的参数

在查看选择算法之前，我们需要理解关于控制器动作的一些东西。

**控制器中的哪些方法会被认为是“动作“？**当选择一个动作时，框架仅仅在控制器中查找公共的实例方法。当然了，它会排除一些”特殊“的方法（构造函数，事件，操作重载等等）和继承自ApiController类的方法。

**HTTP方法。**框架只会选择匹配请求的HTTP方法的动作，它取决于以下几点：

1.	你可以用某个属性来具体说明是HTTP方法：AcceptVerbs，HttpDelete，HttpGet，HttpHead，HttpOptions，HttpPatch，HttpPost或HttpPut。
2.	或者，如果一个控制器方法的名称以”Get”，“Post“，”Put“，”Delete“，”Head“，”Options“或”Patch“开始，那么按照约定该动作就支持HTTP方法。
3.	如果不包含以上几点，但支持POST的方法。

**参数绑定。**参数绑定是指Web API如何如何为参数创建一个值。这里是参数绑定的默认规则：

1.	简单类型直接从URI中提取
2.	复杂类型从请求体重提取

简单类型包括所有.NET框架基本类型（.NET Framework primitive types），再加上DateTime、Decimal、Guid、String和TimeSpan。对于每个动作，最多有一个参数可以读取请求体。

备注：重载默认绑定规则也是有可能的。查看WebAPI Parameter binding under the hood.

有了以上这些背景知识，这里是动作选择的算法：

1.	基于HTTP请求方法匹配到的控制器创建一个动作列表。
2.	动作路由字典包含“action“记录，移除其名字不匹配该值的动作。
3.	根据如下规则，尽力将动作参数匹配到URI：

- a 对于每个动作，当绑定从URI中获得参数时得到一个简单类型的参数列表。执行可选的参数。
- b 从这个列表中，无论是在路由字典中还是URI查询字符串中，都尽力找出针对每个参数名称的匹配。匹配不区分大小写并且不取决于参数顺序。
- c 当列表中的每个参数在URI中都有一个匹配时，选择一个动作。
- d 如果多个动作符合这些标准，那么选择其中一个有最多参数匹配的。

4.	忽略包含[NonAction]属性的动作。

步骤3可能是最容易迷惑的。基本的思想是参数可以从URI、请求体或绑定中获得它的值。对于来自URI的参数，我们会确保URI确实包含一个给参数的值，不论是在路径（通过路由字典）还是在查询字符串中。

例如，考虑如下动作：

```
public void Get(int id)
```

这个id参数绑定到URI上，因此，这个动作可以匹配到包含一个给“id“的值的URI，不论是在路由字典还是查询字符串中。

可选参数是个例外，因为它们是可选的。对于可选参数，如果这个绑定不了从URI中得到这个值也是没关系的。

因为一些不同的原因，复杂类型也是个例外。复杂类型只能通过自定义绑定来绑定到URI上。但是在这种情况下，框架无法事先知道参数可能被绑定到一个特殊的URI。为了弄清楚它，就需要去执行这个绑定。这个选择算法的目标是在执行任何绑定之前，从静态描述中去选择一个动作。因此，复杂类型会从这种匹配算法中执行。

在动作被选取好了，所有的参数绑定也就被执行了。

总结：

1.	动作必须匹配请求的HTTP方法。
2.	动作名（如果存在）必须匹配路由字典中的“action“词条
3.	对于动作的所有参数，如果参数提取自URI，那么参数名必须在路由字典或URI查询字符串中被找到。(可选参数和复杂类型的参数除外。)
4.	尽量去匹配最多的参数数目。但最好的匹配也可能是不包含任何参数的方法。


**扩展示例（Extended Example）**
----------------------


路由：

```
routes.MapHttpRoute(
    name: "ApiRoot",
    routeTemplate: "api/root/{id}",
    defaults: new { controller = "products", id = RouteParameter.Optional }
);
routes.MapHttpRoute(
    name: "DefaultApi",
    routeTemplate: "api/{controller}/{id}",
    defaults: new { id = RouteParameter.Optional }
);

```


控制器：

```
public class ProductsController : ApiController
{
    public IEnumerable<Product> GetAll() {}
    public Product GetById(int id, double version = 1.0) {}
    [HttpGet]
    public void FindProductsByName(string name) {}
    public void Post(Product value) {}
    public void Put(int id, Product value) {}
}

```

HTTP请求：

```
GET http://localhost:34701/api/products/1?version=1.5&details=1
```

**路由匹配（Route Matching）**
--------------------

该URI会匹配到名为”DefaultApi”的路由。这个路由字典包含以下词条：

- controller：“products”
- id：“1“

这个路由字典不包含查询字符串“version”和“details”，但是在动作选择的时候这些仍然会被考虑。




**控制器选择（Controller Selection）**
---------------------------

根据路由字典中的“controller”词条，控制器类型是ProductsController。

**动作选择（Action Selection）**
----------------------

该HTTP请求是一个GET请求。相应的支持GET的控制器动作是GetAll、GetById和FindProductsByName。路由字典中不包含任何“action“词条，所以我们不用去匹配动作名称。

接下来，我们尝试着匹配动作的参数名称，现在仅在GET动作中查找。

Action|	Parameters to Match
-|-
GetAll|	none
GetById	|"id"
FindProductsByName|	"name"


注意到GetById的version参数没有被考虑，因为它是一个可选参数。

显而易见GetAll方法能够匹配，GetById方法也能匹配，因为路由字典中包含“id“。FindProductsByName方法不匹配。

最后是GetById方法获胜，因为它能够匹配到一个参数，相对应的是没有参数能匹配GetAll。该方法伴随以下参数的值来执行：

- id = 1
- version = 1.5

注意到尽管version参数没有在选择算法中使用，但该参数的值也依旧是来自URI的查询字符串中。

**扩展点（Extension Points）**
---------------------
Web API为路由过程的一些部分提供了扩展点。



Interface|	Description
-|-
IHttpControllerSelector|	Selects the controller.
IHttpControllerTypeResolver	|Gets the list of controller types. The DefaultHttpControllerSelector chooses the controller type from this list.
IAssembliesResolver|	Gets the list of project assemblies. The IHttpControllerTypeResolverinterface uses this list to find the controller types.
IHttpControllerActivator	|Creates new controller instances.
IHttpActionSelector|	Selects the action.
IHttpActionInvoker|	Invokes the action.


为任何这些接口提供自己的实现，请使用HttpConfiguration对象上的Services集合：


```
var config = GlobalConfiguration.Configuration;
config.Services.Replace(typeof(IHttpControllerSelector), new MyControllerSelector(config));

```










