**前言**
--

本节的主题是ASP.NET Web API如何将控制器动作的返回值转换成HTTP的响应消息。

Web API控制器动作可以返回下列的任何值：
1，	void
2，	HttpResponseMessage
3，	IHttpActionResult
4，	Some other type

取决于返回的以上哪一种，Web API使用不同的机制来创建HTTP响应。


Return type	|How Web API creates the response
-|-
void	|Return empty 204 (No Content)
HttpResponseMessage|	Convert directly to an HTTP response message.
IHttpActionResult|	Call ExecuteAsync to create an HttpResponseMessage, then convert to an HTTP response message.
Other type	|Write the serialized return value into the response body; return 200 (OK).

本节的剩余部分将详细描述每种返回值。


**void**
----

如果返回类型是type，Web API就会用状态码204（No Content）返回一个空HTTP响应。

示例控制器：

```
public class ValuesController : ApiController
{
    public void Post()
    {
    }
}

```

HTTP相应：

```
HTTP/1.1 204 No Content
Server: Microsoft-IIS/8.0
Date: Mon, 27 Jan 2014 02:13:26 GMT

```

**HttpResponseMessage**
-------------------

如果一个动作返回HttpResponseMessage，Web API就通过HttpResponseMessage的属性构造成消息从而直接将返回值转换成HTTP响应。

```
public class ValuesController : ApiController
{
    public HttpResponseMessage Get()
    {
        HttpResponseMessage response = Request.CreateResponse(HttpStatusCode.OK, "value");
        response.Content = new StringContent("hello", Encoding.Unicode);
        response.Headers.CacheControl = new CacheControlHeaderValue()
        {
            MaxAge = TimeSpan.FromMinutes(20)
        };
        return response;
    } 
}

```
相应：

```
HTTP/1.1 200 OK
Cache-Control: max-age=1200
Content-Length: 10
Content-Type: text/plain; charset=utf-16
Server: Microsoft-IIS/8.0
Date: Mon, 27 Jan 2014 08:53:35 GMT

hello

```
如果传递一个域模型给CreateResponse方法，Web API会使用媒体格式(media formatter)将序列化模型写入到响应体中。


```
public HttpResponseMessage Get()
{
    // Get a list of products from a database.
    IEnumerable<Product> products = GetProductsFromDB();

    // Write the list to the response body.
    HttpResponseMessage response = Request.CreateResponse(HttpStatusCode.OK, products);
    return response;
}

```

**IHttpActionResult**
-----------------

IHttpActionResult接口在Web API 2中被引进。本质上，它定义了一个HttpResponseMessage工厂。以下是使用IHttpActionResult接口的好处：
1，	简单你的控制器的单元测试
2，	为创建HTTP相应将公共逻辑移动到单独的类
3，	通过隐藏构建相应的底层细节，使控制器动作更清晰

IHttpActionResult包含一个单独的方法ExecuteAsync，它会异步地创建一个HttpResponseMessage实例：

```
public interface IHttpActionResult
{
    Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken);
} 

```
如果一个控制器动作返回IHttpActionResult，Web API会调用ExecuteAsync方法来创建HttpResponseMessage。然后将HttpResponseMessage转换到HTTP相应消息里。

以下是一个IHttpActionResult的简单执行，它创建一个文本相应：

```
public class TextResult : IHttpActionResult
{
    string _value;
    HttpRequestMessage _request;

    public TextResult(string value, HttpRequestMessage request)
    {
        _value = value;
        _request = request;
    }
    public Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        var response = new HttpResponseMessage()
        {
            Content = new StringContent(_value),
            RequestMessage = _request
        };
        return Task.FromResult(response);
    }
}

```
控制器动作示例：

```
public class ValuesController : ApiController
{
    public IHttpActionResult Get()
    {
        return new TextResult("hello", Request);
    }
}

```
相应：

```
HTTP/1.1 200 OK
Content-Length: 5
Content-Type: text/plain; charset=utf-8
Server: Microsoft-IIS/8.0
Date: Mon, 27 Jan 2014 08:53:35 GMT

hello

```
更通常的情况是，你会使用System.Web.Http.Results命名空间下定义的IHttpActionResult实现。

在接下来的示例中，如果请求没有匹配到任何已存在的产品ID，控制器就会调用ApiController.NotFound来创建一个404（Not Found）响应。否则，控制器会调用ApiController.OK，它会调用一个意为包含该产品的200（OK）相应。

```
public IHttpActionResult Get (int id)
{
    Product product = _repository.Get (id);
    if (product == null)
    {
        return NotFound(); // Returns a NotFoundResult
    }
    return Ok(product);  // Returns an OkNegotiatedContentResult
}

```

**Other Return Types**
------------------
对于其他所有返回类型，Web API使用媒体格式（media formatter）来序列化返回值。Web API将序列化值写入到响应体中。响应状态码是200（OK）。

```
public class ProductsController : ApiController
{
    public IEnumerable<Product> Get()
    {
        return GetAllProductsFromDB();
    }
}

```

该实现的缺点在于你不能直接返回一个错误码，比如404。

Web API通过在请求中使用Accept头来选择格式。

示例请求：

```
GET http://localhost/api/products HTTP/1.1
User-Agent: Fiddler
Host: localhost:24127
Accept: application/json

```
示例相应：

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Server: Microsoft-IIS/8.0
Date: Mon, 27 Jan 2014 08:53:35 GMT
Content-Length: 56

[{"Id":1,"Name":"Yo-yo","Category":"Toys","Price":6.95}]

```







