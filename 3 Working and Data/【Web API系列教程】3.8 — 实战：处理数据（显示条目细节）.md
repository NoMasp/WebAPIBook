在本节，你将添加查看每本书的详细信息的功能。在app.js中，添加以下代码到视图模型：

```
self.detail = ko.observable();

self.getBookDetail = function (item) {
    ajaxHelper(booksUri + item.Id, 'GET').done(function (data) {
        self.detail(data);
    });
}

```
在Views/Home/Index.cshtml，添加一个数据绑定元素到Details链接：

```
<ul class="list-unstyled" data-bind="foreach: books">
  <li>
    <strong><span data-bind="text: AuthorName"></span></strong>: <span data-bind="text: Title"></span>
    <!-- New code -->
    <small><a href="#" data-bind="click: $parent.getBookDetail">Details</a></small>
  </li>
</ul>

```
它为< a >元素绑定了一个在视图模型中调用getBookDetail函数的点击事件。

在同一个文件，替换以下代码：

```
<div class="col-md-4">
    <!-- TODO: Book details -->
</div>

```
到：

```
<!-- ko if:detail() -->

<div class="col-md-4">
<div class="panel panel-default">
  <div class="panel-heading">
    <h2 class="panel-title">Detail</h2>
  </div>
  <table class="table">
    <tr><td>Author</td><td data-bind="text: detail().AuthorName"></td></tr>
    <tr><td>Title</td><td data-bind="text: detail().Title"></td></tr>
    <tr><td>Year</td><td data-bind="text: detail().Year"></td></tr>
    <tr><td>Genre</td><td data-bind="text: detail().Genre"></td></tr>
    <tr><td>Price</td><td data-bind="text: detail().Price"></td></tr>
  </table>
</div>
</div>

<!-- /ko -->

```
这段代码创建了一个绑定到视图模型中details各个属性的表。

其中的“<!—ko -->”句法让你在DOM元素外部包括一个Knockout绑定。在本例中，if绑定导致本节的标记要显示详细信息时才为非空。

```
<!-- ko if:detail() -->
```


现在如果你运行这个应用程序，并点击其中一个“Detail“链接，这个app会展示出book的详细信息。

![这里写图片描述](http://img.blog.csdn.net/20160227092118493)



























































































