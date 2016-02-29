这篇文章描述了ASP.NET Web API如何将HTTP请求发送（路由）到控制器。

备注：如果你对ASP.NET MVC很熟悉，你会发现Web API路由和MVC路由非常相似。主要区别是Web API使用HTTP方法来选择动作（action），而不是URI路径。你也可以在Web API中使用MVC风格的路由。这篇文章不需要ASP.NET MVC的任何知识。

**路由表**
---

在ASP.NET Web API中，控制器是一个用于处理HTTP请求的类。控制器中的公共方法被称为动作方法或简单动作。当Web API框架收到请求时，它会将该请求路由到相应的动作中。

为了确定哪个动作该被执行，框架就会使用本节将讲解的路由表。Visual Studio的Web API项目模板就创建了一个默认的路由表：


```
routes.MapHttpRoute(
    name: "API Default",
    routeTemplate: "api/{controller}/{id}",
    defaults: new { id = RouteParameter.Optional }
);

```
这个路由被定义在App_Start目录下的WepApiConfig.cs文件中。

![这里写图片描述](http://img.blog.csdn.net/20160227093017930)

路由表中的每条记录都包含了一个路由模板。Web API的默认路由模板是“api/{controller}/{id}”。在这个模板中，”api”是一个字面路径字段，而{controller}和{id}都是占位符变量。

当Web API框架收到了HTTP请求时，它将会尽力匹配URI到路由表中的路由模板的其中一个。如果没有路由被匹配到，客户端就会收到404错误。例如，以下URI会匹配到默认路由：

1. /api/contacts
2. /api/contacts/1
3. /api/products/gizmo1

然而，以下URI不会匹配到，因为它缺乏“api”字段。

```
/contacts/1
```

备注：在路由中使用“api”的原因是为了避免和ASP.NET MVC的路由冲突。也就是说，你可以使用”/contacts”匹配到MVC的路由，使用“api/contacts”匹配到Web API的路由。当然了，如果你不喜欢这种约定，你也可以修改默认路由表。

一旦某个路由匹配到了，Web API就会选择相应的控制器及动作：

1. 为了找到控制器，Web API将“Controller”添加到{controller}变量上。
2.	为了找到动作，Web API会遍历HTTP方法，然后查找一个其名字以HTTP方法的名字开头的动作。例如，有一个GET请求，Web API会查找以“Get....”开头的动作，比如”GetContact”或”GetAllContacts”。这种方式仅仅适用于GET、POST、PUT和DELETE方法。你可以通过在你的控制器中使用属性来启用其他HTTP方法。将晚些看到一个示例（超链接到本章的第三节……
3.	路由模板的其他占位符变量，比如｛id｝，会被映射到动作的参数。

让我们来看一个示例。假定你定义了如下的控制器：

```
public class ProductsController : ApiController
{
    public void GetAllProducts() { }
    public IEnumerable<Product> GetProductById(int id) { }
    public HttpResponseMessage DeleteProduct(int id){ }
}

```
这里是一些可能的HTTP请求，以及相应的得到执行的动作：


HTTP Method	|URI Path|	Action|	Parameter
-|-|-|-
GET	|api/products	|GetAllProducts|	(none)
GET|	api/products/4	|GetProductById|	4
DELETE|	api/products/4	|DeleteProduct	|4
POST	|api/products	|(no match)	 |


注意URI的{id}字段，如果存在，它会被映射到动作的id参数中。在本例，控制器定义了两个GET方法，其中一个包含id参数，而另一个不包含id参数。

同样的，注意到POST请求会失败，因为控制器中并没有定义”POST...”方法。

**路由偏差（Routing Variations）**
------------------------

前一节描述了ASP.NET Web API的基本路由机制。本节将开始描述一些变化。


**HTTP方法**

除了使用这些HTTP方法的命名约定，你也可以通过用HttpGet、HttpPut、HttpPost或HttpDelete属性来赋予这些动作来具体地为每个动作设定HTTP方法。

在下面这个例子中，FindProduct方法被映射到GET请求：

```
public class ProductsController : ApiController
{
    [HttpGet]
    public Product FindProduct(id) {}
}

```
为了让一个动作支持多个HTTP方法，或支持除GET、PUT、POST和DELETE之外的HTTP方法，你可以使用AcceptVerbs属性，它以一个HTTP方法列表为参数。

```
public class ProductsController : ApiController
{
    [AcceptVerbs("GET", "HEAD")]
    public Product FindProduct(id) { }

    // WebDAV method
    [AcceptVerbs("MKCOL")]
    public void MakeCollection() { }
}

```

**通过动作名进行路由**
---------

有了默认的路由模板，Web API使用HTTP方法来选择动作。然而，你也可以创建一个将动作名包含在URI中的路由表。

```
routes.MapHttpRoute(
    name: "ActionApi",
    routeTemplate: "api/{controller}/{action}/{id}",
    defaults: new { id = RouteParameter.Optional }
);

```

在这个路由模板中，{action}参数在控制器中命名了一个动作方法。在这种风格的路由中，应使用属性来指定允许的HTTP方法。例如，假定你的控制器有了以下方法：

```
public class ProductsController : ApiController
{
    [HttpGet]
    public string Details(int id);
}

```
在这种情况下，对于“api/products/details/1”的GET请求被被映射到Details方法。这种风格的路由和ASP.NET MVC很接近，并且可能适合于RPC风格的API。

你可以通过ActionName属性来重写动作名。在接下来的例子中，存在两个都映射到”api/products/thumbnail/id”的动作。其中一个支持GET，另一个支持POST：

```
public class ProductsController : ApiController
{
    [HttpGet]
    [ActionName("Thumbnail")]
    public HttpResponseMessage GetThumbnailImage(int id);

    [HttpPost]
    [ActionName("Thumbnail")]
    public void AddThumbnailImage(int id);
}

```

**无动作（Non-Actions）**
----------------

为了阻止一个方法被当作动作来执行，可以使用NonAction属性。这会框架指明该方法并非一个动作，即使是它可能匹配到路由规则。

```
// Not an action method.
[NonAction]  
public string GetPrivateData() { ... }

```








































