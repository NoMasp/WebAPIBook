**前言**
--

在本部分中，你将在EF上使用Code First Migration来用测试数据建立数据库。

在Tools目录下选择Library Package Manager，然后选择Package Manager Console。在包管理控制台窗口，输入以下命令：

```
Enable-Migrations
```
这条命令会添加一个名为Migrations的文件夹到你的项目，并添加一个名为Configuration.cs的代码文件到Migrations文件夹。

![这里写图片描述](http://img.blog.csdn.net/20160226171747887)

如果在BookService中出现多种上下文类型，请输入”Enable-Migrations –ContextTypeName BookService.Models.BookServiceContext”，具体请看下图。——译者注。

![这里写图片描述](http://img.blog.csdn.net/20160226171814918)

打开Configuration.cs文件。添加以下using语句。

```
using BookService.Models;
```
然后添加以下代码到Configuration.Seed方法：

```
protected override void Seed(BookService.Models.BookServiceContext context)
{
    context.Authors.AddOrUpdate(x => x.Id,
        new Author() { Id = 1, Name = "Jane Austen" },
        new Author() { Id = 2, Name = "Charles Dickens" },
        new Author() { Id = 3, Name = "Miguel de Cervantes" }
        );

    context.Books.AddOrUpdate(x => x.Id,
        new Book() { Id = 1, Title = "Pride and Prejudice", Year = 1813, AuthorId = 1, 
            Price = 9.99M, Genre = "Comedy of manners" },
        new Book() { Id = 2, Title = "Northanger Abbey", Year = 1817, AuthorId = 1, 
            Price = 12.95M, Genre = "Gothic parody" },
        new Book() { Id = 3, Title = "David Copperfield", Year = 1850, AuthorId = 2, 
            Price = 15, Genre = "Bildungsroman" },
        new Book() { Id = 4, Title = "Don Quixote", Year = 1617, AuthorId = 3, 
            Price = 8.95M, Genre = "Picaresque" }
        );
}

```
在Package Manager Console窗口，键入以下命令：

```
Add-Migration Initial
Update-Database

```
第一条命令生成用于创建数据库的代码，第二条命令执行那些代码。数据库使用LocalDB并创建于本地。

![这里写图片描述](http://img.blog.csdn.net/20160226171904419)

**探索API（可选）**
---------

按F5在debug模式下运行应用程序。Visual Studio启动IIS Express并运行你的web应用。Visual Studio会启动一个浏览器并打开app的主页。

当Visual Studio运行了这个web项目，它会给定一个端口号。在下图中，端口号是50524。当你运行应用程序的时候，你可能会看到不同的端口号。

![这里写图片描述](http://img.blog.csdn.net/20160226171940733)

主页使用ASP.NET MVC来实现。在页面顶部有一个写着“API”的链接。该链接会带你去一个自动生成的关于Web API的帮助页面。（想了解这个帮助页面如何生成的，以及你怎样添加你自己的文档进该页面，查看Creating Help Pages for ASP.NET Web API（http://www.asp.net/web-api/overview/creating-web-apis/creating-api-help-pages）。）你可以点击帮助页面的链接以查看API的详细信息，包括请求和相应的格式。

![这里写图片描述](http://img.blog.csdn.net/20160226172011294)

该API支持在数据库上执行CRUD操作。下表是关于API的总结。


Authors| 备注
-|-
GET api/authors|	Get all authors.
GET api/authors/{id}	|Get an author by ID.
POST /api/authors	|Create a new author.
PUT /api/authors/{id}	|Update an existing author.
DELETE /api/authors/{id}	|Delete an author.

Books|备注
-|-
GET /api/books|	Get all books.
GET /api/books/{id}	|Get a book by ID.
POST /api/books	|Create a new book.
PUT /api/books/{id}	|Update an existing book.
DELETE /api/books/{id}|	Delete a book.

**查看数据库（可选）**
---------
当你执行了Update-Database命令，EF会创建数据库并调用Seed方法。当你在本地执行了应用程序后，EF会使用LocalDB。你可以在Visual Studio中查看数据库。在View目录下，选择SQL Server Object Explorer。

![这里写图片描述](http://img.blog.csdn.net/20160226172153845)

在Connect to Server对话框中，在Server Name编辑框，键入“(localdb)\v11.0”。保留Authentication选项为”Windows Authentication”。点击Connect。

![这里写图片描述](http://img.blog.csdn.net/20160226172212404)

Visual Studio会连接到LocalDB并在SQL Server Object Explorer窗口显示已经存在的数据库。你可以展开该节点查看EF创建的表。

![这里写图片描述](http://img.blog.csdn.net/20160226172308733)

为了看到数据，右击一个表并选择View Data。

![这里写图片描述](http://img.blog.csdn.net/20160226172330967)

下面的截图显示了Books表的结果。注意EF通过seed数据聚集了数据库，并且该表包含了指向Authors表的外键。

![这里写图片描述](http://img.blog.csdn.net/20160226172346488)
















