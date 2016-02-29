现在，我们的Web API暴露数据库实体给客户端，而客户端接收直接映射到你的数据库表的数据。然而，这不永远都是个好办法。有时候你可以想要改变发送到客户端的数据的形式。例如，你可以想要：
1，	移除环形引用（见上一章）
2，	隐藏客户端不应该看到的特定属性
3，	为了减少有效载荷而省略一些属性
4，	拼接包含嵌套的对象图，以使它们对客户端更便利
5，	避免”over-posting”漏洞（查看Model Validation(http://www.asp.net/web-api/overview/formats-and-model-binding/model-validation-in-aspnet-web-api)关于over-posting的讨论）
6，	对你的服务层和数据层进行解耦

为了完成它，你应该定义一个数据传输对象(DTO，data transfer object)。DTO是一个定义了数据将会如何在网络上传输的对象。让我们来看看它如何工作于Book实体。在Models文件夹下，添加两个DTO类：

```
namespace BookService.Models
{
    public class BookDTO
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string AuthorName { get; set; }
    }
}

namespace BookService.Models
{
    public class BookDetailDTO
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public int Year { get; set; }
        public decimal Price { get; set; }
        public string AuthorName { get; set; }
        public string Genre { get; set; }
    }
}

```

BookDetailDTO类包含了Book模型的所有属性，除了用于承载作者姓名的字符串AuthorName。BookDTO类包含了BookDetailDTO的属性的子集。

下一步，在BooksController类中替换两个返回DTO的GET方法。我们将使用LINQ的Select语句将Book实体转换到DTO。

```
// GET api/Books
public IQueryable<BookDTO> GetBooks()
{
    var books = from b in db.Books
                select new BookDTO()
                {
                    Id = b.Id,
                    Title = b.Title,
                    AuthorName = b.Author.Name
                };

    return books;
}

// GET api/Books/5
[ResponseType(typeof(BookDetailDTO))]
public async Task<IHttpActionResult> GetBook(int id)
{
    var book = await db.Books.Include(b => b.Author).Select(b =>
        new BookDetailDTO()
        {
            Id = b.Id,
            Title = b.Title,
            Year = b.Year,
            Price = b.Price,
            AuthorName = b.Author.Name,
            Genre = b.Genre
        }).SingleOrDefaultAsync(b => b.Id == id);
    if (book == null)
    {
        return NotFound();
    }

    return Ok(book);
} 

```
这是新的GetBooks方法所生成的SQL查询。你可以查看EF从LINQ Select转换到SQL Select的语句。

```
SELECT 
    [Extent1].[Id] AS [Id], 
    [Extent1].[Title] AS [Title], 
    [Extent2].[Name] AS [Name]
    FROM  [dbo].[Books] AS [Extent1]
    INNER JOIN [dbo].[Authors] AS [Extent2] ON [Extent1].[AuthorId] = [Extent2].[Id]

```
最后，修改PostBook方法以返回DTO。

```
[ResponseType(typeof(Book))]
public async Task<IHttpActionResult> PostBook(Book book)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    db.Books.Add(book);
    await db.SaveChangesAsync();

    // New code:
    // Load author name
    db.Entry(book).Reference(x => x.Author).Load();

    var dto = new BookDTO()
    {
        Id = book.Id,
        Title = book.Title,
        AuthorName = book.Author.Name
    };

    return CreatedAtRoute("DefaultApi", new { id = book.Id }, dto);
}
```


总结：在这个教程中，我们在代码中手动地转换到DTO。另一种方式是使用像AutoMapper这样的库来处理自动转换。






























