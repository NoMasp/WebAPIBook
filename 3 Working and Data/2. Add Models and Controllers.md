**前言**
--

在本部分中，你将添加用于定义数据库实体的模型类。然后你将添加用于在这些实体上执行CRUD（Create、Retrieve、Update、Delete——译者注）操作的Web API 控制器。

**添加模型类**
-----

在本教程中，我们将通过使用“Code First”的方法对实体框架（EF）来创建数据库。对于Code First，你写C#类来相应数据库表，使用EF来创建数据库。（有关详细信息，见Entity Framework Development Approaches.）

首先，我们定义我们的域对象作为POCO。我们将创建以下POCO：
Author
Book

在解决方案资源管理中，右击Models文件夹。选择Add，然后选择Class。名为这个类为Author。

![这里写图片描述](http://img.blog.csdn.net/20160226170228959)

用以下代码替换Author.cs中的所有样板代码。

```
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace BookService.Models
{
    public class Author
    {
        public int Id { get; set; }
        [Required]
        public string Name { get; set; }
    }
}

```
添加另一个命名为Book的类，并替换成以下代码。

```
using System.ComponentModel.DataAnnotations;

namespace BookService.Models
{
    public class Book
    {
        public int Id { get; set; }
        [Required]
        public string Title { get; set; }
        public int Year { get; set; }
        public decimal Price { get; set; }
        public string Genre { get; set; }

        // Foreign Key
        public int AuthorId { get; set; }
        // Navigation property
        public Author Author { get; set; }
    }
}
```

Entity Framework将使用这些模型来创建数据库表。对于每一个模型，Id属性将变成数据库表的主键列。


在Book类中，AuthorId定义了一个外键到Author表。（简单起见，我假定每本书只有一个作者。）该book类还包含一个导航属性给相关的Author。 你可以使用导航属性在代码中访问相关的作者。我在<a href="http://blog.csdn.net/nomasp/article/details/50751577 " target="_blank">【Web API系列教程】3.4 — 实战：处理数据（处理实体关系）</a> 
中描述了关于导航属性的更多信息。

**添加Web API 控制器**
-------------
在这部分，我们将添加支持CRUD（create, read, update 和 delete）的Web API 控制器。这些控制器使用Entity Framework来同数据库层交流。

首先，你应该删除Controllers目录下的ValuesControllers.cs文件。这个文件包含了一个Web API示例，但对于本教程你并不需要它。

![这里写图片描述](http://img.blog.csdn.net/20160226170356430)

然后，编译这个项目。Web API框架使用反射来发现这个模型类，所以它需要编译程序集。

在Solution Explorer中，右击Controllers文件夹。选择Add，然后选择Controller。

![这里写图片描述](http://img.blog.csdn.net/20160226170624773)

在Add Scaffold对话框中，选择“Web API 2 controller with actions, using Entity Framework”。点击Add。

![这里写图片描述](http://img.blog.csdn.net/20160226170651508)


在Add Controller对话框中，执行以下操作：
1，	在模型类下拉框中，选择Author类。（如果你没有在下拉框中看到它，请确保已经编译了这个项目。）
2，	选中“Use async controller action”。
3，	保留控制器名称为“AuthorsController”。
4，	点击加号（+）按钮下一步到Data Context Class.





![这里写图片描述](http://img.blog.csdn.net/20160226170723492)

在New Data Context对话框中，保留默认名称并点击Add。

![这里写图片描述](http://img.blog.csdn.net/20160226170739341)

点击Add以完成Add Controller对话框。这个对话将添加两个类到你的项目中：
AuthorsController定义了一个Web API控制器。这个控制器实现了REST API，客户端使用它来在authors列表上执行CRUD操作。
BookServiceContext在运行时管理实体对象，包括从数据库中聚集对象数据、追踪、保留数据到数据库。它继承自DBContext。

![这里写图片描述](http://img.blog.csdn.net/20160226170832749)


在这个节点上，再次编译这个项目。现在再过一遍相同的步骤为Book实体添加API控制器。这次选择Book作为模型类，并选择已经存在的BookServiceContext类作为数据上下文类。（不要再创建新的数据上下文。）点击Add以添加该控制器。

![这里写图片描述](http://img.blog.csdn.net/20160226170857743)
























































