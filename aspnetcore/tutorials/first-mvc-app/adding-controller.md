
# Adding a controller to a ASP.NET Core MVC app with Visual Studio  
在 Visual Studio 中给 ASP.NET Core MVC 应用程序添加控制器

By [Rick Anderson](https://twitter.com/RickAndMSFT)

[!INCLUDE[adding-controller1](../../includes/mvc-intro/adding-controller1.md)]

* In **Solution Explorer**, right-click **Controllers > Add > New Item**  
在 **项目资源管理器** 中， 鼠标右键点击 **Controllers > Add(添加) > New Item(新建项)**  

![Contextual menu](adding-controller/_static/add_controller.png)

* Select **MVC Controller Class**  
选择 **MVC 控制器类**
* In the **Add New Item** dialog, enter **HelloWorldController**.  
在 **添加新建项** 对话框，输入 **HelloWorldController** 。

![Add MVC controller and name it](adding-controller/_static/ac.png)

[!INCLUDE[adding-controller2](../../includes/mvc-intro/adding-controller2.md)]

In Visual Studio, in non-debug mode (Ctrl+F5), you don't need to build the app after changing  code. Just save the file, refresh your browser and you can see the changes.
在 Visual Studio, 使用 **非调试模式** (Ctrl+F5), 你不用在修改代码后生成应用程序，只需要保存文件，刷新浏览器后就可以看到修改的内容了。
>[!div class="step-by-step"]
[Previous](start-mvc.md)
[Next](adding-view.md)  
