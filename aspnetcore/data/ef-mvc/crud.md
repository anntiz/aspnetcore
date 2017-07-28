# Create, Read, Update, and Delete - EF Core with ASP.NET Core MVC tutorial (2 of 10)  
增、查、改、删 -- ASP.NET Core MVC 使用 EF Core 教程

By [Tom Dykstra](https://github.com/tdykstra) and [Rick Anderson](https://twitter.com/RickAndMSFT)

The Contoso University sample web application demonstrates how to create ASP.NET Core 1.1 MVC web applications using Entity Framework Core 1.1 and Visual Studio 2017. For information about the tutorial series, see [the first tutorial in the series](intro.md).  
Contoso University 简单 web 应用程序演示了如何使用 Visual Studio 2017 和 Entity Framework Core 1.1 创建 ASP.NET Core 1.1 MVC web 应用程序。相关教程信息，请查看 [the first tutorial in the series(系列教程第一部分)](intro.md).  

In the previous tutorial you created an MVC application that stores and displays data using the Entity Framework and SQL Server LocalDB. In this tutorial you'll review and customize the CRUD (create, read, update, delete) code that the MVC scaffolding automatically creates for you in controllers and views.  
在前一部分的教程中，使用了 Entity Framework 和 SQL Server LocalDB 创建一个 MVC 应用程序用于存储和显示数据。在本教程中，你将查看并自定义 MVC 基架在控制器和视图中为你自动创建的 CRUD (create, read, update, delete)代码。 

> [!NOTE] 
> It's a common practice to implement the repository pattern in order to create an abstraction layer between your controller and the data access layer. To keep these tutorials simple and focused on teaching how to use the Entity Framework itself, they don't use repositories. For information about repositories with EF, see [the last tutorial in this series](advanced.md).  
> [!注意]  
> 实现存储模式是一种常见的作法，以在控制器和 DAL（data access layer 数据访问层）之间创建一个抽象层。为了保持本教程的简单化并专注于如何使用 Entity Framework 本身，这里并不使用存储库。关于 EF 中使用存储库的更多信息，请浏览 [the last tutorial in this series](advanced.md)


In this tutorial, you'll work with the following web pages:  
在本教程中，你将使用以下的网页：  

![Student Details page](crud/_static/student-details.png)

![Student Create page](crud/_static/student-create.png)

![Student Edit page](crud/_static/student-edit.png)

![Student Delete page](crud/_static/student-delete.png)

## Customize the Details page  
定制详细信息页

The scaffolded code for the Students Index page left out the `Enrollments` property, because that property holds a collection. In the **Details** page you'll display the contents of the collection in an HTML table.  
由基架为 Students Index  页生成的代码遗漏了 `Enrollments` 属性，因为这个属性保存的内容是一个集合。在  **Details**  页，你将在 HTML 表格中显示这个集合中的内容。

In *Controllers/StudentsController.cs*, the action method for the Details view uses the `SingleOrDefaultAsync` method to retrieve a single `Student` entity. Add code that calls `Include`. `ThenInclude`,  and `AsNoTracking` methods, as shown in the following highlighted code.  
在 *Controllers/StudentsController.cs* 中，Details 视图的操作方法使用 `SingleOrDefaultAsync` 方法去检索单个 `Student` 实体。添加调用`Include` 代码。`ThenInclude`,  和 `AsNoTracking` 方法，如以下显示的代码所示。

[!code-csharp[Main(StudentsController完整代码)](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Details&highlight=8-12)]
```c#
#region snippet_Details
        public async Task<IActionResult> Details(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var student = await _context.Students
                .Include(s => s.Enrollments)
                    .ThenInclude(e => e.Course)
                .AsNoTracking()
                .SingleOrDefaultAsync(m => m.ID == id);

            if (student == null)
            {
                return NotFound();
            }

            return View(student);
        }
#endregion
```

The `Include` and `ThenInclude` methods cause the context to load the `Student.Enrollments` navigation property, and within each enrollment the `Enrollment.Course` navigation property.  You'll learn more about these methods in the [reading related data](read-related-data.md) tutorial.  
`Include` 和 `ThenInclude` 方法导致上下文加载 `Student.Enrollments` 导航属性，以及在每个 enrollment 中的 `Enrollment.Course` 导航属性。您将在 [reading related data(读取相关数据)](read-related-data.md)教程中了解有关这些方法的更多信息。

The `AsNoTracking` method improves performance in scenarios where the entities returned will not be updated in the current context's lifetime. You'll learn more about `AsNoTracking` at the end of this tutorial.  
`AsNoTracking` 方法能够提高在当前上下文的生命周期内返回的实体不被更新的情况下的性能。在本教程的结尾部分，你将会学习更多关于 `AsNoTracking` 的内容。

### Route data  
路由数据

The key value that is passed to the `Details` method comes from *route data*. Route data is data that the model binder found in a segment of the URL. For example, the default route specifies controller, action, and id segments:  
传送给 `Details` 方法的键值来自 *route data(路由数据)*。路由数据模型绑定器在URL段中找到的数据。例如，默认路由指定控制器、动作和id段。

[!code-csharp[Main](intro/samples/cu/Startup.cs?name=snippet_RouteAndSeed&highlight=5)]
```c#
            #region snippet_RouteAndSeed
            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });

            DbInitializer.Initialize(context);
            #endregion
```

In the following URL, the default route maps Instructor as the controller, Index as the action, and 1 as the id; these are route data values.  
在下面的 URL，默认路由将 Instructor 映射为控制器，Index 映射为一个操作方法，1 映射为id（的值）；这些就是路由数据值。

```
http://localhost:1230/Instructor/Index/1?courseID=2021
```

The last part of the URL ("?courseID=2021") is a query string value. The model binder will also pass the ID value to the `Details` method `id` parameter if you pass it as a query string value:  
URL 的最后一部分("?courseID=2021") 是一个查询字符串值。如果将其做为查询字符值进行传递，模型绑定器也将 ID 的值传递到 `Details` 方法的 `id` 参数中。

```
http://localhost:1230/Instructor/Index?id=1&CourseID=2021
```

In the Index page, hyperlink URLs are created by tag helper statements in the Razor view. In the following Razor code, the `id` parameter matches the default route, so `id` is added to the route data.  
在 Index 页中，超链接 URL 是由 Razor 视图的 tag helper 语句创建。 在以下的 Razor 代码中， `id` 参数匹配默认路由，所以 `id` 会被添加到路由数据中。

```html
<a asp-action="Edit" asp-route-id="@item.ID">Edit</a>
```

This generates the following HTML when `item.ID` is 6:  
当 `item.ID` 的值为6时，将产生以下的 HTML 代码：

```html
<a href="/Students/Edit/6">Edit</a>
```

In the following Razor code, `studentID` doesn't match a parameter in the default route, so it's added as a query string.  
在以下的 Razor 代码中， `studentID` 不匹配默认路由中的参数，所以它会被做为查询字符串添加。

```html
<a asp-action="Edit" asp-route-studentID="@item.ID">Edit</a>
```

This generates the following HTML when `item.ID` is 6:  
当 `item.ID` 的值为6时，将产生以下的 HTML 代码：

```html
<a href="/Students/Edit?studentID=6">Edit</a>
```

For more information about tag helpers, see [Tag helpers in ASP.NET Core](xref:mvc/views/tag-helpers/intro).  
更多关于 about tag helpers 的信息，请浏览 [Tag helpers in ASP.NET Core](xref:mvc/views/tag-helpers/intro).

### Add enrollments to the Details view  
添加在校人数到详细信息视图

Open *Views/Students/Details.cshtml*. Each field is displayed using `DisplayNameFor` and `DisplayFor` helper, as shown in the following example:  
打开  *Views/Students/Details.cshtml* 文件，每个字段都使用 `DisplayNameFor` 和 `DisplayFor` 帮助器显示，如下面的例子所示：

[!code-html[Details](intro/samples/cu/Views/Students/Details.cshtml?range=13-18&highlight=2,5)]  

```xml
@model ContosoUniversity.Models.Student

@{
    ViewData["Title"] = "Details";
}

<h2>Details</h2>

<div>
    <h4>Student</h4>
    <hr />
    <dl class="dl-horizontal">
        <dt>
            @Html.DisplayNameFor(model => model.LastName)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.LastName)
        </dd>
        <dt>
            @Html.DisplayNameFor(model => model.FirstMidName)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.FirstMidName)
        </dd>
        <dt>
            @Html.DisplayNameFor(model => model.EnrollmentDate)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.EnrollmentDate)
        </dd>
        <dt>
            @Html.DisplayNameFor(model => model.Enrollments)
        </dt>
    </dl>
</div>
<div>
    <a asp-action="Edit" asp-route-id="@Model.ID">Edit</a> |
    <a asp-action="Index">Back to List</a>
</div>
```

After the last field and immediately before the closing `</dl>` tag, add the following code to display a list of enrollments:  
在最后一个字段的后面，紧接在关闭 "</dl>" 标记之前，添加下面的代码去显示一个在校人数列表：

[!code-html[Details](intro/samples/cu/Views/Students/Details.cshtml?range=31-52)]  

```xml
@model ContosoUniversity.Models.Student

@{
    ViewData["Title"] = "Details";
}

<h2>Details</h2>

<div>
    <h4>Student</h4>
    <hr />
    <dl class="dl-horizontal">
        <dt>
            @Html.DisplayNameFor(model => model.LastName)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.LastName)
        </dd>
        <dt>
            @Html.DisplayNameFor(model => model.FirstMidName)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.FirstMidName)
        </dd>
        <dt>
            @Html.DisplayNameFor(model => model.EnrollmentDate)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.EnrollmentDate)
        </dd>
        <dt>
            @Html.DisplayNameFor(model => model.Enrollments)
        </dt>
        <dd>
            <table class="table">
                <tr>
                    <th>Course Title</th>
                    <th>Grade</th>
                </tr>
                @foreach (var item in Model.Enrollments)
                {
                    <tr>
                        <td>
                            @Html.DisplayFor(modelItem => item.Course.Title)
                        </td>
                        <td>
                            @Html.DisplayFor(modelItem => item.Grade)
                        </td>
                    </tr>
                }
            </table>
        </dd>
    </dl>
</div>
<div>
    <a asp-action="Edit" asp-route-id="@Model.ID">Edit</a> |
    <a asp-action="Index">Back to List</a>
</div>
```
If code indentation is wrong after you paste the code, press CTRL-K-D to correct it.    
如果在粘贴代码后出现代码缩进错误, 请按 CTRL-K-D 进行更正。

This code loops through the entities in the `Enrollments` navigation property. For each enrollment, it displays the course title and the grade. The course title is retrieved from the Course entity that's stored in the `Course` navigation property of the Enrollments entity.  
这段代码将循环遍历 `Enrollments` 导航属性中的实体，对于每个 enrollment，都显示它的课程标题和成绩。课程标题都是从存储在 Enrollments 实体的 Course 导航属性中的 Course 实体中检索出来的。

Run the application, select the **Students** tab, and click the **Details** link for a student. You see the list of courses and grades for the selected student:  
运行应用程序，选择 **Students** 标签，然后单击 **Details** 中的一个学生链接。就可以看到被选中学生的课程和成绩列表：

![Student Details page](crud/_static/student-details.png)

## Update the Create page  
修改 Create(增加)页 

In *StudentsController.cs*, modify the HttpPost `Create` method by adding a try-catch block and removing ID from the `Bind` attribute.  
在 *StudentsController.cs* 文件中，修改 HttpPost 方式的 `Create` 方法，添加一个  try-catch 块结构并从 `Bind` 特性中删除 ID。

[!code-csharp[Main](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Create&highlight=4,6-7,14-21)]  
```c#
        // POST: Students/Create
#region snippet_Create
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Create(
            [Bind("EnrollmentDate,FirstMidName,LastName")] Student student)
        {
            try
            {
                if (ModelState.IsValid)
                {
                    _context.Add(student);
                    await _context.SaveChangesAsync();
                    return RedirectToAction("Index");
                }
            }
            catch (DbUpdateException /* ex */)
            {
                //Log the error (uncomment ex variable name and write a log.
                ModelState.AddModelError("", "Unable to save changes. " +
                    "Try again, and if the problem persists " +
                    "see your system administrator.");
            }
            return View(student);
        }
#endregion
```

This code adds the Student entity created by the ASP.NET MVC model binder to the Students entity set and then saves the changes to the database. (Model binder refers to the ASP.NET MVC functionality that makes it easier for you to work with data submitted by a form; a model binder converts posted form values to CLR types and passes them to the action method in parameters. In this case, the model binder instantiates a Student entity for you using property values from the Form collection.)  
这段代码将由 ASP.NET MVC 模型绑定器创建的 Student 实体添加到 Students 实体集，然后将更改可在到数据库中。(模型绑定器引用 ASP.NET MVC 功能，可以更轻松处理从表单提交的数据；模型绑定器将发布的表单值转换为CLR类型，并将它们传递给参数中的操作方法。在这种情况下，模型绑定器使用 Form 集合中的属性值实例化一个 Student 实体。)

You removed `ID` from the `Bind` attribute because ID is the primary key value which SQL Server will set automatically when the row is inserted. Input from the user does not set the ID value.  
从 `Bind` 特性中删除 `ID` 是因为  `ID` 是 任何一个 SQL Server 插入行时都会自动添加的主键值。而来看成用户的输入并没有提供 ID 值。

Other than the `Bind` attribute, the try-catch block is the only change you've made to the scaffolded code. If an exception that derives from `DbUpdateException` is caught while the changes are being saved, a generic error message is displayed. `DbUpdateException` exceptions are sometimes caused by something external to the application rather than a programming error, so the user is advised to try again. Although not implemented in this sample, a production quality application would log the exception. For more information, see the **Log for insight** section in [Monitoring and Telemetry (Building Real-World Cloud Apps with Azure)](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry).  
除了 `Bind` 特性之外，try-catch 块结构是对由基架生成的代码的唯一更改。如果在保存更新时捕获来自 `DbUpdateException` 的异常，则会显示一个通用的错误信息。`DbUpdateException` 异常是有时候是由应用程序外部的某些原因引起的而不是编程错误，因此建议用户再次尝试。虽然此示例中未实现, 但生产质量应用程序将记录该异常。有关更多的信息，请在 [Monitoring and Telemetry (Building Real-World Cloud Apps with Azure)](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry) 上浏览有 **Log for insight** 部分的内容。

The `ValidateAntiForgeryToken` attribute helps prevent cross-site request forgery (CSRF) attacks. The token is automatically injected into the view by the [FormTagHelper](xref:mvc/views/working-with-forms#the-form-tag-helper) and is included when the form is submitted by the user. The token is validated by the `ValidateAntiForgeryToken` attribute. For more information about CSRF, see [🔧 Anti-Request Forgery](../../security/anti-request-forgery.md).  
`ValidateAntiForgeryToken` 特性有助于防止跨站点请求伪造（CSRF）攻击。这个 token(令牌)是 [FormTagHelper](xref:mvc/views/working-with-forms#the-form-tag-helper) 自动注入视图，并在用户提交表单时包含该令牌。 令牌由 `ValidateAntiForgeryToken` 特性验证。更多有关 CSRF 的令牌，请浏览  [🔧 Anti-Request Forgery](../../security/anti-request-forgery.md).  


<a id="overpost"></a>
### Security note about overposting  
关于 overposting(过多发布) 的安全说明

The `Bind` attribute that the scaffolded code includes on the `Create` method is one way to protect against overposting in create scenarios. For example, suppose the Student entity includes a `Secret` property that you don't want this web page to set.  
 基架代码包含在 `Create` 方法中的 `Bind` 特性是在创建方案中防止过多发布的一种方法。例如，假设 Student 实体中包含一个你不希望在这个网页中设置的 `Secret` 属性。

```csharp
public class Student
{
    public int ID { get; set; }
    public string LastName { get; set; }
    public string FirstMidName { get; set; }
    public DateTime EnrollmentDate { get; set; }
    public string Secret { get; set; }
}
```

Even if you don't have a `Secret` field on the web page, a hacker could use a tool such as Fiddler, or write some JavaScript, to post a `Secret` form value. Without the `Bind` attribute limiting the fields that the model binder uses when it creates a Student instance, the model binder would pick up that `Secret` form value and use it to create the Student entity instance. Then whatever value the hacker specified for the `Secret` form field would be updated in your database. The following image shows the Fiddler tool adding the `Secret` field (with the value "OverPost") to the posted form values.  
即使你的的网页上没有 `Secret` 字段，黑客也可以使用诸如 Fiddler 之类的工具，或者编写一段 JavaScript代码，去提交一个 `Secret`表单值。如果不使用 `Bind` 特性限制模型绑定器创建 Student 实例时使用的字段，模型绑定器将获取  `Secret` 表单值，并使用它去创建 Student 实体实例。然后，无论黑客为  `Secret` 表单字段指定的是什么值都会被更新到数据库中。下图显示了 Fiddler 工具将 `Secret` 字段(字段的值为 "OverPost") 添加到已提交的表单值中。

![Fiddler adding Secret field](crud/_static/fiddler.png)

The value "OverPost" would then be successfully added to the `Secret` property of the inserted row, although you never intended that the web page be able to set that property.  
然后，值 "OverPost" 将被成功地添加到插入行的 `Secret`属性中，尽管你从未打算在网页中设置该属性。

You can prevent overposting in edit scenarios by reading the entity from the database first and then calling `TryUpdateModel`, passing in an explicit allowed properties list. That is the method used in these tutorials.  
你可以先从数据库中读取实体，然后再调用  `TryUpdateModel` 来阻止编辑场景中的 overposting (过多发布)。传入一个显示允许（白名单）的属性列表。这是在本教程中使用的方法。

An alternative way to prevent overposting that is preferred by many developers is to use view models rather than entity classes with model binding. Include only the properties you want to update in the view model. Once the MVC model binder has finished, copy the view model properties to the entity instance, optionally using a tool such as AutoMapper. Use `_context.Entry` on the entity instance to set its state to `Unchanged`, and then set `Property("PropertyName").IsModified` to true on each entity property that is included in the view model. This method works in both edit and create scenarios.  
防止 overposting (过多发布)的另一种办法是使用视图模型而不是具有模型绑定的实体类。仅包含在视图模型中需要更新的属性。 MVC 模型绑定器完成后，将视图模型属性复制到实体实例中，可以使用诸如 AutoMapper 之类的工具。在实体实例上使用 `_context.Entry`，将其状态设置为 `Unchanged`，然后在视图模型中包含的每个实体属性上设置 `Property("PropertyName").IsModified` 设置为 true。这个方法适用于编辑和创建场景。

### Test the Create page  
测试 Create 页

The code in *Views/Students/Create.cshtml* uses `label`, `input`, and `span` (for validation messages) tag helpers for each field.  
对 *Views/Students/Create.cshtml* 文件中的每一个字段使用 `label`, `input`, 和 `span`(用于验证消息) 标记辅助代码。

Run the page by selecting the **Students** tab and clicking **Create New**.  
通过选择 **Students** 标签并单击 **Create New** 运行这个页面。

Enter names and an invalid date and click **Create** to see the error message.  
输入名字 和一个无效的日期然后单击 **Create** 以查看错误信息。

![Date validation error](crud/_static/date-error.png)

This is server-side validation that you get by default; in a later tutorial you'll see how to add attributes that will generate code for client-side validation also. The following highlighted code shows the model validation check in the `Create` method.  
这是默认情况下得到的服务器端验证；在后面的教程，你将看到如何添加为客户端验证生成代码的特性。以下突出显示的代码显示了在 `Create` 方法中的模型验证检查。

[!code-csharp[Main](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_Create&highlight=8)]
```c#
        // POST: Students/Create
#region snippet_Create
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Create(
            [Bind("EnrollmentDate,FirstMidName,LastName")] Student student)
        {
            try
            {
                if (ModelState.IsValid)
                {
                    _context.Add(student);
                    await _context.SaveChangesAsync();
                    return RedirectToAction("Index");
                }
            }
            catch (DbUpdateException /* ex */)
            {
                //Log the error (uncomment ex variable name and write a log.
                ModelState.AddModelError("", "Unable to save changes. " +
                    "Try again, and if the problem persists " +
                    "see your system administrator.");
            }
            return View(student);
        }
#endregion
```

Change the date to a valid value and click **Create** to see the new student appear in the **Index** page.  
把日期更改为一个有效的值然后单击 **Create** 查看出现在 **Index**  页面的新 student 。

## Update the Edit page  
更新 Edit 页

In *StudentController.cs*, the HttpGet `Edit` method (the one without the `HttpPost` attribute) uses the `SingleOrDefaultAsync` method to retrieve the selected Student entity, as you saw in the `Details` method. You don't need to change this method.  
在 *StudentController.cs* 文件中， HttpGet 方式的 `Edit` 方法（没有 `HttpPost` 特性的那一个）使用 `SingleOrDefaultAsync` 方法来检索已经选定的 Student 实体，就象你在 `Details`方法中看到的那样。你不需要修改这个方法。

### Recommended HttpPost Edit code: Read and update  
推荐 HttpPost Edit 代码：读取和更新

Replace the HttpPost Edit action method with the following code.  
使用以下的代码替换 HttpPost Edit 操作方法的内容。

[!code-csharp[Main](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_ReadFirst)]
```c#
#elif (ReadFirst)
#region snippet_ReadFirst
        [HttpPost, ActionName("Edit")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> EditPost(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }
            var studentToUpdate = await _context.Students.SingleOrDefaultAsync(s => s.ID == id);
            if (await TryUpdateModelAsync<Student>(
                studentToUpdate,
                "",
                s => s.FirstMidName, s => s.LastName, s => s.EnrollmentDate))
            {
                try
                {
                    await _context.SaveChangesAsync();
                    return RedirectToAction("Index");
                }
                catch (DbUpdateException /* ex */)
                {
                    //Log the error (uncomment ex variable name and write a log.)
                    ModelState.AddModelError("", "Unable to save changes. " +
                        "Try again, and if the problem persists, " +
                        "see your system administrator.");
                }
            }
            return View(studentToUpdate);
        }
#endregion
#endif
```

These changes implement a security best practice to prevent overposting. The scaffolder generated a `Bind` attribute and added the entity created by the model binder to the entity set with a `Modified` flag. That code is not recommended for many scenarios because the `Bind` attribute clears out any pre-existing data in fields not listed in the `Include` parameter.  
这些更改实现国安全最佳做法以防止过多发布。基架生成了一个 `Bind` 特性并添加模型绑定器创建的实体到 `Modified` 标志的实体集中。在很多情况下都不推荐使用这些代码，因为 `Bind` 特性清除了 `Include` 参数中未列出来的字段中的任何预先存在的代码。

The new code reads the existing entity and calls `TryUpdateModel` to update fields in the retrieved entity [based on user input in the posted form data](xref:mvc/models/model-binding#how-model-binding-works). The Entity Framework's automatic change tracking sets the `Modified` flag on the fields that are changed by form input. When the `SaveChanges` method is called, the Entity Framework creates SQL statements to update the database row. Concurrency conflicts are ignored, and only the table columns that were updated by the user are updated in the database. (A later tutorial shows how to handle concurrency conflicts.)  
新代码读取现在实体并调用 `TryUpdateModel` 去更新检索实体 [based on user input in the posted form data](xref:mvc/models/model-binding#how-model-binding-works)中的字段。 Entity Framework的自动更改跟踪在通过表单输入更改的字段上设置 `Modified` 标志。当 `SaveChanges` 方法被调用，Entity Framework 会创建 SQL 语句去更新数据库中的行。忽略并发冲突，并且在数据库中只更新由用户更新的表列(后面的教程将演示如何处理并发冲突)。

As a best practice to prevent overposting, the fields that you want to be updateable by the **Edit** page are whitelisted in the `TryUpdateModel` parameters. (The empty string preceding the list of fields in the parameter list is for a prefix to use with the form fields names.) Currently there are no extra fields that you're protecting, but listing the fields that you want the model binder to bind ensures that if you add fields to the data model in the future, they're automatically protected until you explicitly add them here.  
作为防止 overposting(过多发布) 的最佳做法， 你可以通过在 **Edit** 页面把要更新的字段加入到 `TryUpdateModel` 参数的白名单(参数列表中的字段列表之前的空字符串是用于表单字段名称的前缀)。当前没有要保护的额外字段，但是列出你希望模型绑定器绑定的字段以确保如果你在将来为数据模型添加字段时，它们会被自动保护直到你在这里显式添加它们为止。

As a result of these changes, the method signature of the HttpPost `Edit` method is the same as the HttpGet `Edit` method; therefore you've renamed the method `EditPost`.  
作为这些更改的结果，HttpPost 方式的 `Edit` 方法的方法签名与 HttpGet 方式的 `Edit` 方法相同；所以你已经重命名了  `EditPost` 方法。

### Alternative HttpPost Edit code: Create and attach  
替代的 HttpPost Edit 代码：创建和附加

The recommended HttpPost edit code ensures that only changed columns get updated and preserves data in properties that you don't want included for model binding. However, the read-first approach requires an extra database read, and can result in more complex code for handling concurrency conflicts. An alternative is to attach an entity created by the model binder to the EF context and mark it as modified. (Don't update your project with this code, it's only shown to illustrate an optional approach.)  
推荐的 HttpPost edit 代码可确保只有变了的列得到更新，并保留不想包含在模型绑定的属性中的数据。然而，读取优先方法需要额外的数据库读取，并会导致更复杂的代码来处理并发冲突。另一种方法是将由模型绑定器创建的实体附加到 EF 上下文并将其标记为已修改（不要用此代码更新你的项目，这里只是演示了一个可选的方法）。

[!code-csharp[Main](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_CreateAndAttach)]
```c#
#if (CreateAndAttach)
#region snippet_CreateAndAttach
        public async Task<IActionResult> Edit(int id, [Bind("ID,EnrollmentDate,FirstMidName,LastName")] Student student)
        {
            if (id != student.ID)
            {
                return NotFound();
            }
            if (ModelState.IsValid)
            {
                try
                {
                    _context.Update(student);
                    await _context.SaveChangesAsync();
                    return RedirectToAction("Index");
                }
                catch (DbUpdateException /* ex */)
                {
                    //Log the error (uncomment ex variable name and write a log.)
                    ModelState.AddModelError("", "Unable to save changes. " +
                        "Try again, and if the problem persists, " +
                        "see your system administrator.");
                }
            }
            return View(student);
        }
#endregion
#elif (ReadFirst)
#region snippet_ReadFirst
        [HttpPost, ActionName("Edit")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> EditPost(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }
            var studentToUpdate = await _context.Students.SingleOrDefaultAsync(s => s.ID == id);
            if (await TryUpdateModelAsync<Student>(
                studentToUpdate,
                "",
                s => s.FirstMidName, s => s.LastName, s => s.EnrollmentDate))
            {
                try
                {
                    await _context.SaveChangesAsync();
                    return RedirectToAction("Index");
                }
                catch (DbUpdateException /* ex */)
                {
                    //Log the error (uncomment ex variable name and write a log.)
                    ModelState.AddModelError("", "Unable to save changes. " +
                        "Try again, and if the problem persists, " +
                        "see your system administrator.");
                }
            }
            return View(studentToUpdate);
        }
#endregion
#endif
```

You can use this approach when the web page UI includes all of the fields in the entity and can update any of them.  
当网页UI包含实体中的所有字段并且可以更新它们中的任意一个的时候，可以使用这个方法。

The scaffolded code uses the create-and-attach approach but only catches `DbUpdateConcurrencyException` exceptions and returns 404 error codes.  The example shown catches any database update exception and displays an error message.  
基架代码使用创建附加方法，但只捕获 `DbUpdateConcurrencyException` 异常并返回 404 错误代码。显示的示例会捕获任何数据库更新异常并显示错误信息。

### Entity States
实体状态

The database context keeps track of whether entities in memory are in sync with their corresponding rows in the database, and this information determines what happens when you call the `SaveChanges` method. For example, when you pass a new entity to the `Add` method, that entity's state is set to `Added`. Then when you call the `SaveChanges` method, the database context issues a SQL INSERT command.  
数据库上下文跟踪内存中的实体是否与数据库中相应的行同步，并且此信息确定当你调用 `SaveChanges` 方法时会发生什么。例如，当你传递一个新的实体到 `Add` 方法时，该实体的状态设置为 `Added`。当调用 `SaveChanges` 方法时，数据库上下文则会发出一个 INSERT 的 SQL 命令。

An entity may be in one of the following states:  
实体可能处于下列状态之一：

* `Added`. The entity does not yet exist in the database. The `SaveChanges` method issues an INSERT statement.  
 `Added`：该实体还不存在于数据库中。 `SaveChanges` 方法发出一个 INSERT 语句。

* `Unchanged`. Nothing needs to be done with this entity by the `SaveChanges` method. When you read an entity from the database, the entity starts out with this status.  
`Unchanged`：`SaveChanges`方法不需要对该实体进行任何操作。当从数据库中读取一个实体时，就是从这个状态开始的。

* `Modified`. Some or all of the entity's property values have been modified. The `SaveChanges` method issues an UPDATE statement.  
 `Modified`：该实体中的部分或全部属性被修改。`SaveChanges` 方法发出一个 UPDATE 语句。

* `Deleted`. The entity has been marked for deletion. The `SaveChanges` method issues a DELETE statement.  
`Deleted`：该实体被标记为删除。`SaveChanges` 方法发出一个 DELETE 语句。

* `Detached`. The entity isn't being tracked by the database context.  
`Detached`：该实体没有被数据库上下文跟踪。

In a desktop application, state changes are typically set automatically. You read an entity and make changes to some of its property values. This causes its entity state to automatically be changed to `Modified`. Then when you call `SaveChanges`, the Entity Framework generates a SQL UPDATE statement that updates only the actual properties that you changed.  
在桌面应用程序中，状态改变通常是自动设置的。读取一个实体并改变其中的一些属性值，会导致该实体的状态自动更改为 `Modified`。那么当你调用  `SaveChanges` 方法时， Entity Framework 会生成一个 UPDATE 的 SQL 语句，只更新你改变了的实际属性。

In a web app, the `DbContext` that initially reads an entity and displays its data to be edited is disposed after a page is rendered. When the HttpPost `Edit` action method is called,  a new web request is made and you have a new instance of the `DbContext`. If you re-read the entity in that new context, you simulate desktop processing.  
在 web 应用程序中， 最初读取一个实体并显示其要编辑的数据的 `DbContext` 在页面被渲染之后释放。当调用 HttpPost 方式的 `Edit` 操作方法时，将发出一个新的 web 请求并得到一个新的 `DbContext` 实例。如果在一个新的上下文中重新读取了实体，则可以模拟桌面应用程序的处理。

But if you don't want to do the extra read operation, you have to use the entity object created by the model binder.  The simplest way to do this is to set the entity state to Modified as is done in the alternative HttpPost Edit code shown earlier. Then when you call `SaveChanges`, the Entity Framework updates all columns of the database row, because the context has no way to know which properties you changed.  
但如果你不想做额外的读取操作，就必须使用由模型绑定器创建的实体对象。最简单在方法是将实体状态修改为前面所示的替代的 HttpPost Edit 代码。当调用  `SaveChanges` 方法时，Entity Framework 将更新数据库行中的所有列，因为上下文没有办法知道你修改了哪些属性。

If you want to avoid the read-first approach, but you also want the SQL UPDATE statement to update only the fields that the user actually changed, the code is more complex. You have to save the original values in some way (such as by using hidden fields) so that they are available when the HttpPost `Edit` method is called. Then you can create a Student entity using the original values, call the `Attach` method with that original version of the entity, update the entity's values to the new values, and then call `SaveChanges`.  
如果想避免读取优先方法，同时又希望 SQL UPDATE 语句只更新用户实际更改的字段，代码会更复杂。你必须以某种方式(例如使用隐藏字段)保存原始值，以便在调用 
HttpPost `Edit` 方法时他们是可用的。你可以使用原始值创建一个 Student 实体，调用该实体原始版本的 `Attach` 方法，将实体的值更新为新的值，然后调用  `SaveChanges` 方法。  

### Test the Edit page  
测试 Edit 页

Run the application and select the **Students** tab, then click an **Edit** hyperlink.  
运行这个应用程序并且选择 **Students** 标签，然后单击 **Edit** 超链接。

![Students edit page](crud/_static/student-edit.png)

Change some of the data and click **Save**. The **Index** page opens and you see the changed data.  
修改一些数据并单击  **Save**。在打开的 **Index** 页面查看修改后的数据。

## Update the Delete page  更新 Delete 页面  

In *StudentController.cs*, the template code for the HttpGet `Delete` method uses the `SingleOrDefaultAsync` method to retrieve the selected Student entity, as you saw in the Details and Edit methods. However, to implement a custom error message when the call to `SaveChanges` fails, you'll add some functionality to this method and its corresponding view.  
在 *StudentController.cs* 文件中，HttpGet `Delete` 方法的模板代码使用 `SingleOrDefaultAsync` 方法检索选中的 Student 实体，就象是在 Details 和 Edit 方法中看到的那样。但是，当调用 `SaveChanges` 失败要实现一个自定义的错误信息时，你需要在该方法及其相应视图中添加一些功能。

As you saw for update and create operations, delete operations require two action methods. The method that is called in response to a GET request displays a view that gives the user a chance to approve or cancel the delete operation. If the user approves it, a POST request is created. When that happens, the HttpPost `Delete` method is called and then that method actually performs the delete operation.  
跟你在 update 和 create 操作中看到的一样， delete 操作也需要两个操作方法。 响应 GET 请求时调用的方法显示一个视图，让用户选择批准或是取消 delete 操作。如果用户批准删除则创建一个 POST 请求。这个时候， HttpPost `Delete` 方法将被调用并执行真正的删除操作。

You'll add a try-catch block to the HttpPost `Delete` method to handle any errors that might occur when the database is updated. If an error occurs, the HttpPost Delete method calls the HttpGet Delete method, passing it a parameter that indicates that an error has occurred. The HttpGet Delete method then redisplays the confirmation page along with the error message, giving the user an opportunity to cancel or try again.  
在 HttpPost `Delete` 方法中添加一个 try-catch 语句块处理在数据库更新时可能发生的任何错误。如果错误出再，HttpPost Delete 方法调用 HttpGet Delete 方法，同时传递一个指示错误发生的参数。HttpGet Delete 方法重新显示确认删除的页面以及相应的错误信息，让用户一个取消或再次尝试(删除)的机会。

Replace the HttpGet `Delete` action method with the following code, which manages error reporting.  
替换 HttpGet `Delete` 操作方法为以下代码，该代码用于管理错误报告。

[!code-csharp[Main](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DeleteGet&highlight=1,9,16-21)]

```c#
        // GET: Students/Delete/5
#region snippet_DeleteGet
        public async Task<IActionResult> Delete(int? id, bool? saveChangesError = false)
        {
            if (id == null)
            {
                return NotFound();
            }

            var student = await _context.Students
                .AsNoTracking()
                .SingleOrDefaultAsync(m => m.ID == id);
            if (student == null)
            {
                return NotFound();
            }

            if (saveChangesError.GetValueOrDefault())
            {
                ViewData["ErrorMessage"] =
                    "Delete failed. Try again, and if the problem persists " +
                    "see your system administrator.";
            }

            return View(student);
        }
#endregion
```

This code accepts an optional parameter that indicates whether the method was called after a failure to save changes. This parameter is false when the HttpGet `Delete` method is called without a previous failure. When it is called by the HttpPost `Delete` method in response to a database update error, the parameter is true and an error message is passed to the view.  
此代码接受一个可选的参数，该参数指示方法在保存更改失败后被调用。当 HttpGet `Delete` 方法在没有之前的失败被调用时该参数的值为 false。 当 HttpPost `Delete` 方法在响应一个数据库更新错误被调用时，该参数的值是 true 并同时传递一个错误信息到相应的视图。

### The read-first approach to HttpPost Delete  HttpPost Delete 的 read-first 方法

Replace the HttpPost `Delete` action method (named `DeleteConfirmed`) with the following code, which performs the actual delete operation and catches any database update errors.

[!code-csharp[Main](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DeleteWithReadFirst&highlight=6,8-11,13-14,18-23)]

This code retrieves the selected entity, then calls the `Remove` method to set the entity's status to `Deleted`. When `SaveChanges` is called, a SQL DELETE command is generated.

### The create-and-attach approach to HttpPost Delete

If improving performance in a high-volume application is a priority, you could avoid an unnecessary SQL query by instantiating a Student entity using only the primary key value and then setting the entity state to `Deleted`. That's all that the Entity Framework needs in order to delete the entity. (Don't put this code in your project; it's here just to illustrate an alternative.)

[!code-csharp[Main](intro/samples/cu/Controllers/StudentsController.cs?name=snippet_DeleteWithoutReadFirst&highlight=7-8)]

If the entity has related data that should also be deleted, make sure that cascade delete is configured in the database. With this approach to entity deletion, EF might not realize there are related entities to be deleted.

### Update the Delete view

In *Views/Student/Delete.cshtml*, add an error message between the h2 heading and the h3 heading, as shown in the following example:

[!code-html[](intro/samples/cu/Views/Students/Delete.cshtml?range=7-9&highlight=2)]

Run the page by selecting the **Students** tab and clicking a **Delete** hyperlink:

![Delete confirmation page](crud/_static/student-delete.png)

Click **Delete**. The Index page is displayed without the deleted student. (You'll see an example of the error handling code in action in the concurrency tutorial.)

## Closing database connections

To free up the resources that a database connection holds, the context instance must be disposed as soon as possible when you are done with it. The ASP.NET Core built-in [dependency injection](../../fundamentals/dependency-injection.md) takes care of that task for you.

In *Startup.cs* you call the [AddDbContext extension method](https://github.com/aspnet/EntityFramework/blob/03bcb5122e3f577a84498545fcf130ba79a3d987/src/Microsoft.EntityFrameworkCore/EntityFrameworkServiceCollectionExtensions.cs) to provision the `DbContext` class in the ASP.NET DI container. That method sets the service lifetime to `Scoped` by default. `Scoped` means the context object lifetime coincides with the web request life time, and the `Dispose` method will be called automatically at the end of the web request.

## Handling Transactions

By default the Entity Framework implicitly implements transactions. In scenarios where you make changes to multiple rows or tables and then call `SaveChanges`, the Entity Framework automatically makes sure that either all of your changes succeed or they all fail. If some changes are done first and then an error happens, those changes are automatically rolled back. For scenarios where you need more control -- for example, if you want to include operations done outside of Entity Framework in a transaction -- see [Transactions](https://docs.microsoft.com/ef/core/saving/transactions).

## No-tracking queries

When a database context retrieves table rows and creates entity objects that represent them, by default it keeps track of whether the entities in memory are in sync with what's in the database. The data in memory acts as a cache and is used when you update an entity. This caching is often unnecessary in a web application because context instances are typically short-lived (a new one is created and disposed for each request) and the context that reads an entity is typically disposed before that entity is used again.

You can disable tracking of entity objects in memory by calling the `AsNoTracking` method. Typical scenarios in which you might want to do that include the following:

* During the context lifetime you don't need to update any entities, and you don't need EF to [automatically load navigation properties with  entities retrieved by separate queries](read-related-data.md). Frequently these conditions are met in a controller's HttpGet action methods.

* You are running a query that retrieves a large volume of data, and only a small portion of the returned data will be updated. It may be more efficient to turn off tracking for the large query, and run a query later for the few entities that need to be updated.

* You want to attach an entity in order to update it, but earlier you retrieved the same entity for a different purpose. Because the entity is already being tracked by the database context, you can't attach the entity that you want to change. One way to handle this situation is to call `AsNoTracking` on the earlier query.

For more information, see [Tracking vs. No-Tracking](https://docs.microsoft.com/ef/core/querying/tracking).

## Summary

You now have a complete set of pages that perform simple CRUD operations for Student entities. In the next tutorial you'll expand the functionality of the **Index** page by adding sorting, filtering, and paging.

>[!div class="step-by-step"]
[Previous](intro.md)
[Next](sort-filter-page.md)  
