**前言**
--

路由是指Web API如何匹配到具体的动作。Web API 2支持一个新的路由类型，它被称为属性路由。正如其名，属性路由使用属性来定义路由。属性路由给予你在web API的URI上的更多控制。例如，你能轻易的创建用于描述层级资源的URI。

早期的路由风格被称为基于约定的路由，现在仍然被完整支持，你可以将这两种技术用于同一个项目中。

本主题演示如何启用属性的路由，并描述属性路由的各种选项。关于使用属性路由的实战教程，请查看Create a REST API with Attribute Routing in Web API 2。

•	Why Attribute Routing?
•	Enabling Attribute Routing
•	Adding Route Attributes
•	Route Prefixes
•	Route Constraints
•	Optional URI Parameters and Default Values
•	Route Names
•	Route Order

**前提条件（Prerequisites）**
-------------------

Visual Studio 2013 或 Visual Studio Express 2013

或者，使用NuGet Package Manager来安装必要的包。在Visual Studio的Tools目录下，选择Library Package Manager，然后选择Package Manager Console。在Package Manager Console窗口输入以下命令：

```
Install-Package Microsoft.AspNet.WebApi.WebHost
```

**Why Attribute Routing?**
----------------------

Web API的首个发行版使用基于约定的路由。在那种路由中，你定义一个或多个路由模板，它们是一些基本的参数字符串。当框架收到一个请求时，它会将URI匹配到路由模板中。（关于基于约定的路由的更多信息，请查看Routing in ASP.NET Web API）

基于约定的路由的一个优势是模板是定义在单一地方的，并且路由规则会被应用到所有的控制器。不幸的是，基于约定的路由很难去支持一个在RESTful API中很常见的URI模式。例如，资源通常包含着子资源：客户包含着订单，电影包含着演员，书籍包含着作者等等。所以很自然地创建映射这些关系的URI：
/customers/1/orders
有了属性路由，就可以很轻易地定义一个针对该URI的路由。你只需要简单的添加一个属性到控制器动作上：

```
[Route("customers/{customerId}/orders")]
public IEnumerable<Order> GetOrdersByCustomer(int customerId) { ... }

```

这里还有些因为有了属性路由而变得更加容易的其他模式：


**API versioning**

在本例中，”api/v1/products”相对于”api/v2/products”可能会路由到不同的控制器。
/api/v1/products
/api/v2/products


**Overloaded URI segments**

在本例中，”1”是个订单数字，但是“pending”映射到一个集合。
/orders/1
/orders/pending

**Multiple parameter types**

在本例中，“1”是个订单数字，但是“2013/06/10”却是个日期。
/orders/1
/orders/2013/06/10

**启用属性路由（Enabling Attribute Routing）**
----------------------------------

为了启用属性路由，需要在配置时调用MapHttpAttributeRoutes。这个扩展方法被定义在System.Web.Http.HttpConfigurationExtensions类中。

```
using System.Web.Http;

namespace WebApplication
{
    public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            // Web API routes
            config.MapHttpAttributeRoutes();

            // Other Web API configuration not shown.
        }
    }
}

```
属性路由也可以和基于约定的路由结合起来。为了定义基于约定的路由，调用MapHttpRoute方法。

```
public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        // Attribute routing.
        config.MapHttpAttributeRoutes();

        // Convention-based routing.
        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "api/{controller}/{id}",
            defaults: new { id = RouteParameter.Optional }
        );
    }
}

```

关于配置Web API的更多信息，请查看Configuring ASP.NET Web API 2。


在Web API 2之前，Web API项目目标生成的代码像是这样：

```
protected void Application_Start()
{
    // WARNING - Not compatible with attribute routing.
    WebApiConfig.Register(GlobalConfiguration.Configuration);
}

```

如果属性路由没有被启用，这个代码将会抛出异常。如果你升级一个已有的Web API项目来使用属性路由，请确保像下面这样升级了配置代码：

```
protected void Application_Start()
{
    // Pass a delegate to the Configure method.
    GlobalConfiguration.Configure(WebApiConfig.Register);
}

```

备注：关于更多信息，请查看Configuring Web API with ASP.NET Hosting

**添加路由属性（Adding Route Attributes）**
-------------------------------

这里是一个使用属性定义路由的示例：

```
public class OrdersController : ApiController
{
    [Route("customers/{customerId}/orders")]
    [HttpGet]
    public IEnumerable<Order> FindOrdersByCustomer(int customerId) { ... }
}

```

字符串“customers/{customerId}/orders”是一个用于路由的URI模板。Web API会尽力将请求的URI匹配到模板中。在本例中，”customers“和”orders“都是字面字段，而”{customerId}”是变量参数。以下这些URI会匹配这个模板：

```
1，	http://localhost/customers/1/orders
2，	http://localhost/customers/bob/orders
3，	http://localhost/customer/1234-5678/orders
```

你能够使用约束来限制这些匹配，这将会在本主题的后面进行介绍。

注意到路由模板“{customerId}”参数匹配到方法中的customerId参数名。当Web API执行控制器动作时，它会尽力绑定路由参数。例如，当URI是 http: //example.com/customers/1/orders 时，Web API会尽力将值”1“和动作中的customerId参数进行绑定。

一个URI模板可以有多个参数：

```
[Route("customers/{customerId}/orders/{orderId}")]
public Order GetOrderByCustomer(int customerId, int orderId) { ... }

```

任何没有路由属性的控制器方法都使用基于约定的路由。在此基础上，你能够在同一个项目中同时使用这两种路由类型。

**HTTP Methods**
------------

Web API也会基于HTTP方法的请求（GET、POST等）来选择动作。默认地，Web API会根据控制器方法名且不区分大小写地查找匹配。例如，一个控制器方法名为PutCustomers，它匹配一个HTTP的PUT请求。

你也可以通过给方法加上这些属性来重载这个规则：

•	[HttpDelete]
•	[HttpGet]
•	[HttpHead]
•	[HttpOptions]
•	[HttpPatch]
•	[HttpPost]
•	[HttpPut]

下面的例子映射CreateBook方法到HTTP的POST请求。

```
[Route("api/books")]
[HttpPost]
public HttpResponseMessage CreateBook(Book book) { ... }

```

对于所有的HTTP方法，包括非标准方法，可以使用AcceptVerbs属性，它需要传入一个HTTP方法的列表。

```
// WebDAV method
[Route("api/books")]
[AcceptVerbs("MKCOL")]
public void MakeCollection() { }

```

**路由前缀（Route Prefixes）**
--------------------

通常，控制器中的路由都以同样的前缀开始。例如：

```
public class BooksController : ApiController
{
    [Route("api/books")]
    public IEnumerable<Book> GetBooks() { ... }

    [Route("api/books/{id:int}")]
    public Book GetBook(int id) { ... }

    [Route("api/books")]
    [HttpPost]
    public HttpResponseMessage CreateBook(Book book) { ... }
}

```

你可以通过使用[RoutePrefix]属性来为整个控制器设置一个公共前缀。

```
[RoutePrefix("api/books")]
public class BooksController : ApiController
{
    // GET api/books
    [Route("")]
    public IEnumerable<Book> Get() { ... }

    // GET api/books/5
    [Route("{id:int}")]
    public Book Get(int id) { ... }

    // POST api/books
    [Route("")]
    public HttpResponseMessage Post(Book book) { ... }
}

```

使用在方法属性上使用一个通配符（~）来重载路由前缀。

```
[RoutePrefix("api/books")]
public class BooksController : ApiController
{
    // GET /api/authors/1/books
    [Route("~/api/authors/{authorId:int}/books")]
    public IEnumerable<Book> GetByAuthor(int authorId) { ... }

    // ...
}

```
路由前缀也可以包含参数：


```
[RoutePrefix("customers/{customerId}")]
public class OrdersController : ApiController
{
    // GET customers/1/orders
    [Route("orders")]
    public IEnumerable<Order> Get(int customerId) { ... }
}

```

**路由约束（Route Constraints）**
-----------------------


路由约束能够让你限制路由模板中的参数如何被匹配。大体的语法是“{parameter:constraint}”。例如：


```
[Route("users/{id:int}"]
public User GetUserById(int id) { ... }

[Route("users/{name}"]
public User GetUserByName(string name) { ... }

```
在这里，第一个路由只有当URI的“id”字段是整型时才会被选择。否则将会选择第二个路由。

下表列出了被支持的约束。


Constraint	|Description|	Example
-|-|-
alpha|	Matches uppercase or lowercase Latin alphabet characters (a-z, A-Z)	|{x:alpha}
bool	|Matches a Boolean value.	|{x:bool}
datetime	|Matches a DateTime value.	|{x:datetime}
decimal	|Matches a decimal value.	|{x:decimal}
double	|Matches a 64-bit floating-point value.	|{x:double}
float|	Matches a 32-bit floating-point value.|	{x:float}
guid|	Matches a GUID value.	|{x:guid}
int	|Matches a 32-bit integer value.	|{x:int}
length|	Matches a string with the specified length or within a specified range of lengths.|	{x:length(6)} {x:length(1,20)}
long	|Matches a 64-bit integer value.|	{x:long}
max|	Matches an integer with a maximum value.|	{x:max(10)}
maxlength|	Matches a string with a maximum length.	|{x:maxlength(10)}
min|	Matches an integer with a minimum value.	|{x:min(10)}
minlength	|Matches a string with a minimum length.	|{x:minlength(10)}
range	|Matches an integer within a range of values.	|{x:range(10,50)}
regex	|Matches a regular expression.|	{x:regex(^\d{3}-\d{3}-\d{4}$)}

注意到其中一些约束在括号内还需要参数，比如“min”。你可以应用多个约束到一个参数，通过冒号分隔。

```
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { ... }

```

**自定义路由约束（Custom Route Constraints）**
---------------------------------
你可以通过实现IHttpRouteConstraint接口来创建一个自定义路由约束。例如，以下约束限制了一个参数到非零整型值。

```
public class NonZeroConstraint : IHttpRouteConstraint
{
    public bool Match(HttpRequestMessage request, IHttpRoute route, string parameterName, 
        IDictionary<string, object> values, HttpRouteDirection routeDirection)
    {
        object value;
        if (values.TryGetValue(parameterName, out value) && value != null)
        {
            long longValue;
            if (value is long)
            {
                longValue = (long)value;
                return longValue != 0;
            }

            string valueString = Convert.ToString(value, CultureInfo.InvariantCulture);
            if (Int64.TryParse(valueString, NumberStyles.Integer, 
                CultureInfo.InvariantCulture, out longValue))
            {
                return longValue != 0;
            }
        }
        return false;
    }
}

```

下面的代码展示了如何去注册约束：


```
public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        var constraintResolver = new DefaultInlineConstraintResolver();
        constraintResolver.ConstraintMap.Add("nonzero", typeof(NonZeroConstraint));

        config.MapHttpAttributeRoutes(constraintResolver);
    }
}

```
现在你可以将该约束应用到你的路由中了：

```
[Route("{id:nonzero}")]
public HttpResponseMessage GetNonZero(int id) { ... }

```

你也可以通过实现IInlineConstraintResolver接口来替换整个DefaultInlineConstraintResolver类。这样做会替换掉所有的内建约束，除非你实现的IInlineConstraintResolver特意添加了它们。


**可选的URI参数和默认值**
------------

你可以通过添加问好标记到路由参数让一个URI参数变成可选的。如果一个路由参数是可选的，你必须为方法参数定义默认值。


```
public class BooksController : ApiController
{
    [Route("api/books/locale/{lcid:int?}")]
    public IEnumerable<Book> GetBooksByLocale(int lcid = 1033) { ... }
}

```
在本例中，/api/books/locale/1033和/api/books/locale会返回相同的资源。

或者，你可以特定一个默认值在路由模板中，如下所示：


```
public class BooksController : ApiController
{
    [Route("api/books/locale/{lcid:int=1033}")]
    public IEnumerable<Book> GetBooksByLocale(int lcid) { ... }
}

```

这和前一个例子大体相同，但当默认值被应用时存在细微差别。
1，	在第一个例子（“{Icid?}”），默认值1033会被直接分配到方法参数，所以参数将会拥有一个准确的值。
2，	在第二个例子（“{Icid=1033}”），默认值1033会通过模型绑定过程。默认的模型绑定将会把1033转换成数字值1033。然而，你可以遇到一个自定义的模型绑定，而这可能会出错。
（多数情况下，除非你在你的管道中有自定义模型绑定，否则这两只表单形式是等价的。）

**路由名称（Route Names）**
----------------

在Web API中，每种路由都有一个名称。路由名称对于生成链接是非常有用的，正因此你才能在HTTP相应中包含一个链接。

为了指定路由名称，在属性上（attribute）设置Name属性（property）。以下示例展示了如何选择一个路由名称，以及当生成一个链接时如何使用路由名称。

```
public class BooksController : ApiController
{
    [Route("api/books/{id}", Name="GetBookById")]
    public BookDto GetBook(int id) 
    {
        // Implementation not shown...
    }

    [Route("api/books")]
    public HttpResponseMessage Post(Book book)
    {
        // Validate and add book to database (not shown)

        var response = Request.CreateResponse(HttpStatusCode.Created);

        // Generate a link to the new book and set the Location header in the response.
        string uri = Url.Link("GetBookById", new { id = book.BookId });
        response.Headers.Location = new Uri(uri);
        return response;
    }
}

```

**路由顺序（Route Order）**
-----------------


当框架试图用路由匹配URI时，它会得到一个特定的路由顺序。为了指定顺序，在路由属性上设置RouteOrder属性。小写的值在前，默认顺序值是零。

以下是如何确定所有的顺序的过程：
1.	比较每个路由属性的RouteOrder属性
2.	在路由模板上查找每个URI字段。对于每个字段，顺序由以下因素确定：
	-	字面字段
	-	包含约束的路由参数
	-	不包含约束的路由参数
	- 包含约束的通配符参数字段
	- 不包含约束的通配符参数字段
3. In the case of a tie，路由的顺序由路由模板的不区分大小写的原始字符串比较来确定。

这是一个示例。假定你定义如下控制器：


```
[RoutePrefix("orders")]
public class OrdersController : ApiController
{
    [Route("{id:int}")] // constrained parameter
    public HttpResponseMessage Get(int id) { ... }

    [Route("details")]  // literal
    public HttpResponseMessage GetDetails() { ... }

    [Route("pending", RouteOrder = 1)]
    public HttpResponseMessage GetPending() { ... }

    [Route("{customerName}")]  // unconstrained parameter
    public HttpResponseMessage GetByCustomer(string customerName) { ... }

    [Route("{*date:datetime}")]  // wildcard
    public HttpResponseMessage Get(DateTime date) { ... }
}

```


这些路由的顺序如下：

1.	orders/details
2.	orders/{id}
3.	orders/{customerName}
4.	orders/{*date}
5.	orders/pending

注意到“details”是一个字面字段，并且出现在“{id}”的前面，而“pending”出现在最后是因为它的RouteOrder是1。（这个例子假定不存在customer被命名为”details”和“pending”。通常来说，要尽量避免含糊不清的路由。在本例中，对于GetByCustomer的一个更好的路由模板是”customers/{customerName}”。）


