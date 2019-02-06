## Finish the controller
The last step is to finish the controller code. ในตอนนี้ controller ได้รายการสิ่งที่ต้องทำมาจากชั้นบริการ (service layer) และต้องนำรายการต่าง ๆ เข้าไปยัง `TodoViewModel` แล้วจึงนำ model ดังกล่าวผูกเข้ากับ view ที่สร้างไว้ก่อนหน้านี้:

**Controllers/TodoController.cs**

```csharp
public async Task<IActionResult> Index()
{
    var items = await _todoItemService.GetIncompleteItemsAsync();

    var model = new TodoViewModel()
    {
        Items = items
    };

    return View(model);
}
```

If you haven't already, make sure these `using` statements are at the top of the file:

```csharp
using AspNetCoreTodo.Services;
using AspNetCoreTodo.Models;
```

หากคุณใช้ Visual Studio หรือ Visual Studio Code โปรแกรมจะแนะนำการใช้คำสั่ง `using` ข้างต้น เมื่อคุณวางเคอร์เซอร์ไว้บนเส้นหยักสีแดง

## Test it out
กด F5 เพื่อเริ่มการทำงานของแอปพลิเคชัน (หากใช้ Visual Studio หรือ Visual Studio Code) หรือป้อนคำสั่ง `dotnet run` ในเทอร์มินัล หากโค้ดคอมไพล์ได้โดยไม่เกิดข้อผิดพลาด เซอร์ฟเวอร์จะเริ่มทำงานอยู่บนพอร์ต 5000 ซึ่งเป็นค่าตั้งต้นของโปรแกรม

ถ้าเบราว์เซอร์ไม่ได้เปิดขึ้นมาโดยอัตโนมัติ ขอให้เปิดเว็บเบราว์เซอร์ด้วยตนเองแล้วไปที่ http://localhost:5000/todo ก็จะพบกับ view ที่คุณสร้างขึ้น with the data pulled from your fake database (for now).

Although it's possible to go directly to `http://localhost:5000/todo`, it would be nicer to add an item called **My to-dos** to the navbar. To do this, you can edit the shared layout file.
