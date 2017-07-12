# Controller methods and views  
控制器方法和视图

By [Rick Anderson](https://twitter.com/RickAndMSFT)

We have a good start to the movie app, but the presentation is not ideal. We don't want to see the time (12:00:00 AM in the image below) and **ReleaseDate** should be two words.  
对于电影应用程序我们已经有了一个好的开始，但这个程序的展示并不理想。 我们不希望看到时间(在下面的图片中的12:00:00 AM) 并且 **ReleaseDate** 应该是两个词(Release Date)。

![Index view: Release Date is one word (no space) and every movie release date shows a time of 12 AM](working-with-sql/_static/m55.png)

Open the *Models/Movie.cs* file and add the highlighted lines shown below:  
打开 *Models/Movie.cs* 文件，并添加下面突出显示的行：
```C#
        [Display(Name = "Release Date")]   //添加行1
        [DataType(DataType.Date)]          //添加行2
        public DateTime ReleaseDate { get; set; }
```
[!code-csharp[Main](start-mvc/sample/MvcMovie/Models/MovieDateWithExtraUsings.cs?name=snippet_1&highlight=13-14)]

Right click on a red squiggly line **> Quick Actions and Refactorings**.  
鼠标在红色波浪线的行上单击右键选择 **> Quick Actions and Refactorings(快速操作和重构)**

  ![Contextual menu shows **> Quick Actions and Refactorings**.](controller-methods-views/_static/qa.png)


Tap `using System.ComponentModel.DataAnnotations;`  
单击  `using System.ComponentModel.DataAnnotations;`

  ![using System.ComponentModel.DataAnnotations at top of list](controller-methods-views/_static/da.png)

  Visual studio adds `using System.ComponentModel.DataAnnotations;`.  
  Visual Studio 添加新的引用 `using System.ComponentModel.DataAnnotations;`.  

Let's remove the `using` statements that are not needed. They show up by default in a light grey font. Right click anywhere in the *Movie.cs* file **> Remove and Sort Usings**.  
我们把 `using` 声明中不需要的部分移除掉，默认情况下，它们显示为灰色的字体。 在 *Movie.cs*  文件的任意地方单击鼠标右键并选择 **> Remove and Sort Usings(对Usings进行删除和排序)**

![Remove and Sort Usings](controller-methods-views/_static/rm.png)

The updated code:  
更新之后的代码  

[!code-csharp[Main](./start-mvc/sample/MvcMovie/Models/MovieDate.cs?name=snippet_1)]

<!-- include start -->

[!INCLUDE[adding-model](../../includes/mvc-intro/controller-methods-views.md)]

>[!div class="step-by-step"]
[前一页](working-with-sql.md)
[下一页](search.md)  
