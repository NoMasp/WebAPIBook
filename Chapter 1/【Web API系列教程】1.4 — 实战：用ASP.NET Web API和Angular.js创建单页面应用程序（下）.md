**练习2：创建SPA界面**
-----------

在本练习中，你将首先创建Geek Quiz的web前端，使用AngularJS专注于单页面应用程序的交互。然后你将使用CSS3来执行丰富的动画和提供一个当从一个问题转换到另一个问题时切换上下文的可视化效果以加强用户体验。

**任务1：使用AngularJS来创建SPA界面**

在本任务中，你将使用AngularJS来实现Geek Quiz应用程序的客户端。AngularJS是一个开源的JavaScript框架，它能够搭配MVC以加强基于浏览器的应用程序，使其对于开发和测试都更加便利。

你将从在Visual Studio的Package Manager Console来安装AngularJS开始。然后，你将创建一个控制器用以提供Geek Quiz应用的行为和使用AngularJS模板引擎来提交quiz问题和答案的视图。

备注：关于AngularJS的更多信息，请查看http://angularjs.org/。
1.	打开Visual Studio Express 2013 for Web并打开位于Source/Ext2-CreatingASPAInterface/Begin文件夹下的GeekQuiz.sln解决方案。或者你也可以继续上一个练习所保留的解决方案。
2.	打开Tools | Library Package Manager下的Package Manager Console，键入以下命令以安装AngularJS.Core的NuGet包。
Install-Package AngularJS.Core
3.	在Solution Explorer，右击GeekQuiz项目下的Scripts文件夹，并选择Add | New Folder。为文件夹命名为app并按Enter。
4.	右击你刚刚创建的app文件夹，并选择Add | JavaScript File。

![这里写图片描述](http://img.blog.csdn.net/20160225094115541)

5.	在Specify Name for Item对话框，在Iten name文本框中键入quiz-controller并点击OK。

![这里写图片描述](http://img.blog.csdn.net/20160225094134688)

6.	在quiz-controller.js文件中，添加以下代码以声明和初始化AngularJS的QuizCtrl控制器。

```
angular.module('QuizApp', [])
    .controller('QuizCtrl', function ($scope, $http) {
        $scope.answered = false;
        $scope.title = "loading question...";
        $scope.options = [];
        $scope.correctAnswer = false;
        $scope.working = false;

        $scope.answer = function () {
            return $scope.correctAnswer ? 'correct' : 'incorrect';
        };
    });

```

备注：QuizCtrl的构造函数期望的参数命名为$scope。scope的初始状态应该是在构造函数中被设置，通过将属性添加到$scope对象。该属性包含了视图模型，并且当控制器被注册时可以访问到模板。

QuizCtrl控制器在一个名为QuizApp的模块内定义。模块是让你能够将应用程序分成独立组件的工作单元。使用模块的主要优势是代码更加易于理解、方便单元测试、可重复使用和可维护。
7.	现在你将给scope添加行为以响应来自视图的事件触发。添加以下代码到QuizCtrl控制器的底部，它在$scope对象中定义了nextQuestion函数。

```
.controller('QuizCtrl', function ($scope, $http) { 
    ...

    $scope.nextQuestion = function () {
        $scope.working = true;
        $scope.answered = false;
        $scope.title = "loading question...";
        $scope.options = [];

        $http.get("/api/trivia").success(function (data, status, headers, config) {
            $scope.options = data.options;
            $scope.title = data.title;
            $scope.answered = false;
            $scope.working = false;
        }).error(function (data, status, headers, config) {
            $scope.title = "Oops... something went wrong";
            $scope.working = false;
        });
    };
};

```

备注：该函数从上一个练习中创建的Web API Trivia中取出下一个问题，并将问题的数据附加到$scope对象上。
8.	添加以下代码到QuizCtrl控制器的底部，它在$scope对象中定义了sendAnswer函数。

```
.controller('QuizCtrl', function ($scope, $http) { 
    ...

    $scope.sendAnswer = function (option) {
        $scope.working = true;
        $scope.answered = true;

        $http.post('/api/trivia', { 'questionId': option.questionId, 'optionId': option.id }).success(function (data, status, headers, config) {
            $scope.correctAnswer = (data === true);
            $scope.working = false;
        }).error(function (data, status, headers, config) {
            $scope.title = "Oops... something went wrong";
            $scope.working = false;
        });
    };
};

```

备注：这个函数发送了根据用户选择的答案到Trivia Web API，并存储该结果——例如，该结果是否正确——到$scope对象中。

9.	下一步是创建AngularJS模板用以定义quiz的视图。打开Views | Home文件夹下的Index.cshtml文件，并替换文件内容为以下代码：


```
@{
    ViewBag.Title = "Play";
}

<div id="bodyContainer" ng-app="QuizApp">
    <section id="content">
        <div class="container" >
            <div class="row">
                <div class="flip-container text-center col-md-12" ng-controller="QuizCtrl" ng-init="nextQuestion()">
                    <div class="back" ng-class="{flip: answered, correct: correctAnswer, incorrect:!correctAnswer}">
                        <p class="lead">{{answer()}}</p>
                        <p>
                            <button class="btn btn-info btn-lg next option" ng-click="nextQuestion()" ng-disabled="working">Next Question</button>
                        </p>
                    </div>
                    <div class="front" ng-class="{flip: answered}">
                        <p class="lead">{{title}}</p>
                        <div class="row text-center">
                            <button class="btn btn-info btn-lg option" ng-repeat="option in options" ng-click="sendAnswer(option)" ng-disabled="working">{{option.title}}</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</div>

@section scripts {
    @Scripts.Render("~/Scripts/angular.js")
    @Scripts.Render("~/Scripts/app/quiz-controller.js")
}

```
备注：AngularJS模板是声明性的规范，使用模型和控制器的信息将静态标记在用户可见的浏览器中转换成动态视图。以下是模板中用到的AngularJS元素和元素属性的示例：
1，	ng-app指令告诉AngularJS表示应用程序根元素的DOM元素
2，	ng-controller指令在指令声明的位置将控制器添加到DOM上
3，	花括号标记{{}}表明绑定到在控制器中定义的scope属性
4，	ng-click指令被用于响应用户点击并执行定义在scope中的函数

10.	打开Content文件夹下的Site.css文件，并添加以下高亮的样式到文件底部，它提供了一个不错的quiz视图。


```
.validation-summary-valid {
     display: none;
}

/* Geek Quiz styles */
.flip-container .back,
.flip-container .front {
     border: 5px solid #00bcf2;
     padding-bottom: 30px;
     padding-top: 30px;
}

#content {
    position:relative;
    background:#fff;
    padding:50px 0 0 0;
}

.option {
     width:140px;
     margin: 5px;
}

div.correct p {
     color: green;
}

div.incorrect p {
     color: red;
}

.btn {
     border-radius: 0;
}

.flip-container div.front, .flip-container div.back.flip {
    display: block;
}

.flip-container div.front.flip, .flip-container div.back {
    display: none;
}

```
**任务2：运行解决方案**

在本任务中，你将执行你用AngularJS创建的新的用户界面来回答一些quiz问题。

1 	按F5来运行解决方案
2 注册一个新账户。重做练习1中任务3的注册步骤。
备注：如果你使用的是上一个练习所保留的解决方案，那你就可以使用此前注册的账户。
3 Home页面应该会出现，并显示第一个quiz问题。通过点击其中一个选项以回答问题。这将会触发此前定义的sendAnswer函数，它会发送选中的选项到Trivia Web API。

![这里写图片描述](http://img.blog.csdn.net/20160225094252940)


4 再点击其中一个按钮之后，结果就会出现。点击Next Question来显示接下来的问题。这将会触发在控制器中定义的nextQuestion函数。

![这里写图片描述](http://img.blog.csdn.net/20160225094330979)

5 下一个问题应该会出现。尽可能地继续多回答几个问题。在完成所有的问题后，你应该会返回到第一个问题。
![这里写图片描述](http://img.blog.csdn.net/20160225094350863)

6 返回Visual Studio并按Shift+F5来停止调试。
**任务3：使用CSS3创建翻转动画**

在本任务重，你将使用CSS3执行丰富的动画，通过在一个问题被回答和下一个问题被提取出来时添加一个翻转效果。

1 在Solution Explorer中，右击GeekQuiz项目下的Content文件夹，并选择Add | Existing Item。

![这里写图片描述](http://img.blog.csdn.net/20160225094427698)

2 在Add Existing Item对话框，导航到Source/Assets文件夹，并选择Flip.css。点击OK。

![这里写图片描述](http://img.blog.csdn.net/20160225094446527)

3 打开你刚刚添加的Flip.css并检查其内容。
4 跳转到flip transformation注释，以下样式使用CSS的perspective和rotateY变换来生成了一个“卡片翻转”效果。

```
/* flip transformation */
.flip-container div.front {
    -moz-transform: perspective(2000px) rotateY(0deg);
    -webkit-transform: perspective(2000px) rotateY(0deg);
    -o-transform: perspective(2000px) rotateY(0deg);
    transform: perspective(2000px) rotateY(0deg);
}

    .flip-container div.front.flip {
        -moz-transform: perspective(2000px) rotateY(179.9deg);
        -webkit-transform: perspective(2000px) rotateY(179.9deg);
        -o-transform: perspective(2000px) rotateY(179.9deg);
        transform: perspective(2000px) rotateY(179.9deg);
    }

.flip-container div.back {
    -moz-transform: perspective(2000px) rotateY(-180deg);
    -webkit-transform: perspective(2000px) rotateY(-180deg);
    -o-transform: perspective(2000px) rotateY(-180deg);
    transform: perspective(2000px) rotateY(-180deg);
}

    .flip-container div.back.flip {
        -moz-transform: perspective(2000px) rotateY(0deg);
        -webkit-transform: perspective(2000px) rotateY(0deg);
        -ms-transform: perspective(2000px) rotateY(0);
        -o-transform: perspective(2000px) rotateY(0);
        transform: perspective(2000px) rotateY(0);
    }

```
5 跳转到hide back of pane during flip注释。

```
/* hide back of pane during flip */
.front, .back {
    -moz-backface-visibility: hidden;
    -webkit-backface-visibility: hidden;
    backface-visibility: hidden;
}

```
6 打开App_Start文件夹下的BundleConfig.cs文件，并添加引用到Flip.css。

```
bundles.Add(new StyleBundle("~/Content/css").Include(
    "~/Content/bootstrap.css",
    "~/Content/site.css",
    "~/Content/Flip.css"));

```
7 按F5以运行解决方案并登录。
8 通过点击任一选项来回答问题。注意两个视图间的翻转效果。

![这里写图片描述](http://img.blog.csdn.net/20160225094715246)

9 点击Next Question以提取下一个问题。翻转效果也会再次出现。

![这里写图片描述](http://img.blog.csdn.net/20160225094804184)

总结
==

通过完成这个动手实验室，你已经学会了：
1，	使用ASP.NET Scaffolding来创建ASP.NET Web API控制器
2，	实现Web API的Get动作以提取下一个quiz问题
3，	实现Web API的Post动作以存储quiz答案
4，	在Visual Studio的Package Manger Console安装AngularJS
5，	实现AngularJS模板和控制器
6，	使用CSS3变换来执行动作效果


































































































































































