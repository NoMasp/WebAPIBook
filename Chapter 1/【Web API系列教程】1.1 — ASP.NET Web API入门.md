**前言**
--

HTTP不仅仅服务于web页面，同时也是构建暴露服务和数据的API的强大平台。HTTP有着简单、灵活和无处不在的特点。你能想到的几乎所有平台都包含有一个HTTP库，所以HTTP服务可以遍及广泛的客户端，包括浏览器、移动设备和传统桌面应用程序。

ASP.NET Web API是一个在.NET框架上构建web API的框架。在本教程中，你将使用ASP.NET Web API来创建一个返回产品列表的web API。

**创建Web API项目**
-----------
在本教程中，你将使用ASP.NET Web API来创建一个返回产品列表的web API。前端页面使用jQuery来显示结果。

![这里写图片描述](http://img.blog.csdn.net/20160224182107579)

开启Visual Studio并在开始页面选择New Project。或者在File目录下选择New，然后选择Project。

在Template面板中，选择Installed Templates，然后展开Visual C#节点。在Visual C#节点下，选择Web。在项目模板列表中，选择ASP.NET Web Application。命名项目为“ProductsApp”并点击OK。


![这里写图片描述](http://img.blog.csdn.net/20160224182313597)

在NEW ASP.NET Project对话框中，选择Empty模板。在”Add folders and core references for”，选中Web API。点击OK。

注释：你也可以用“Web API”模板来创建Web API。Web API模板使用了ASP.NET MVC来提供API的帮助页面。我在本教程中使用Empty模板是因为我希望不用MVC来展示Web API。通常，你不必了解ASP.NET MVC就能使用Web API。

**添加模型**
----

模型是在你的应用程序中表示数据的对象。ASP.NET Web API能够将你的模型自动序列化成JSON、XML或其他格式，然后将其序列化数据写入到HTTP响应消息的body中。只要客户端能够读取序列化格式，它就能够反序列化出对象。几乎所有客户端都能解析XML或JSON。而且，客户端还能通过在HTTP请求的Accept header中设置来指明它想要的格式。

让我们来创建一个表示产品的简单模型吧。

如果Solution Explorer没有显示出来，点击View菜单，然后选择Solution Explorer。在Solution Explorer中，右击Models文件夹。从上下文菜单中选择Add，然后选择Class。

![这里写图片描述](http://img.blog.csdn.net/20160224182418786)

命名该类为“Product”。添加以下属性到Product类中。

```
namespace ProductsApp.Models
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Category { get; set; }
        public decimal Price { get; set; }
    }
}
```

**添加控制器**
-----

在Web API中，控制器（controller）是处理HTTP请求的对象。我们将添加一个能够根据ID来返回多个或一个产品的控制器。

备注：如果你还没有使用过ASP.NET MVC，你应该已经对控制器很熟悉了。Web API的控制器和MVC的控制器很相近，但是它继承的是ApiController而不是Controller。

在Solution Explorer中，右击Controllers文件夹。选择Add，然后选择Controller。

![这里写图片描述](http://img.blog.csdn.net/20160224182504913)

在Add Scaffold对话框中，选择Web API Controller – Empty。点击Add。

![这里写图片描述](http://img.blog.csdn.net/20160224182519990)

在Add Controller对话框，给控制器命名为”ProductsController”。点击Add。

![这里写图片描述](http://img.blog.csdn.net/20160224182543648)

接下来便会在Controllers文件夹下创建一个名为ProductsController.cs的文件。

![这里写图片描述](http://img.blog.csdn.net/20160224182616195)

备注：其实你不必非得把控制器添加到Controllers文件夹下。文件夹名称只是为了更方便你组织源文件。

如果文件没有打开，那就双击文件打开它。在文件中替换成以下代码：

```
using ProductsApp.Models;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Web.Http;

namespace ProductsApp.Controllers
{
    public class ProductsController : ApiController
    {
        Product[] products = new Product[] 
        { 
            new Product { Id = 1, Name = "Tomato Soup", Category = "Groceries", Price = 1 }, 
            new Product { Id = 2, Name = "Yo-yo", Category = "Toys", Price = 3.75M }, 
            new Product { Id = 3, Name = "Hammer", Category = "Hardware", Price = 16.99M } 
        };

        public IEnumerable<Product> GetAllProducts()
        {
            return products;
        }

        public IHttpActionResult GetProduct(int id)
        {
            var product = products.FirstOrDefault((p) => p.Id == id);
            if (product == null)
            {
                return NotFound();
            }
            return Ok(product);
        }
    }
}

```

为了让示例简单化，products被存储在控制器类中的固定数组中。当然，在实际应用程序中，你可能想要查询数据库或使用其他外部数据源。

控制器定义了两个返回产品的方法：
1. GetAllProducts方法将整个产品列表作为IEnumerable<Product>类型返回。
2. GetProduct方法通过它的ID来查找单个产品。


没错，你已经有一个可以使用的web API了。控制器上的每个方法都对应一个或多个URI：

Controlle Method	|URI
-|-
GetAllProducts	|/api/products
GetProduct	|/api/products/id

对于GetProduct方法，URI中的id是一个占位符。例如，为了得到一个ID为5的产品，URI是api/products/5。

**使用JavaScript和jQuery来调用Web API**
-----------------------------

在本节中，我们将添加一个使用AJAX来调用Web API的HTML页面。我们将使用jQuery来产生AJAX调用并用返回结果来更新页面。

在Solution Explorer中，右击项目并选择Add，然后选择New Item。

![这里写图片描述](http://img.blog.csdn.net/20160224183004685)

在Add New Item对话框中，选择Visual C#下的Web节点，然后选择HTML Page选项。命名页面为“index.html”。

![这里写图片描述](http://img.blog.csdn.net/20160224183025915)

用以下代码替换文件中的全部：

```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
  <title>Product App</title>
</head>
<body>

  <div>
    <h2>All Products</h2>
    <ul id="products" />
  </div>
  <div>
    <h2>Search by ID</h2>
    <input type="text" id="prodId" size="5" />
    <input type="button" value="Search" onclick="find();" />
    <p id="product" />
  </div>

  <script src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-2.0.3.min.js"></script>
  <script>
    var uri = 'api/products';

    $(document).ready(function () {
      // Send an AJAX request
      $.getJSON(uri)
          .done(function (data) {
            // On success, 'data' contains a list of products.
            $.each(data, function (key, item) {
              // Add a list item for the product.
              $('<li>', { text: formatItem(item) }).appendTo($('#products'));
            });
          });
    });

    function formatItem(item) {
      return item.Name + ': $' + item.Price;
    }

    function find() {
      var id = $('#prodId').val();
      $.getJSON(uri + '/' + id)
          .done(function (data) {
            $('#product').text(formatItem(data));
          })
          .fail(function (jqXHR, textStatus, err) {
            $('#product').text('Error: ' + err);
          });
    }
  </script>
</body>
</html>

```
有好几个方法去得到jQuery。在本例中，我使用Microsoft Ajax CDN。你也可以在http://jquery.com/下载它，让ASP.NET “Web API”项目包含jQuery。

**得到产品列表**
------------

为了得到Products列表，可以发送一个HTTP的GET请求到“/api/products”。

jQuery的getJSON函数会发送AJAX请求。其中包含了JSON对象数组。done函数指定了一个当请求成功时触发的回调。在回调中，我们用产品信息更新DOM。

```
$(document).ready(function () {
    // Send an AJAX request
    $.getJSON(apiUrl)
        .done(function (data) {
            // On success, 'data' contains a list of products.
            $.each(data, function (key, item) {
                // Add a list item for the product.
                $('<li>', { text: formatItem(item) }).appendTo($('#products'));
            });
        });
});

```

**通过ID得到产品**
-------------

如果想要通过ID来取得产品，可以发送HTTP的GET请求到”/api/products/id“，其中id就是产品的ID。

```
function find() {
    var id = $('#prodId').val();
    $.getJSON(apiUrl + '/' + id)
        .done(function (data) {
            $('#product').text(formatItem(data));
        })
        .fail(function (jqXHR, textStatus, err) {
            $('#product').text('Error: ' + err);
        });
}

```

我们仍然使用getJSON来发送AJAX请求，但是这次我们将ID放到URI请求中。它的响应会是一个代表了单个产品的JSON对象。

**运行应用程序**
------
按F5开始调试应用程序，web页面看起来会是下面这样：

![这里写图片描述](http://img.blog.csdn.net/20160224183641100)

为了通过ID获得产品，输入ID并点击Search。

![这里写图片描述](http://img.blog.csdn.net/20160224183717288)

如果你输入了一个无效的ID，那么服务器就会返回HTTP错误消息。

![这里写图片描述](http://img.blog.csdn.net/20160224183752887)

**使用F12查看HTTP请求和响应**
----------------

当你工作于HTTP服务时，如果能够查看HTTP请求和响应的详细无疑是非常有帮助的。你可以在IE9中使用F12开发者工具来做这些操作。在IE9中，按F12来打开工具。点击Network面板，并点击Start Capturing。现在返回到web页面，并按F5来重新加载web页面。IE将会捕捉到浏览器和web服务器之间的HTTP传输。下图显示了一个页面的所有HTTP传输。

![这里写图片描述](http://img.blog.csdn.net/20160224184117809)

定位到相对URI”api/products/“。选中并点击Go to detailed view。在详细视图中，这里多个面板用于查看请求和响应的header和body。

例如，如果你点击Request headers，你就会看到客户端在Accept header请求了”application/json“。

![这里写图片描述](http://img.blog.csdn.net/20160224184543065)

如果你点击了Response body，你就会看到产品列表如何被序列化成JSON。其他浏览器也有相似的功能。另一个有用的工具是Fiddler，它是一个web调试代理工具。你可以使用Fiddler来查看HTTP传输，也可以合成HTTP请求，后者能够给予你在请求上对于HTTP头部的完全控制。