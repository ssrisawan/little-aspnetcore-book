## ทำ controller ให้แล้วเสร็จ
ขั้นตอนสุดท้ายคือการจัดการกับโค้ดของ controller ให้แล้วเสร็จ ในตอนนี้ controller ได้รายการสิ่งที่ต้องทำมาจากชั้นบริการแล้ว จากนั้น จะต้องนำรายการต่าง ๆ ป้อนให้ `TodoViewModel` จากนั้นจึงนำ model ดังกล่าวผูกเข้ากับ view ที่สร้างไว้ก่อนหน้านี้:

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

ตรวจสอบให้แน่ใจว่ามีคำสั่ง `using` เหล่านี้อยู่ตอนต้นของไฟล์:

```csharp
using AspNetCoreTodo.Services;
using AspNetCoreTodo.Models;
```

หากคุณใช้ Visual Studio หรือ Visual Studio Code โปรแกรมจะแนะนำการใช้คำสั่ง `using` ข้างต้น เมื่อคุณวางเคอร์เซอร์ไว้บนเส้นหยักสีแดง

## มาลองทดสอบกัน
กด F5 เพื่อเริ่มการทำงานของแอปพลิเคชัน (หากใช้ Visual Studio หรือ Visual Studio Code) หรือป้อนคำสั่ง `dotnet run` ในเทอร์มินัล หากโค้ดคอมไพล์ได้โดยไม่เกิดข้อผิดพลาด เซอร์ฟเวอร์จะเริ่มทำงานอยู่บนพอร์ต 5000 ซึ่งเป็นค่าตั้งต้นของโปรแกรม

ถ้าเบราว์เซอร์ไม่ได้เปิดขึ้นมาโดยอัตโนมัติ ขอให้เปิดเว็บเบราว์เซอร์ด้วยตนเองแล้วไปที่ http://localhost:5000/todo ก็จะพบกับ view ที่คุณสร้างขึ้นพร้อมกับข้อมูลที่ถูกดึงขึ้นมาจากฐานข้อมูลหลอก ๆ (สำหรับตอนนี้)

แม้จะสามารถเปิดไปยัง `http://localhost:5000/todo` โดยตรงได้ แต่คงจะดีกว่าหากมีรายการที่เรียกว่า **My to-dos** อยู่บนแถบนำทาง (navbar) ด้วย ซึ่งสามารถทำได้ด้วยการแก้ไขไฟล์เลย์เอาต์ที่ใช้ร่วมกัน
