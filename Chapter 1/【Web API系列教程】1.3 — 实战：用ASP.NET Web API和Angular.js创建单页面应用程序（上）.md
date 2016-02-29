前言
==

在传统的web应用程序中，客户端（浏览器）通过请求页面来启动与服务器的通信。然后服务器处理该请求，并发送HTML页面到客户端。在随后页面上的操作中——例如，用户导航到一个链接或提交一个包含数据的表单——一个新的请求便被发送到服务器，并且重新开始了数据流：服务器处理请求，并将新页面发送到浏览器以响应客户端的新动作请求。

在单页面应用程序（SPA）中，在初始化请求后整个页面在浏览器中被加载出来，但通过Ajax请求来进行后续的交互。这意为着部分页面改变后浏览器仅需要更新，而不需要整个重新加载。SPA实现减少了应用程序对于用户动作的响应时间，从而制造了跟流畅的体验。

在这个动手实验中，你将使用这些先进技术的优势来实现Geek Quiz，这是一个基于SPA概念的网站。你将首先用ASP.NET Web API实现服务层以暴露所需的操作来检索问题和存储答案。然后，你将使用AngularJS和CSS3的变换效果来构建丰富多彩的UI。

回顾
==

**收获**
--

在本动手实验中，你将学习到：
1，	创建ASP.NET Web API服务来发送和接受JSON数据
2，	使用AngularJS来创建响应的UI 
3，	使用CSS3变化来加强UI体验

**前提工作**
----
以下是完成这个动手实验所必须的：
Visual Studio Express 2013 for Web 或更先进的版本

**安装**
--
为了在这个动手实验中运行这个项目，你需要先安装好开发环境：
1，	打开资源管理器并跳转到lab’s的Source文件夹。
2，	右击Setup.cmd并选择Run as administrator来运行安装程序，这会配置你的开发环境，并为本实验安装Visual Studio代码片（code snippet）。
3，	如果弹出了用户控制窗口，请确认该操作。
备注：在运行安装前请确保你已经检查了本实验所需的所以依赖项。

**使用代码片**
-----

在实验文档各处，你都被提示去插入代码块。为了更便利，所有这些代码都通过Visual Studio Code Snippets提供，这能够让你在Visual Studio 2013下读取这些代码，而不用手动来做。

备注：每个练习都伴随着该练习Begin文件夹下的开始解决方案，这能让你分别完成每个练习。你要知道这些在练习过程中加入的代码片可能在开始解决方案中被遗失或在你完成练习前无法工作。在每个练习的源码中，你也会发现一个包含了完成相应联系所需的剩余步骤的Visual Studio解决方案的End文件夹。如果你在完成这个动手实验过程中需要额外的帮助，你就可以将这些解决方案当作指导。

---

练习
==

该动手实验包含以下练习：
1，	Creating a Web API
2，	Creating a SPA Interface

预计完成本实验的时间：60分钟

备注：当你第一次启动Visual Studio时，你必须选择一个预定义的设置。每个预定义都被设计成匹配不同的开发风格，并且决定了窗体布局、编辑器行为、智能感知代码片和对话框选项。当使用General Development Settings集合时，实验过程中会给出动作描述以帮助在Visual Studio中完成给定的任务。如果你为你的开发环境选择了不同的设置，那么你该考虑到在接下来的步骤中可能会有差别。

**练习1：创建Web API**
-------------

服务层是SPA的一个关键部分，它能够处理来自UI发送的Ajax请求和在响应调用时返回数据。数据的返回应该是机器可读的格式，这样才能被客户端解析和使用。

Web API框架是ASP.NET栈的一部分，并被设计成更容易地实现HTTP服务，通常是通过RESTful API发送和接收JSON或XML格式的数据。在本练习中，你将创建一个Web站点用以托管Geek Quiz应用程序，然后使用ASP.NET Web API实现后台服务用以暴露和维持知识竞赛（quiz）的数据。

**任务1：为Geek Quiz创建初始项目**

在本任务中，你将基于Visual Studio的One ASP.NET项目类型来创建一个支持ASP.NET Web API的新的ASP.NET MVC项目。One ASP.NET合并了所有ASP.NET技术，并给予你随意搭配和选择的权利。你将添加Entity Framework的模板类和添加quiz问题的数据库。

1.	打开Visual Stuio Express 2013 for Web，并选择File | New Project... 来创建一个新解决方案。
![这里写图片描述](http://img.blog.csdn.net/20160225092800161)
2.	在New Project对话框，选择Visual C# | Web节点下的ASP.NET Web Application。请确保选择了.NET Framework，命名为GeekQuiz，选择一个Location并点击OK。
![这里写图片描述](http://img.blog.csdn.net/20160225092824031)
3.	在New ASP.NET Project对话框，选择MVC模板并选择Web API选项。同样请确保Authentication选项被设置到Individual User Account。点击OK以继续。
![这里写图片描述](http://img.blog.csdn.net/20160225092847786)
4.	在Solution Explorer，右击GeekQuiz项目的Models文件，并选择Add | Existing Item...
![这里写图片描述](http://img.blog.csdn.net/20160225092911641)


5.	在Add Existing Item对话框，导航到Source/Assets/Model文件夹，并选中所有文件。点击Add。


![这里写图片描述](http://img.blog.csdn.net/20160225092941438)

![这里写图片描述](http://img.blog.csdn.net/20160225093001380)
备注：通过添加这些文件，你其实就为Geek Quiz应用程序添加了数据模型、Entity Framework数据库上下文和数据库初始化类。

Entity Framework(EF)是一个对象关系映射，它使你能够通过对抽象化的应用程序模型编程来取代关系型存储编程来创建数据库访问。

以下是你刚刚添加的这些类的描述：

```
TriviaOption: 代表了对应于quiz问题的单个选项
TriviaQues：代表了一个quiz问题和通过Options属性暴露了该问题相关的选项
TriviaAnswer: 代表了对应于一个quiz问题的用户选择的选项
TriviaContext: 代表了Geek Quiz应用程序的Entity Framework数据库上下文。这个类从DContext继承而来并暴露了DbSet属性，后者表示以上所描述的实体对象集合。
TriviaDatabaseInitializer: 通过继承自CreateDatabaseIfNotExists为TriviaContext类实现了Entity Framework初始化功能。该类的默认行为是如果数据库不存在则创建它，以及在Seed方法内插入实体对象。
```

6 打开Global.asax.cs文件，并添加以下using语句。

```
using GeekQuiz.Models;
```
7 在Application_Start方法的开始添加以下代码，它将TriviaDatabaseInitializer设置成数据库初始器。

```
public class MvcApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        System.Data.Entity.Database.SetInitializer(new TriviaDatabaseInitializer()); 

        AreaRegistration.RegisterAllAreas();
        GlobalConfiguration.Configure(WebApiConfig.Register);
        FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
        RouteConfig.RegisterRoutes(RouteTable.Routes);
        BundleConfig.RegisterBundles(BundleTable.Bundles);
    }
}

```
8 修改Home控制器以约束对用户的认证。打开Controller文件夹下的HomeController.cs文件，并为HomeController类的定义添加Authorize属性。

```
namespace GeekQuiz.Controllers
{
    [Authorize]
    public class HomeController : Controller
    {
        public ActionResult Index()
        {
            return View();
        }

        ...
    }
}

```

备注：Authorize过滤器会检查用户是否是经过认证的。如果用户未认证，它会返回HTTP状态码401（Unauthorized）而不会执行用户操作。你可以将过滤器应用成全局的，也可以是控制器级别的，也可以是独立操作级别的。

9 现在你应该来修改web页面布局和绑定。打开Views | Shared文件夹下的_Layout.cshtml文件，通过将My ASP.NET Application修改成Geek Quiz来更新title元素的内容。

```
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@ViewBag.Title - Geek Quiz</title>
    @Styles.Render("~/Content/css")
    @Scripts.Render("~/bundles/modernizr")

</head>

```
10 在同一个文件中，通过移除About与Contact链接并重命名Home链接为Play来更新导航栏。另外，将Applicaiton name链接也修改成Geek Quiz。导航栏的HTML代码应该看起来是下面这样的：

```
<div class="navbar navbar-inverse navbar-fixed-top">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            @Html.ActionLink("Geek Quiz", "Index", "Home", null, new { @class = "navbar-brand" })
        </div>
        <div class="navbar-collapse collapse">
            <ul class="nav navbar-nav">
                <li>@Html.ActionLink("Play", "Index", "Home")</li>
            </ul>
            @Html.Partial("_LoginPartial")
        </div>
    </div>
</div>

```
11 通过将My ASP.NET Application替换成Geek Quiz以更新页面布局的footer节点。用以下高亮代码来修改<footer>元素的内容：

```
<div class="container body-content">
    @RenderBody()
    <hr />
    <footer>
        <p>&copy; @DateTime.Now.Year - Geek Quiz</p>
    </footer>
</div>

```
**任务2：创建名为TriviaController的Web API**

在前一个任务中，你已经创建了Geek Quiz这个web应用程序的初始结构。现在你要建立一个简单的Web API服务，用以和quiz的数据模型互动并暴露以下动作：
	GET /api/trivia: 从quiz列表取出已认证用户的下一个问题来让用户回答
	POST /api/trivia: 为已认证用户逐一存储quiz答案
你将使用Visual Studio提供的ASP.NET支架工具来为Web API控制器类创建基线。
1 打开App_Start文件夹下的WebApiConfig.cs文件。该文件定义了Web API服务的配置项，例如路由如何映射到Web API控制器的动作。
2 在文件首部添加如下using语句。

```
using Newtonsoft.Json.Serialization;
```
3 通过在Register方法内添加如下高亮代码，以全局性配置Web API动作方法返回的JSON数据格式。

```
public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        // Web API configuration and services

        // Use camel case for JSON data.
        config.Formatters.JsonFormatter.SerializerSettings.ContractResolver = new CamelCasePropertyNamesContractResolver();

        // Web API routes
        config.MapHttpAttributeRoutes();

        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "api/{controller}/{id}",
            defaults: new { id = RouteParameter.Optional }
        );
    }
}

```

备注: CamelCasePropertyNamesContractResolver会自动的将属性名称转换成camel式，它是JavaScript中属性的简便名称。
4 在Solution Explorer，右击GeekQuiz项目的Controller文件夹，选择Add | New Scaffolded Item...

![这里写图片描述](http://img.blog.csdn.net/20160225093246928)


5 在Add Scaffold对话框，请确保左侧面板的Common节点被选中。然后选择中间面板的Web API 2 Controller – Empty模板并点击Add。

![这里写图片描述](http://img.blog.csdn.net/20160225093335474)

备注：ASP.NET Scafolding是一个ASP.NET Web应用程序的代码生成框架。Visual Studio 2013对于MVC和Web API项目包含了预安装的代码生成器。当你想快速添加与数据模型交互的代码以减少开发标准的数据操作所需的时间，你就可以在项目中使用支架（scaffolding）。

支架（scaffolding）同样会确保项目中的依赖项都被安装。例如，如果你从一个空的ASP.NET项目开始，然后使用支架来添加了Web API控制器，与之相关的Web API NuGet包和引用会被自动添加你的项目中。

6 在Add Controller对话框，在Controller name文本框中键入TriviaController并点击Add。
![这里写图片描述](http://img.blog.csdn.net/20160225093404710)

7 之后TriviaController.cs文件就会被添加到GeekQuiz项目的Controller文件夹下，并包含一个空的TriviaController类。然后在文件开头添加以下using语句。

```
using System.Data.Entity;
using System.Threading;
using System.Threading.Tasks;
using System.Web.Http.Description;
using GeekQuiz.Models;

```
8 在TriviaController类开头添加以下代码，在控制器中定义、初始化和配置TriviaContext实例。

```
public class TriviaController : ApiController
{
    private TriviaContext db = new TriviaContext();

    protected override void Dispose(bool disposing)
    {
        if (disposing)
        {
            this.db.Dispose();
        }

        base.Dispose(disposing);
    }
}

```

备注：TriviaController的Dispose方法执行TriviaContext实例的Dispose方法，它会确保当TriviaContext实例被创建或垃圾回收时，所有被上下文对象使用的资源都被释放掉。这也包括了被Entity Framework打开的数据库连接。

9 在TrivialController类的结尾加上以下helper方法。这个方法根据具体用户从数据库中取出quiz问题给用户回答。

```
private async Task<TriviaQuestion> NextQuestionAsync(string userId)
{
    var lastQuestionId = await this.db.TriviaAnswers
        .Where(a => a.UserId == userId)
        .GroupBy(a => a.QuestionId)
        .Select(g => new { QuestionId = g.Key, Count = g.Count() })
        .OrderByDescending(q => new { q.Count, QuestionId = q.QuestionId })
        .Select(q => q.QuestionId)
        .FirstOrDefaultAsync();

    var questionsCount = await this.db.TriviaQuestions.CountAsync();

    var nextQuestionId = (lastQuestionId % questionsCount) + 1;
    return await this.db.TriviaQuestions.FindAsync(CancellationToken.None, nextQuestionId);
}

```
10 在TrivialController类中添加以下Get动作的方法。这个动作的方法调用了在上一步中定义的用于为认证用户取出下一个问题的NextQuestionAsync方法。

```
// GET api/Trivia
[ResponseType(typeof(TriviaQuestion))]
public async Task<IHttpActionResult> Get()
{
    var userId = User.Identity.Name;

    TriviaQuestion nextQuestion = await this.NextQuestionAsync(userId);

    if (nextQuestion == null)
    {
        return this.NotFound();
    }

    return this.Ok(nextQuestion);
}

```
11 在TriviaController类中添加以下helper方法。

```
private async Task<bool> StoreAsync(TriviaAnswer answer)
{
    this.db.TriviaAnswers.Add(answer);

    await this.db.SaveChangesAsync();
    var selectedOption = await this.db.TriviaOptions.FirstOrDefaultAsync(o => o.Id == answer.OptionId
        && o.QuestionId == answer.QuestionId);

    return selectedOption.IsCorrect;
}

```
12 在TriviaController类中添加以下Post动作方法。这个动作方法将答案和认证用户联系起来，并调用StoreAsync方法。然后，它会根据帮助方法的返回值发送一个包含布尔变量的响应。

```
// POST api/Trivia
[ResponseType(typeof(TriviaAnswer))]
public async Task<IHttpActionResult> Post(TriviaAnswer answer)
{
    if (!ModelState.IsValid)
    {
        return this.BadRequest(this.ModelState);
    }

    answer.UserId = User.Identity.Name;

    var isCorrect = await this.StoreAsync(answer);
    return this.Ok<bool>(isCorrect);
}

```
13 通过向TriviaContorller的类定义添加Authorize属性，来修改Web API控制器以约束对认证用户的访问。

```
[Authorize]
public class TriviaController : ApiController
{
    ...
}

```
**任务3：运行解决方案**

在本任务中，你将核实你在上一个任务重建立的Web API服务是否如预期般工作。你将会使用Internet Explorer的F12 Developer Tools来捕获网络数据传输和检测来着Web API服务的全部响应。

备注：确保Visual Studio工具栏的Start按钮上被选中为Internet Explorer。

1 按F5以运行解决方案。Log in页面应该会出现在浏览器上。
备注：当应用程序启动时，默认的MVC路由被触发，它会默认地映射到HomeController类的Index动作上。因为HomeController被约束为认证用户（在练习1中你已经将Authorize属性添加到该类上），并且现在还没有用户经过认证，所以应用程序会向登录页面发送原始请求。
![这里写图片描述](http://img.blog.csdn.net/20160225093546555)

2 点击Register来创建一个新用户。
![这里写图片描述](http://img.blog.csdn.net/20160225093603571)

3 在Register页面，输入一个用户名和密码，然后点击Register。

![这里写图片描述](http://img.blog.csdn.net/20160225093621665)
4 接下来应用程序就会注册一个新账号，并且该账号是经过认证的，随后会自动导航到主页。

![这里写图片描述](http://img.blog.csdn.net/20160225093641899)
5 在浏览器上按F12以打开Developer Tools面板。按Ctrl+4或点击Network图标，然后点击绿色箭头按钮以开始捕获网络传输。

![这里写图片描述](http://img.blog.csdn.net/20160225093704571)

6 在浏览器地址栏的URL上添加api/trivia。你将会检测到来自TriviaController里的Get动作的响应细节。
![这里写图片描述](http://img.blog.csdn.net/20160225093731308)

备注：一旦下载完成，你将会被提示对下载的文件进行操作。留着这个对话框，为了在Developers Tool窗口查看响应的内容。
7 现在你可以检测到响应体了。点击Details，然后点击Response body。你能发现下载的数据是一个对应着TriviaQuestion类的options(它是一个TriviaOption对象的列表)、id和title属性的对象。
![这里写图片描述](http://img.blog.csdn.net/20160225093753133)


8 返回到Visual Studio并按Shift+F5以停止调试。


















