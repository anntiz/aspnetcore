
# Adding a model to an ASP.NET Core MVC app  
添加模型到 ASP.NET Core MVC 应用程序


[!INCLUDE[adding-model](../../includes/mvc-intro/adding-model1.md)]

In Solution Explorer, right click the **MvcMovie** project > **Add** > **New Folder**. Name the folder *Models*.  
在 **Solution Explorer(项目资源管理器)** 中，鼠标右键单击 **MvcMovie** 项目，选择 **Add(添加)** > **New Folder(新建文件夹)**。将文件夹命名为 *Models*。

Right click the *Models* folder > **Add** > **Class**. Name the class **Movie** and add the following properties:  
鼠标右键单击 *Models* 文件夹，然后选择 **Add(添加)** > **Class(类)**。将该类命名为 **Movie** 并添加以下属性：
```C#
//#define MovieNoEF
#if MovieNoEF
#region snippet_1
using System;

namespace MvcMovie.Models
{
    public class Movie
    {
        public int ID { get; set; }
        public string Title { get; set; }
        public DateTime ReleaseDate { get; set; }
        public string Genre { get; set; }
        public decimal Price { get; set; }
    }
}
#endregion
#endif
```

[!code-csharp[Main](../../tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Models/MovieNoEF.cs?name=snippet_1)]

The `ID` field is required by the database for the primary key.  
其中，`ID` 字段是数据库必需的主键字段

Build the project to verify you don't have any errors. You now have a **M**odel in your **M**VC app.  
构建这个项目来验证你没有出现任何错误。 现在你的 **MVC** 应用程序中已经得到一个 **模型** 了。

## Scaffolding a controller  
使用基架创建控制器

In **Solution Explorer**, right-click the *Controllers* folder **> Add > Controller**.  
在 **Solution Explorer(项目资源管理器)** 中, 鼠标右键单击 *Controllers* 文件夹，然后选择 **> Add(添加) > Controller(控制器)**

![view of above step](adding-model/_static/add_controller.png)

In the **Add MVC Dependencies** dialog, select **Minimal Dependencies**, and select **Add**.  
在 **Add MVC Dependencies(添加MVC依赖关系)** 对话框中, 选择 **Minimal Dependencies(最小依赖关系)**, 然后选择 **Add(添加)**

![view of above step](adding-model/_static/add_depend.png)

Visual Studio adds the dependencies needed to scaffold a controller, but the controller itself is not created. The next invoke of **> Add > Controller** creates the controller.  
Visual Studio 添加一些必需的依赖关系到基架控制器中，但是这个控制器还没有被创建。下一次再执行 **> Add(添加) > Controller(控制器)**的时候才会创建这些控制器。

In **Solution Explorer**, right-click the *Controllers* folder **> Add > Controller**.  
在 **Solution Explorer(项目资源管理器)** 中, 鼠标右键单击 *Controllers* 文件夹，然后选择 **> Add(添加) > Controller(控制器)**

![view of above step](adding-model/_static/add_controller.png)

In the **Add Scaffold** dialog, tap **MVC Controller with views, using Entity Framework > Add**.  
在 **Add Scaffold（新建基架）** 对话框中, 单击 **MVC Controller with views, using Entity Framework > Add（添加）**
![Add Scaffold dialog](adding-model/_static/add_scaffold2.png)

Complete the **Add Controller** dialog:  
完成这个 **Add Controller(添加控制器)** 对话框

* **Model class:** *Movie (MvcMovie.Models)*
* **Data context class:** Select the **+** icon and add the default **MvcMovie.Models.MvcMovieContext**  
**Data context class:** 选择 **+** 按钮并添加默认的 **MvcMovie.Models.MvcMovieContext**  

![Add Data context](adding-model/_static/dc.png)

* **Views:** Keep the default of each option checked  
**Views:** 保留所有选项的默认值  
* **Controller name:** Keep the default *MoviesController*  
**Controller name:** 保留默认的名字 *MoviesController*  
* Tap **Add**  
单击 **Add(添加)**

![Add Controller dialog](adding-model/_static/add_controller2.png)

Visual Studio creates:  
Visual Studio 创建下面的内容：

* An Entity Framework Core [database context class](xref:data/ef-mvc/intro#create-the-database-context) (*Data/MvcMovieContext.cs*)  
一个Entity Framework Core 数据库上下文类
```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

namespace MvcMovie.Models
{
    public class MvcMovieContext : DbContext
    {
        public MvcMovieContext (DbContextOptions<MvcMovieContext> options)
            : base(options)
        {
        }

        public DbSet<MvcMovie.Models.Movie> Movie { get; set; }
    }
}
```

* A movies controller (*Controllers/MoviesController.cs*)  
一个 movies 控制器
* Razor view files for Create, Delete, Details, Edit and Index pages (*Views/Movies/\*.cshtml*)  
包含 Create(增加), Delete(删除), Details(详细), Edit(编辑) and Index(索引) 等页面 (*Views/Movies/\*.cshtml*) 的Razor 视图文件

The automatic creation of the database context and [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) (create, read, update, and delete) action methods and views is known as *scaffolding*. You'll soon have a fully functional web application that lets you manage a movie database.  
我们把这个自动创建数据库上下文和CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) (create, read, update, and delete)操作方法及视图的称为 *scaffolding(基架)*。你可以很快就得到一个功能完整的Web应用程序用于管理你的电影数据库。

If you run the app and click on the **Mvc Movie** link, you'll get an error similar to the following:  
如果运行这个应用程序并点击上面的 **Mvc Movie** 链接，将会得到类似下面的错误信息：
```
An unhandled exception occurred while processing the request.
SqlException: Cannot open database "MvcMovieContext-<GUID removed>" 
requested by the login. The login failed.
Login failed for user Rick
```

You need to create the database, and you'll use the EF Core [Migrations](xref:data/ef-mvc/migrations) feature to do that. Migrations lets you create a database that matches your data model and update the database schema when your data model changes.  
这里要先创建数据库，你可以用EF Core [Migrations](xref:data/ef-mvc/migrations)（迁移）功能来完成它。 通过 **Migrations(迁移)** 你可以创建一个和数据模型相匹配的数据库，并且在数据模型发生改变的时候能够更新数据库的架构。

## Add EF tooling and perform initial migration  
添加 EF 工具并执行初始迁移

In this section you'll use the Package Manager Console (PMC) to:

* Add the Entity Framework Core Tools package. This package is required to add migrations and update the database.
* Add an initial migration.
* Update the database with the initial migration.

From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.

  ![PMC menu](adding-model/_static/pmc.png)

In the PMC, enter the following commands:

``` PMC
Install-Package Microsoft.EntityFrameworkCore.Tools
Add-Migration Initial
Update-Database
```

The `Add-Migration` command creates code to create the initial database schema. The schema is based on the model specified in the `DbContext`(In the *Data/MvcMovieContext.cs file). The `Initial` argument is used to name the migrations. You can use any name, but by convention you choose a name that describes the migration. See [Introduction to migrations](xref:data/ef-mvc/migrations#introduction-to-migrations) for more information.

The `Update-Database` command runs the `Up` method in the *Migrations/\<time-stamp>_InitialCreate.cs* file, which creates the database.

[!INCLUDE[adding-model](../../includes/mvc-intro/adding-model3.md)]

[!code-csharp[Main](../../tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Startup.cs?name=ConfigureServices&highlight=6-7)]

[!INCLUDE[adding-model](../../includes/mvc-intro/adding-model4.md)]

![Intellisense contextual menu on a Model item listing the available properties for ID, Price, Release Date, and Title](adding-model/_static/ints.png)

## Additional resources

* [Tag Helpers](xref:mvc/views/tag-helpers/intro)
* [Globalization and localization](xref:fundamentals/localization)

>[!div class="step-by-step"]
[Previous Adding a View](adding-view.md)
[Next Working with SQL](working-with-sql.md)  
