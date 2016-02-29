**前言**
--

本指南将会教你使用ASP.NET Web API作后端创建web应用程序的基本技能。本指南使用Entity Framework 6作为数据层，使用knockout.js作为客户端的JavaScript应用程序。本指南也会展示部署应用到Azure App service Web Apps。

本指南使用搭配Entity Framework 6的ASP.NET Web API 2来创建一个操作后端数据库的web应用程序。这是一个你将创建的应用程序截图。

![这里写图片描述](http://img.blog.csdn.net/20160226165424065)

这个应用使用single-page application (SPA) 设计。“Single-page application”是一个通过加载HTML页面然后动态更新页面以取代加载新页面的web应用程序的统称。在初始化页面加载后，应用通过AJAX请求和服务器交流。应用通过AJAX请求返回的JSON数据来更新UI。

AJAX不新颖，但今天这里使用了JavaScript框架，它使得建立一个大而精密的SPA应用程序更加容易。本教程使用了Knockout.js，但你可以使用任何JavaScript客户端框架。

以下是这个应用程序的主要构造块：
1，	ASP.NET MVC 创建HTML页面。
2，	ASP.NET Web API 处理AJAX请求并返回JSON数据。
3，	Knockout.js (数据)绑定HTML元素到JSON数据。
4，	Entity Framework 和数据库交流。

**创建项目**
----

打开Visual Studio。在File目录下，选择New，然后选择Project。（或在开始页面点击New Project。）

在New Project对话框中，点击左面板的Web和中间面板的ASP.NET Web Application。给项目命名为BookService并点击OK。

![这里写图片描述](http://img.blog.csdn.net/20160226165543340)

在New ASP.NET Project对话框中，选择Web API容器。

![这里写图片描述](http://img.blog.csdn.net/20160226165607513)

如果你希望将项目托管在Azure App Service，请使Host in the cloud被选中。

**配置Azure设置（可选）**
-------------

如果你保留Host in cloud选项被选中，Visual Studio就会指引你去登陆Microsoft Azure。

![这里写图片描述](http://img.blog.csdn.net/20160226165809360)

在你登录到Azure后，Visual Studio还会让你去配置web应用。为站点输入名称，选择你的Azure订阅，并选择国家和地区。在Database server下，选择Create new server。输入管理员用户名和密码。

![这里写图片描述](http://img.blog.csdn.net/20160226170004192)

















































