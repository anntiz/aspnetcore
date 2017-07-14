# Adding a New Field  
添加新字段

By [Rick Anderson](https://twitter.com/RickAndMSFT)

In this section you'll use [Entity Framework](http://docs.efproject.net/en/latest/platforms/aspnetcore/new-db.html) Code First Migrations to add a new field to the model and migrate that change to the database.  
在本节中，你将使用 [Entity Framework](http://docs.efproject.net/en/latest/platforms/aspnetcore/new-db.html) 的代码优先迁移功能添加一个新字段到模型中，并将迁移更改到数据库中。

When you use EF Code First to automatically create a database, Code First adds a table to the database to help track whether the schema of the database is in sync with the model classes it was generated from. If they aren't in sync, EF throws an exception. This makes it easier to find inconsistent database/code issues.  
在使用 EF Code First(代码优先)自动创建数据库时，Code First(代码优先)将表添加到数据库中，以帮助跟踪数据库的架构是否与它生成的模型类同步。如果它们不同步，EF将会抛出一个异常，这个使得在查找数据库/代码不一致的问题时变得更加容易。

## Adding a Rating Property to the Movie Model  
添加一个 Rating 属性到 Movie 模型中

Open the *Models/Movie.cs* file and add a `Rating` property:
打开 *Models/Movie.cs* 文件并添加一个 `Rating` 属性：
```C#

//#define MovieDateRating
#if MovieDateRating
using System;
using System.ComponentModel.DataAnnotations;
namespace MvcMovie.Models
{
    public class Movie
    {
        public int ID { get; set; }
        public string Title { get; set; }

        [Display(Name = "Release Date")]
        [DataType(DataType.Date)]
        public DateTime ReleaseDate { get; set; }
        public string Genre { get; set; }
        public decimal Price { get; set; }
        public string Rating { get; set; }
    }

}
#endif
```

[!code-csharp[Main](start-mvc/sample/MvcMovie/Models/MovieDateRating.cs?highlight=11&range=7-18)]

Build the app (Ctrl+Shift+B).  
生成解决方案(Ctrl+Shift+B)

Because you've added a new field to the `Movie` class, you also need to update the binding white list so this new property will be included. In *MoviesController.cs*, update the `[Bind]` attribute for both the `Create` and `Edit` action methods to include the `Rating` property:  
因为你添加了一个新字段到 `Movie` 类中，还需要更新绑定的白名单把这个属性包含进去。 在 *MoviesController.cs* 中，同时更新 `Create` 和 `Edit` 这两个方法的 `[Bind]` 特性(attribute)，把 `Rating` 属性包含进去:

```csharp
[Bind("ID,Title,ReleaseDate,Genre,Price,Rating")]
   ```

You also need to update the view templates in order to display, create and edit the new `Rating` property in the browser view.  
还需要更新视图模板以便显示，在浏览器视图中创建和编辑新的 `Rating` 属性。

Edit the */Views/Movies/Index.cshtml* file and add a `Rating` field:  
编辑 */Views/Movies/Index.cshtml* 文件并添加一个 `Rating` 字段：

[!code-HTML[Main](start-mvc/sample/MvcMovie/Views/Movies/IndexGenreRating.cshtml?highlight=17,39&range=24-64)]

Update the */Views/Movies/Create.cshtml* with a `Rating` field. You can copy/paste the previous "form group" and let intelliSense help you update the fields. IntelliSense works with [Tag Helpers](xref:mvc/views/tag-helpers/intro). Note: In the RTM verison of Visual Studio 2017 you need to install the [Razor Language Services](https://marketplace.visualstudio.com/items?itemName=ms-madsk.RazorLanguageServices) for Razor intelliSense. This will be fixed in the next release.  
给 */Views/Movies/Create.cshtml*  添加一个 `Rating` 字段。可以先复制/粘贴前面的 "form group" （复制 class ="form group" 的整个div标签） 并通过智能提示来帮你完成，智能提示和[Tag Helpers](xref:mvc/views/tag-helpers/intro) 一起工作。注意：使用 RTM 版本的 Visual Studio 2017 必须先为 Razor 智能提示 安装 [Razor Language Services](https://marketplace.visualstudio.com/items?itemName=ms-madsk.RazorLanguageServices) 。这个问题将在下一个 release 版本中得到修复。

![The developer has typed the letter R for the attribute value of asp-for in the second label element of the view. An Intellisense contextual menu has appeared showing the available fields, including Rating, which is highlighted in the list automatically. When the developer clicks the field or presses Enter on the keyboard, the value will be set to Rating.](new-field/_static/cr.png)

The app won't work until we update the DB to include the new field. If you run it now, you'll get the following `SqlException`:  
这个应用程序在我们把新字段更新到数据库之前是不会工作的，如果现在运行这个程序，将会得到以下的异常 (`SqlException`)：

`SqlException: Invalid column name 'Rating'.`  （无效的列名 'Rating'）

You're seeing this error because the updated Movie model class is different than the schema of the Movie table of the existing database. (There's no Rating column in the database table.)  
你之所以看到这个错误是因为更新之后的 Movie 模型类和已经存在的数据库中的 Movie 表的结构不相同(数据库的表中并没有包含 Rating 列.)  

There are a few approaches to resolving the error:  
有几种办法可以解决这个错误：

1. Have the Entity Framework automatically drop and re-create the database based on the new model class schema. This approach is very convenient early in the development cycle when you are doing active development on a test database; it allows you to quickly evolve the model and database schema together. The downside, though, is that you lose existing data in the database — so you don't want to use this approach on a production database! Using an initializer to automatically seed a database with test data is often a productive way to develop an application.  
1. 让 Entity Framework 自动删除并重新基于新的模型类创建数据库。这个方法在开发周期的早期当你还是在测试数据库上进行开发的时候是非常方便的。可以让你快速地将模型和数据库架构一起演化。但这个方法的缺点是会丢失数据库中的现在数据，所以你不会希望在生产数据库上使用这个方法！在开发应用程序的时候对测试用的数据库使用初始值进行种子化是很常用的有效方法。

2. Explicitly modify the schema of the existing database so that it matches the model classes. The advantage of this approach is that you keep your data. You can make this change either manually or by creating a database change script.  
2. 显式修改现在数据库使其跟模型类匹配。这种方法的好处是可以保数据库的数据，你可以通过手动方式或是通过数据库脚本来完成这个修改。

3. Use Code First Migrations to update the database schema.  
3. 使用 Code First 迁移来更新数据库架构。

For this tutorial, we'll use Code First Migrations.  
在这个教程中，我们使用 Code First 迁移这个方法。

Update the `SeedData` class so that it provides a value for the new column. A sample change is shown below, but you'll want to make this change for each `new Movie`.  
更新 `SeedData` 类 为它的新列提供一个值。 下面显示的是一个修改的示例， 你要为每一个 `new Movie` 做这个修改。
```csharp
#define SeedRating 
#if SeedRating

using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using System;
using System.Linq;

namespace MvcMovie.Models
{
    public static class SeedData
    {
        public static void Initialize(IServiceProvider serviceProvider)
        {
            using (var context = new MvcMovieContext(
                serviceProvider.GetRequiredService<DbContextOptions<MvcMovieContext>>()))
            {
                if (context.Movie.Any())
                {
                    return;   // DB has been seeded
                }

                context.Movie.AddRange(
                #region snippet1
                     new Movie
                     {
                         Title = "When Harry Met Sally",
                         ReleaseDate = DateTime.Parse("1989-1-11"),
                         Genre = "Romantic Comedy",
                         Rating = "R",
                         Price = 7.99M
                     },
                #endregion
                     new Movie
                     {
                         Title = "Ghostbusters ",
                         ReleaseDate = DateTime.Parse("1984-3-13"),
                         Genre = "Comedy",
                         Rating = "G",
                         Price = 8.99M
                     },

                     new Movie
                     {
                         Title = "Ghostbusters 2",
                         ReleaseDate = DateTime.Parse("1986-2-23"),
                         Genre = "Comedy",
                         Rating = "PG",
                         Price = 9.99M
                     },

                   new Movie
                   {
                       Title = "Rio Bravo",
                       ReleaseDate = DateTime.Parse("1959-4-15"),
                       Genre = "Western",
                       Rating = "NA",
                       Price = 3.99M
                   }
                );
                context.SaveChanges();
            }
        }
    }
}



#endif
```
[!code-csharp[Main](start-mvc/sample/MvcMovie/Models/SeedDataRating.cs?name=snippet1&highlight=6)]

Build the solution.  
生成（或重新生成）项目


From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.  
打开 **Tools(工具)** 菜单，选择  **NuGet Package Manager(NuGet包管理器) > Package Manager Console(程序包管理器控制台)**

  ![PMC menu](adding-model/_static/pmc.png)

In the PMC, enter the following commands:  
在 **程序包管理器控制台** 中，输入以下命令：  

```PMC
Add-Migration Rating
Update-Database
```

The `Add-Migration` command tells the migration framework to examine the current `Movie` model with the current `Movie` DB schema and create the necessary code to migrate the DB to the new model. The name "Rating" is arbitrary and is used to name the migration file. It's helpful to use a meaningful name for the migration file.  
`Add-Migration` 命令告诉迁移框架去检查当前的 `Movie` 模型和 `Movie` 数据库架构并创建必要的代码把数据库迁移到新的模型上， 命令后面的 "Rating" 可以是任意的名字，它用于命名迁移文件。对于迁移文件使用一个有意义的名称会很有帮助。

If you delete all the records in the DB, the initialize will seed the DB and include the `Rating` field. You can do this with the delete links in the browser or from SSOX.  
如果删除了数据库中的所有记录，初始化程序将种子化数据库并包含 `Rating` 字段，你可以通过浏览器的删除链接或是SSOX（SQL Server 对象资源管理器）来完成进行操作。

Run the app and verify you can create/edit/display movies with a `Rating` field. You should also add the `Rating` field to the `Edit`, `Details`, and `Delete` view templates.  
运行这个应用程序并验证你可以 创建/编辑/显示 具有 `Rating` 字段的电影。接下来还应该把 `Rating` 字段增加到 `Edit`, `Details`, 和 `Delete` 视图模板中。

>[!div class="step-by-step"]
[前一页](search.md)
[后一页](validation.md)  
