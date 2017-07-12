
# Working with SQL Server LocalDB  
操作 SQL Server LocalDB

By [Rick Anderson](https://twitter.com/RickAndMSFT)

The `MvcMovieContext` object handles the task of connecting to the database and mapping `Movie` objects to database records. The database context is registered with the [Dependency Injection](xref:fundamentals/dependency-injection) container in the The  `ConfigureServices` method in the *Startup.cs* file:  :  
`MvcMovieContext` 对象处理连接到数据库并将 `Movie` 对象映射到数据库的任务。 数据库上下文使用 [Dependency Injection(依赖注入)]容器在 *Startup.cs* 文件中的 `ConfigureServices` 方法进行注册:  

```C#
        // This method gets called by the runtime. Use this method to add services to the container.
        #region ConfigureServices
        public void ConfigureServices(IServiceCollection services)
        {
            // Add framework services.
            services.AddMvc();

            services.AddDbContext<MvcMovieContext>(options =>
                    options.UseSqlServer(Configuration.GetConnectionString("MvcMovieContext")));
        }
#endregion
```

[!code-csharp[Main](../../tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Startup.cs?name=ConfigureServices&highlight=6-7)]

The ASP.NET Core [Configuration](xref:fundamentals/configuration) system reads the `ConnectionString`. For local development, it gets the connection string from the *appsettings.json* file:  
ASP.NET Core [Configuration(配置)](xref:fundamentals/configuration)系统读取 `ConnectionString`。对于本地开发，它从 *appsettings.json* 文件获取连接字符串。

```JSON
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "ConnectionStrings": {
    "MvcMovieContext": "Server=(localdb)\\mssqllocaldb;Database=MvcMovieContext-20613a4b-deb5-4145-b6cc-a5fd19afda13;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

[!code-javascript[Main](start-mvc/sample/MvcMovie/appsettings.json?highlight=2&range=8-10)]

When you deploy the app to a test or production server, you can use an environment variable or another approach to set the connection string to a real SQL Server. See [Configuration](xref:fundamentals/configuration) for more information.  
当把这个应用程序部署到一个测试或生产服务器的时候，你可以使用环境变量或是其他的方法设置连接字符串到一个真实的 SQL Server。查看 [Configuration](xref:fundamentals/configuration) 获取更多的相关信息。

## SQL Server Express LocalDB

LocalDB is a lightweight version of the SQL Server Express Database Engine that is targeted for program development. LocalDB starts on demand and runs in user mode, so there is no complex configuration. By default, LocalDB database creates "\*.mdf" files in the *C:/Users/\<user\>* directory.  
LocalDB 是一个轻量级版本的用于程序开的 SQL Server Express 数据库引擎。LocalDB 按需要启动并以用户模式运行， 所以不需要复杂的配置。默认情况下，LocalDB数据库在 *C:/Users/\<user\>* 文件夹中创建 "\*.mdf" 文件。

* From the **View** menu, open **SQL Server Object Explorer** (SSOX).  
从 **View(视图)** 菜单, 打开 **SQL Server Object Explorer(SQL Server 对象资源管理器)**

  ![View menu](working-with-sql/_static/ssox.png)

* Right click on the `Movie` table **> View Designer**  
鼠标右键单击 `Movie` 表 **> View Designer(视图设计器)** 

  ![Contextual menu open on Movie table](working-with-sql/_static/design.png)

  ![Movie table open in Designer](working-with-sql/_static/dv.png)

Note the key icon next to `ID`. By default, EF will make a property named `ID` the primary key.  
注册在 `ID` 旁边的钥匙图标。默认情况下，EF将会把被命名为 `ID` 的属性作为主键。

* Right click on the `Movie` table **> View Data**  
鼠标右键单击 `Movie` 表， 选择 **> View Data(查看数据)**

  ![Contextual menu open on Movie table](working-with-sql/_static/ssox2.png)

  ![Movie table open showing table data](working-with-sql/_static/vd22.png)

## Seed the database  
种子数据库

Create a new class named `SeedData` in the *Models* folder. Replace the generated code with the following:  
在 *Models* 文件夹中创建一个新类并命名为 `SeedData`，将生成的代码替换为以下内容：

[!code-csharp[Main](start-mvc/sample/MvcMovie/Models/SeedData.cs?name=snippet_1)]
```C#
//#define First
#if First
// Seed without Rating
#region snippet_1 
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
                // Look for any movies.
                if (context.Movie.Any())
                {
                    return;   // DB has been seeded
                }

                context.Movie.AddRange(
                     new Movie
                     {
                         Title = "When Harry Met Sally",
                         ReleaseDate = DateTime.Parse("1989-1-11"),
                         Genre = "Romantic Comedy",
                         Price = 7.99M
                     },

                     new Movie
                     {
                         Title = "Ghostbusters ",
                         ReleaseDate = DateTime.Parse("1984-3-13"),
                         Genre = "Comedy",
                         Price = 8.99M
                     },

                     new Movie
                     {
                         Title = "Ghostbusters 2",
                         ReleaseDate = DateTime.Parse("1986-2-23"),
                         Genre = "Comedy",
                         Price = 9.99M
                     },

                   new Movie
                   {
                       Title = "Rio Bravo",
                       ReleaseDate = DateTime.Parse("1959-4-15"),
                       Genre = "Western",
                       Price = 3.99M
                   }
                );
                context.SaveChanges();
            }
        }
    }
}
#endregion
#endif
```
If there are any movies in the DB, the seed initializer returns and no movies are added.  
如果已经有任何电影在数据库中，种子的初始化程序将返回而不会电影被添加到数据库中。

```csharp
if (context.Movie.Any())
{
    return;   // DB has been seeded.
}
```

Add the seed initializer to the end of the `Configure` method in the *Startup.cs* file:  
将种子初始值设定项添加到 Startup.cs * 文件中 `Configure` 方法的末尾：
```C#
 SeedData.Initialize(app.ApplicationServices);
```
[!code-csharp[Main](start-mvc/sample/MvcMovie/Startup.cs?highlight=9&name=snippet_seed)]

Test the app  
测试应用程序

* Delete all the records in the DB. You can do this with the delete links in the browser or from SSOX.  
要删除数据库中的所有记录。你可以从浏览器中的删除链接或是从SSOX(SQL Server 对象资源管理器)中完成。 

* Force the app to initialize (call the methods in the `Startup` class) so the seed method runs. To force initialization, IIS Express must be stopped and restarted. You can do this with any of the following approaches:  
强制应用程序进行初始化（在 `Startup` 类中调用方法 ）以便让种子方法运行。 强制初始化，IIS Express 必须停止后重启，你可以使用下面的任意一种方法执行操作：

  * Right click the IIS Express system tray icon in the notification area and tap **Exit** or **Stop Site**  
  鼠标右键单击通知区域中的 IIS Express 系统图标并单击 **Exit(退出)** 或 **Stop Site(停止站点)** 
  

    ![IIS Express system tray icon](working-with-sql/_static/iisExIcon.png)

    ![Contextual menu](working-with-sql/_static/stopIIS.png)

   * If you were running VS in non-debug mode, press F5 to run in debug mode  
   如果 VS 正运行在非调试模式中，按 F5 在调试模式下运行
   
   * If you were running VS in debug mode, stop the debugger and press F5  
   如果 VS 正运行在调试模式下，按 F5 停止调试
   
The app shows the seeded data.  
应用程序显示的种子数据  

![MVC Movie application open in Microsoft Edge showing movie data](working-with-sql/_static/m55.png)

>[!div class="step-by-step"]
[前一页](adding-model.md)
[后一页](controller-methods-views.md)  
