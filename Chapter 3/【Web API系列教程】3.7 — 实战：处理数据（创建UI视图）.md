在本节，你将开始为app定义HTML，并在HTML和视图模型间添加数据绑定。

打开Views/Home/Index.cshtml文件。用以下代码替换掉文件的所有内容。

```
@section scripts {
  @Scripts.Render("~/bundles/app")
}

<div class="page-header">
  <h1>BookService</h1>
</div>

<div class="row">

  <div class="col-md-4">
    <div class="panel panel-default">
      <div class="panel-heading">
        <h2 class="panel-title">Books</h2>
      </div>
      <div class="panel-body">
        <ul class="list-unstyled" data-bind="foreach: books">
          <li>
            <strong><span data-bind="text: AuthorName"></span></strong>: <span data-bind="text: Title"></span>
            <small><a href="#">Details</a></small>
          </li>
        </ul>
      </div>
    </div>
    <div class="alert alert-danger" data-bind="visible: error"><p data-bind="text: error"></p></div>
  </div>

  <div class="col-md-4">
    <!-- TODO: Book details -->
  </div>

  <div class="col-md-4">
    <!-- TODO: Add new book -->
  </div>
</div>

```
这里的绝大部分div元素都是Bootstrap风格，它是有着数据绑定属性的重要元素。这个元素将HTML链接到视图模型中。

例如：

```
<p data-bind="text: error">
```

在这个示例中，“text”绑定会使得p元素显示在视图模型中error属性的值。error的回调被声明在ko.obserable中：

```
self.error = ko.observable(); 
```

无论何时一个新值被修改到error上，Knockout都会在< p >元素上更新该文本。

而foreach绑定告诉Knockout在books数组中循环遍历。对于该数组的每一项，Knockout会创建一个新的< li \>元素。绑定foreach引用的上下文到数组的每一项。例如：

```
<span data-bind="text: Author"></span>
```

这里有一个text绑定读取每一个book的Author属性。

如果你现在运行该应用程序，它看起来会是这样：

![这里写图片描述](http://img.blog.csdn.net/20160227091847552)

在页面加载后，books列表会被异步的加载。现在，“Details”链接还不具备功能。我们将在下一节为其添加此功能。






















































































