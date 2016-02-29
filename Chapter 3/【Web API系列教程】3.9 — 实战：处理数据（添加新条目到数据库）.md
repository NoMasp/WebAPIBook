在本节，你将添加让用户可以创建新book的功能。在app.js中，添加如下代码到视图模型：

```
self.authors = ko.observableArray();
self.newBook = {
    Author: ko.observable(),
    Genre: ko.observable(),
    Price: ko.observable(),
    Title: ko.observable(),
    Year: ko.observable()
}

var authorsUri = '/api/authors/';

function getAuthors() {
    ajaxHelper(authorsUri, 'GET').done(function (data) {
        self.authors(data);
    });
}

self.addBook = function (formElement) {
    var book = {
        AuthorId: self.newBook.Author().Id,
        Genre: self.newBook.Genre(),
        Price: self.newBook.Price(),
        Title: self.newBook.Title(),
        Year: self.newBook.Year()
    };

    ajaxHelper(booksUri, 'POST', book).done(function (item) {
        self.books.push(item);
    });
}

getAuthors();

```

在Index.cshtml中，替换以下代码：

```
<div class="col-md-4">
    <!-- TODO: Add new book -->
</div>

```
到：

```
<div class="col-md-4">
<div class="panel panel-default">
  <div class="panel-heading">
    <h2 class="panel-title">Add Book</h2>
  </div>

  <div class="panel-body">
    <form class="form-horizontal" data-bind="submit: addBook">
      <div class="form-group">
        <label for="inputAuthor" class="col-sm-2 control-label">Author</label>
        <div class="col-sm-10">
          <select data-bind="options:authors, optionsText: 'Name', value: newBook.Author"></select>
        </div>
      </div>

      <div class="form-group" data-bind="with: newBook">
        <label for="inputTitle" class="col-sm-2 control-label">Title</label>
        <div class="col-sm-10">
          <input type="text" class="form-control" id="inputTitle" data-bind="value:Title"/>
        </div>

        <label for="inputYear" class="col-sm-2 control-label">Year</label>
        <div class="col-sm-10">
          <input type="number" class="form-control" id="inputYear" data-bind="value:Year"/>
        </div>

        <label for="inputGenre" class="col-sm-2 control-label">Genre</label>
        <div class="col-sm-10">
          <input type="text" class="form-control" id="inputGenre" data-bind="value:Genre"/>
        </div>

        <label for="inputPrice" class="col-sm-2 control-label">Price</label>
        <div class="col-sm-10">
          <input type="number" step="any" class="form-control" id="inputPrice" data-bind="value:Price"/>
        </div>
      </div>
      <button type="submit" class="btn btn-default">Submit</button>
    </form>
  </div>
</div>
</div>

```
这段代码创建了一个表单，用于提交新的作者。作者下拉框的值被数据绑定到视图模型的authors中。对于其他的表单输入，这些值都被数据绑定到视图模型的newBook属性。

这个表单上的提交事件被数据绑定到addBook函数：

```
<form class="form-horizontal" data-bind="submit: addBook">
```

这个addBook函数读取数据绑定表单输入中的当前值，并创建JSON对象。然后会POST这个JSON对象到/api/books。


































































