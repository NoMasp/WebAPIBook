在这最后一节中，你将把应用程序发布到Azure。在Solution Explorer中，右击项目并选择Publish。

![这里写图片描述](http://img.blog.csdn.net/20160227092521655)


点击Publish打开Publish Web对话框。如果你在新建项目的时候选中了Host in Cloud，那么链接和设置就已经都配置好了。在这种情况下，仅仅是需要点击Settings面板，然后选择“Execute Code First Migrations”。（如果你没有在开始的时候选中”Host in Cloud”，那么请跟随如下步骤）（http://www.asp.net/web-api/overview/data/using-web-api-with-entity-framework/part-10#new-website）

![这里写图片描述](http://img.blog.csdn.net/20160227092547984)

为了部署应用，点击Publish。你可以在Web Publish Activity窗口查看发布过程。（在View目录下，选择Other Windows，然后选择Web Publish Activity。）

![这里写图片描述](http://img.blog.csdn.net/20160227092622063)

当Visual Studio完成app的部署，默认的浏览器会自动打开部署网站的URL，你所创建的应用程序现在就运行在云端了。浏览器地址栏中的URL显示了站点以及被从网络中加载出来了。

![这里写图片描述](http://img.blog.csdn.net/20160227092640210)

**部署到新站点**
------

如果你没有在建立项目的时候选中Host in Cloud，那么你现在需要配置一个新的web应用。在Solution Explorer，右击项目，并选择Publish。选择Profile面板，并点击Microsoft Azure Websites。如果你现在没有登录到Azure，你需要先去登陆。

![这里写图片描述](http://img.blog.csdn.net/20160227092710985)

在Existing Websites对话框，点击New。

![这里写图片描述](http://img.blog.csdn.net/20160227092728533)

输入站点名称。选择你的Azure订阅和地区。在Database server下，选择Create New Server，或已经存在的server。点击Create。

![这里写图片描述](http://img.blog.csdn.net/20160227092745095)

点击Settings面板，然后选择“Execute Code First Migrations”。然后点击Publish。

























































