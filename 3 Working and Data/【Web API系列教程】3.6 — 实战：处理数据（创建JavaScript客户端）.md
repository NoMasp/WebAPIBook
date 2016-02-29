在本节，你将使用HTML、JavaScript和Knockout.js库为应用程序创建客户端。我们将按如下步骤建立客户端应用：
1，	展示books列表
2，	展示book详细信息
3，	添加一本新书

Knockout.js库使用了模型-视图-视图模型（MVVM）模式：
1，	模型是在业务域(在本例中是books和authors)中数据在服务器端的表现形式。
2，	视图是表示层（HTML）。
3，	视图模型是维持模型的JavaScript对象。视图模型是UI的代码抽象。它不具备HTML表现形式，相反，它表示抽象特征的视图，例如”书籍列表“。

视图被数据绑定到视图模型。视图模型的更新将自动反映在视图中。视图模型也从视图中获取事件，比如按钮的点击。

![这里写图片描述](http://img.blog.csdn.net/20160227091013547)

这个实现使得在你的app中修改布局和UI更加容易，因为你可以改变这些绑定而无须任何代码。例如，你可能以ul的方式展示一个项列表，那么可以将其改变成表。

**添加Knockout库**
-----------

在Visual Studio中，点击Tools目录，选择Library Package Manager。然后选择Package Manager Console。在Package Manager Console窗口，输入以下命令：


```
Install-Package knockoutjs
```

这条命令将Knockout文件添加到Scripts文件夹下。

**创建视图模型**
------


在Scripts文件夹下添加一个名为app.js的JavaScript文件。（在Solution Explorer中，右击Scripts文件夹，选择Add，然后选择JavaScript File。）粘贴以下代码：


```
var ViewModel = function () {
    var self = this;
    self.books = ko.observableArray();
    self.error = ko.observable();

    var booksUri = '/api/books/';

    function ajaxHelper(uri, method, data) {
        self.error(''); // Clear error message
        return $.ajax({
            type: method,
            url: uri,
            dataType: 'json',
            contentType: 'application/json',
            data: data ? JSON.stringify(data) : null
        }).fail(function (jqXHR, textStatus, errorThrown) {
            self.error(errorThrown);
        });
    }

    function getAllBooks() {
        ajaxHelper(booksUri, 'GET').done(function (data) {
            self.books(data);
        });
    }

    // Fetch the initial data.
    getAllBooks();
};

ko.applyBindings(new ViewModel());

```

在Knockout中，observable类启用了数据绑定。当observable的内容改变时，observable会通知所有的数据绑定控制器，所以它们能够去更新它们自身。（而observable类是一个observable的数组版本。）以此开始，我们的视图模型有了两个observable：
1，	books维持books列表。
2，	error包含如果AJAX调用失败时的错误信息

该getAllbooks方法产生AJAX调用以得到books列表。然后将其结果添加到books数组中。

而ko.applyBinding方法是Knockout库的一部分。它使用视图模型作为一个参数并建立数据绑定。

**添加Script Bundle**
---------------

Bundle是一个在ASP.NET 4.5中出现的新特性，它使得组合或包装多个文件进一个文件更加容易。Bundle减少了向服务器的请求，而这恰能改进页面加载时间。

打开App_Start文件夹下的BundleConfig.cs文件，添加如下代码到RegisterBundles方法。

```
public static void RegisterBundles(BundleCollection bundles)
{
    // ...

    // New code:
    bundles.Add(new ScriptBundle("~/bundles/app").Include(
              "~/Scripts/knockout-{version}.js",
              "~/Scripts/app.js"));
}
```






































































