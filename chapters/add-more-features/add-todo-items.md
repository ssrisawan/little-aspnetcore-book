## เพิ่มรายการสิ่งที่ต้องทำ

ผู้ใช้จะสามารถเพิ่มสิ่งที่ต้องทำขึ้นมาใหม่ได้โดยใช้ฟอร์มง่าย ๆ ใต้รายการสิ่งที่ต้องทำที่มีอยู่:

![Final form](final-form.png)

การเพิ่มความสามารถเช่นนี้จำเป็นต้องใช้หลายขั้นตอน ได้แก่:

* เพิ่มฟอร์มเข้าไปใน view
* สร้างแอคชันใหม่ให้กับ controller เพื่อรองรับฟอร์มดังกล่าว
* เพิ่มโค้ดไปยังชั้นบริการเพื่ออัพเดตฐานข้อมูล

### เพิ่มฟอร์มใหม่

ใน view `Views/Todo/Index.cshtml` มีการจองพื้นที่ไว้สำหรับฟอร์มสำหรับเพิ่มรายการไว้แล้ว:

```html
<div class="panel-footer add-item-form">
  <!-- TODO: Add item form -->
</div>
```

เพื่อเป็นการแยกส่วนงานต่าง ๆ ออกจากกัน เราจะสร้าง view ขึ้นมาเฉพาะส่วนหรือ **partial view** โดยเป็นไฟล์ที่แยกออกมาและทำหน้าที่เป็นส่วนหนึ่งของ view ที่มีขนาดใหญ่กว่า

สร้าง view `AddItemPartial.cshtml`:

**Views/Todo/AddItemPartial.cshtml**

```html
@model TodoItem

<form asp-action="AddItem" method="POST">
    <label asp-for="Title">Add a new item:</label>
    <input asp-for="Title">
    <button type="submit">Add</button>
</form>
```

ตัวช่วยแท็ก `asp-action` สามารถสร้าง URL ขึ้นมาสำหรับฟอร์ม เช่นเดียวกับการใช้ส่วนย่อย `<a>` ในกรณีนี้ ตัวช่วย `asp-action` จะถูกแทนที่ด้วยเส้นทางที่แท้จริงไปยัง `AddItem` ที่กำลังจะถูกสร้างขึ้น:

```html
<form action="/Todo/AddItem" method="POST">
```

การเพิ่มตัวช่วยแท็ก `asp-` ไปยังส่วนย่อย `<form>` ยังช่วยเพิ่มฟิลด์ซ่อนสำหรับเก็บโทเคนเพื่อการตรวจสอบ (verification token) เข้าไปยังฟอร์มด้วย โทเคนเพื่อการตรวจสอบนี้สามารถนำมาใช้เพื่อป้องกันการโจมตีด้วยการปลอมแปลงคำร้องขอแบบข้ามไซต์หรือ cross-site request forgery (CSRF) ได้ โดยเราจะได้ตรวจสอบโทเคนนี้ในขณะที่เขียนแอคชัน

นั่นคือทั้งหมดที่ต้องทำเพื่อสร้าง view เฉพาะส่วนหรือ partial view ซึ่งตอนนี้กำลังจะถูกอ้างถึงจากใน view Todo หลักของเรา:

**Views/Todo/Index.cshtml**

```html
<div class="panel-footer add-item-form">
  @await Html.PartialAsync("AddItemPartial", new TodoItem())
</div>
```

### เพิ่มแอคชัน

เมื่อผู้ใช้คลิก Add บนฟอร์มที่เราเพิ่งสร้างขึ้นมา เบราว์เซอร์จะสร้างคำขอ POST ไปยัง `/Todo/AddItem` ในแอปพลิเคชันของเรา ซึ่งไม่สามารถใช้งานได้ในตอนนี้เนื่องจากยังไม่มีแอคชันที่สามารถดำเนินการกับเส้นทาง `/Todo/AddItem` ได้ หากคุณลองคลิกตอนนี้ ASP.NET Core จะแสดงข้อผิดพลาด `404 Not Found` ขึ้นมา

เราจำเป็นต้องสร้างแอคชันในชื่อ `AddItem` ไว้บน `TodoController`:

```csharp
[ValidateAntiForgeryToken]
public async Task<IActionResult> AddItem(TodoItem newItem)
{
    if (!ModelState.IsValid)
    {
        return RedirectToAction("Index");
    }

    var successful = await _todoItemService.AddItemAsync(newItem);
    if (!successful)
    {
        return BadRequest("Could not add item.");
    }

    return RedirectToAction("Index");
}
```

Notice how the new `AddItem` action accepts a `TodoItem` parameter? This is the same `TodoItem` model you created in the _MVC basics_ chapter to store information about a to-do item. When it's used here as an action parameter, ASP.NET Core will automatically perform a process called **model binding**.

Model binding looks at the data in a request and tries to intelligently match the incoming fields with properties on the model. In other words, when the user submits this form and their browser POSTs to this action, ASP.NET Core will grab the information from the form and place it in the `newItem` variable.

The `[ValidateAntiForgeryToken]` attribute before the action tells ASP.NET Core that it should look for (and verify) the hidden verification token that was added to the form by the `asp-action` tag helper. This is an important security measure to prevent cross-site request forgery (CSRF) attacks, where your users could be tricked into submitting data from a malicious site. The verification token ensures that your application is actually the one that rendered and submitted the form.

Take a look at the `AddItemPartial.cshtml` view once more. The `@model TodoItem` line at the top of the file tells ASP.NET Core that the view should expect to be paired with the `TodoItem` model. This makes it possible to use `asp-for="Title"` on the `<input>` tag to let ASP.NET Core know that this input element is for the `Title` property.

Because of the `@model` line, the partial view will expect to be passed a `TodoItem` object when it's rendered. Passing it a `new TodoItem` via `Html.PartialAsync` initializes the form with an empty item. (Try appending `{ Title = "hello" }` and see what happens!)

During model binding, any model properties that can't be matched up with fields in the request are ignored. Since the form only includes a `Title` input element, you can expect that the other properties on `TodoItem` (the `IsDone` flag, the `DueAt` date) will be empty or contain default values.

> Instead of reusing the `TodoItem` model, another approach would be to create a separate model (like `NewTodoItem`) that's only used for this action and only has the specific properties (Title) you need for adding a new to-do item. Model binding is still used, but this way you've separated the model that's used for storing a to-do item in the database from the model that's used for binding incoming request data. This is sometimes called a **binding model** or a **data transfer object** (DTO). This pattern is common in larger, more complex projects.

After binding the request data to the model, ASP.NET Core also performs **model validation**. Validation checks whether the data bound to the model from the incoming request makes sense or is valid. You can add attributes to the model to tell ASP.NET Core how it should be validated.

The `[Required]` attribute on the `Title` property tells ASP.NET Core's model validator to consider the title invalid if it is missing or blank. Take a look at the code of the `AddItem` action: the first block checks whether the `ModelState` (the model validation result) is valid. It's customary to do this validation check right at the beginning of the action:

```csharp
if (!ModelState.IsValid)
{
    return RedirectToAction("Index");
}
```

If the `ModelState` is invalid for any reason, the browser will be redirected to the `/Todo/Index` route, which refreshes the page.

Next, the controller calls into the service layer to do the actual database operation of saving the new to-do item:

```csharp
var successful = await _todoItemService.AddItemAsync(newItem);
if (!successful)
{
    return BadRequest(new { error = "Could not add item." });
}
```

The `AddItemAsync` method will return `true` or `false` depending on whether the item was successfully added to the database. If it fails for some reason, the action will return an HTTP `400 Bad Request` error along with an object that contains an error message.

Finally, if everything completed without errors, the action redirects the browser to the `/Todo/Index` route, which refreshes the page and displays the new, updated list of to-do items to the user.

### Add a service method

If you're using a code editor that understands C#, you'll see red squiggely lines under `AddItemAsync` because the method doesn't exist yet.

As a last step, you need to add a method to the service layer. First, add it to the interface definition in `ITodoItemService`:

```csharp
public interface ITodoItemService
{
    Task<TodoItem[]> GetIncompleteItemsAsync();

    Task<bool> AddItemAsync(TodoItem newItem);
}
```

Then, the actual implementation in `TodoItemService`:

```csharp
public async Task<bool> AddItemAsync(TodoItem newItem)
{
    newItem.Id = Guid.NewGuid();
    newItem.IsDone = false;
    newItem.DueAt = DateTimeOffset.Now.AddDays(3);

    _context.Items.Add(newItem);

    var saveResult = await _context.SaveChangesAsync();
    return saveResult == 1;
}
```

The `newItem.Title` property has already been set by ASP.NET Core's model binder, so this method only needs to assign an ID and set the default values for the other properties. Then, the new item is added to the database context. It isn't actually saved until you call `SaveChangesAsync()`. If the save operation was successful, `SaveChangesAsync()` will return 1.

### Try it out

Run the application and add some items to your to-do list with the form. Since the items are being stored in the database, they'll still be there even after you stop and start the application again.

> As an extra challenge, try adding a date picker using HTML and JavaScript, and let the user choose an (optional) date for the `DueAt` property. Then, use that date instead of always making new tasks that are due in 3 days.
