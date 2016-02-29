**前言**
--

本部分描述了EF如何加载相关实体的细节，并且如何在你的模型类中处理环形导航属性。（本部分预备了背景知识，而这不是完成这个教程所必须的。你也可以跳到第五节）

**预加载和延迟加载**
--------

预加载和延迟加载的英文名称分别是Eager Loading和Lazy Loading。

当EF与关系数据库一同使用时，了解EF是如何加载相关数据是非常重要的。

去查看EF生成的SQL查询也是很有帮助的。为了追踪SQL，添加下列代码到BookServiceContext构造器中：

```
public BookServiceContext() : base("name=BookServiceContext")
{
    // New code:
    this.Database.Log = s => System.Diagnostics.Debug.WriteLine(s);
}

```
如果发送一个GET请求到/api/books，它返回像下面这样的JSON：

```
[
  {
    "BookId": 1,
    "Title": "Pride and Prejudice",
    "Year": 1813,
    "Price": 9.99,
    "Genre": "Comedy of manners",
    "AuthorId": 1,
    "Author": null
  },
  ...

```
你能看到Author属性是空的，即便book包含有效的AuthorId。那是因为EF没有在加载相关的Author实体。关于SQL查询的跟踪日志如下：

```
SELECT 
    [Extent1].[BookId] AS [BookId], 
    [Extent1].[Title] AS [Title], 
    [Extent1].[Year] AS [Year], 
    [Extent1].[Price] AS [Price], 
    [Extent1].[Genre] AS [Genre], 
    [Extent1].[AuthorId] AS [AuthorId]
    FROM [dbo].[Books] AS [Extent1]

```
该SQL跟踪在Visual Studio的Output窗口中显示。——译者注

SELECT语句从Books表中获取数据，但并没有引用Author表。
作为参考，这里是在BooksController类中的方法，它返回books的列表。

```
public IQueryable<Book> GetBooks()
{
    return db.Books;
}

```
来看看我们如何才能让Author作为返回的JSON数据的一部分。在Entity Framework中有三种方式加载相关数据：预加载（eager loading）、延迟加载（lazy loading）和显式加载（explicit loading）。我们应该在这三种技术中有所取舍，所以了解它们是如何工作的就非常重要了。

**Eager Loading(预加载)**
------------------

在预加载中，EF加载相关数据作为初始化数据库查询的一部分。为了执行预加载，使用System.Data.Entity.Include扩展方法。

```
public IQueryable<Book> GetBooks()
{
    return db.Books
        // new code:
        .Include(b => b.Author);
}

```

这会告诉EF将Author数据包含在查询中。如果你做了这个改变并运行了app，现在JSON数据会是如下所示：

```
[
  {
    "BookId": 1,
    "Title": "Pride and Prejudice",
    "Year": 1813,
    "Price": 9.99,
    "Genre": "Comedy of manners",
    "AuthorId": 1,
    "Author": {
      "AuthorId": 1,
      "Name": "Jane Austen"
    }
  },
  ...

```
其跟踪日志显示EF在Book和Author表中执行了一个join操作。

```
SELECT 
    [Extent1].[BookId] AS [BookId], 
    [Extent1].[Title] AS [Title], 
    [Extent1].[Year] AS [Year], 
    [Extent1].[Price] AS [Price], 
    [Extent1].[Genre] AS [Genre], 
    [Extent1].[AuthorId] AS [AuthorId], 
    [Extent2].[AuthorId] AS [AuthorId1], 
    [Extent2].[Name] AS [Name]
    FROM  [dbo].[Books] AS [Extent1]
    INNER JOIN [dbo].[Authors] AS [Extent2] ON [Extent1].[AuthorId] = [Extent2].[AuthorId]

```

**Lazy Loading（延迟加载）**
------------------

在延迟加载中，当实体的导航属性是非关联时，EF会自动加载一个相关的实体。为了使用延迟加载，使导航属性变成虚拟的。例如，在Book类中：

```
public class Book
{
    // (Other properties)

    // Virtual navigation property
    public virtual Author Author { get; set; }
}

```

现在考虑如下代码：

```
var books = db.Books.ToList();  // Does not load authors
var author = books[0].Author;   // Loads the author for books[0]

```
当延迟加载开启时，在books[0]上访问Author属性会使EF为author查询数据库。

延迟加载需要多段数据库操作过程，因为每次EF发送一个查询它都会取出一次相关实体。通常，你希望为序列化的对象禁用延迟加载。序列化已经在模型上读取了所有可能触发加载相关实体的属性。例如，下面是当延迟加载开启后EF序列化books列表时的SQL查询。你可以看到EF对于三个作者做了三次不同的查询。

```
SELECT 
    [Extent1].[BookId] AS [BookId], 
    [Extent1].[Title] AS [Title], 
    [Extent1].[Year] AS [Year], 
    [Extent1].[Price] AS [Price], 
    [Extent1].[Genre] AS [Genre], 
    [Extent1].[AuthorId] AS [AuthorId]
    FROM [dbo].[Books] AS [Extent1]

SELECT 
    [Extent1].[AuthorId] AS [AuthorId], 
    [Extent1].[Name] AS [Name]
    FROM [dbo].[Authors] AS [Extent1]
    WHERE [Extent1].[AuthorId] = @EntityKeyValue1

SELECT 
    [Extent1].[AuthorId] AS [AuthorId], 
    [Extent1].[Name] AS [Name]
    FROM [dbo].[Authors] AS [Extent1]
    WHERE [Extent1].[AuthorId] = @EntityKeyValue1

SELECT 
    [Extent1].[AuthorId] AS [AuthorId], 
    [Extent1].[Name] AS [Name]
    FROM [dbo].[Authors] AS [Extent1]
    WHERE [Extent1].[AuthorId] = @EntityKeyValue1

```
但还有很多时候你可能想要使用延迟加载。预加载会造成EF生成非常复杂的联接。或者你可能需要对于小的数据集合的相关实体，延迟加载会更加有效。

避免序列化问题的一种方式是序列化数据传输对象（DTOs）而不是实体对象。我将会在后面的文章中展示这种实现。

**显式加载（Explicit Loading）**
----------------------

显式加载和延迟加载非常类似，除了你在代码中显式地获取相关数据；当你访问导航属性时它不会自动发生。显示加载会在加载相关数据时给你更多的控制权，但也需要额外的代码。关于显示加载的更多信息，请查看Loading Related Entities。 http://msdn.microsoft.com/en-us/data/jj574232#explicit


**导航属性和环形引用（Navigation Properties and Circular References）**
---------------------------------------------
当我定义Book和Author模型时，我在Book类中为Book-Author关系定义了导航属性，但我没有在其他方向定义导航属性。

如果你在Author类中也定义相应的导航属性会怎样呢？

```
public class Author
{
    public int AuthorId { get; set; }
    [Required]
    public string Name { get; set; }

    public ICollection<Book> Books { get; set; }
}

```
不幸的是，当你在序列化模型时这会产生一个问题。如果你加载相关数据，它会产生环形对象图。

![这里写图片描述](http://img.blog.csdn.net/20160226173431736)

当JSON或XML格式试图序列化图时，它将会抛出一个异常。这两个格式抛出不同异常信息。这里是JSON格式的示例：

```
{
  "Message": "An error has occurred.",
  "ExceptionMessage": "The 'ObjectContent`1' type failed to serialize the response body for content type 
      'application/json; charset=utf-8'.",
  "ExceptionType": "System.InvalidOperationException",
  "StackTrace": null,
  "InnerException": {
    "Message": "An error has occurred.",
    "ExceptionMessage": "Self referencing loop detected with type 'BookService.Models.Book'. 
        Path '[0].Author.Books'.",
    "ExceptionType": "Newtonsoft.Json.JsonSerializationException",
    "StackTrace": "...”
     }
}

```

这里是XML格式的示例：

```
<Error>
  <Message>An error has occurred.</Message>
  <ExceptionMessage>The 'ObjectContent`1' type failed to serialize the response body for content type 
    'application/xml; charset=utf-8'.</ExceptionMessage>
  <ExceptionType>System.InvalidOperationException</ExceptionType>
  <StackTrace />
  <InnerException>
    <Message>An error has occurred.</Message>
    <ExceptionMessage>Object graph for type 'BookService.Models.Author' contains cycles and cannot be 
      serialized if reference tracking is disabled.</ExceptionMessage>
    <ExceptionType>System.Runtime.Serialization.SerializationException</ExceptionType>
    <StackTrace> ... </StackTrace>
  </InnerException>
</Error>

```
一个解决方案是使用DTO，我将会在下一节中描述它。你可以配置JSON或XML格式化程序来处理图循环。关于更多信息，请查看Handling Circular Object References. (http://www.asp.net/web-api/overview/formats-and-model-binding/json-and-xml-serialization#handling_circular_object_references)

对于本教程，你不需要Author.Book导航熟悉，所以你可以去掉它。






























