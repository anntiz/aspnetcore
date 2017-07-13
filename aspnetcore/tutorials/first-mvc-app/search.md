# Adding Search  
添加搜索功能

[!INCLUDE[adding-search](../../includes/mvc-intro/search1.md)]

You can quickly rename the `searchString` parameter to `id` with the **rename** command. Right click on `searchString` **> Rename**.  
你可以使用 **rename(重命名)** 命令来将 `searchString` 参数快速重命名为 `id`， 鼠标在 `searchString` 上单击右键， 选择 **> Rename(重合名)**


![Contextual menu](search/_static/rename.png)

The rename targets are highlighted.  
重命名的目标被突出显示

![Code editor showing the variable highlighted throughout the Index ActionResult method](search/_static/rename2.png)

Change the parameter to `id` and all occurrences of `searchString` change to `id`.  
修改参数名称为 `id` 并将所有匹配的 `searchString` 更改为 `id`。

![Code editor showing the variable has been changed to id](search/_static/rename3.png)

[!INCLUDE[adding-model](../../includes/mvc-intro/search2.md)]

Notice how intelliSense helps us update the markup.  
请注意智能感知是如何帮助我们更新标记的。

![Intellisense contextual menu with method selected in the list of attributes for the form element](search/_static/int_m.png)

![Intellisense contextual menu with get selected in the list of method attribute values](search/_static/int_get.png)

Notice the distinctive font in the `<form>` tag. That distinctive font indicates the tag is supported by [Tag Helpers](../../mvc/views/tag-helpers/intro.md).  
注意 `<form>` 标签中的不同字体，这些不同的字体表明这个标签是由 [Tag Helpers](../../mvc/views/tag-helpers/intro.md) 支持的。

![form tag with purple text](search/_static/th_font.png)

[!INCLUDE[adding-model](../../includes/mvc-intro/search3.md)]

>[!div class="step-by-step"]
[前一页](controller-methods-views.md)
[后一页](new-field.md)  
