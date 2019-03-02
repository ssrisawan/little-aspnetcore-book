## การใช้อัตลักษณ์ในแอปพลิเคชัน

ในตอนนี้ ผู้ใช้ทั้งหมดใช้รายการสิ่งที่ต้องทำร่วมกันเนื่องจากแต่ละรายการที่จัดเก็บไว้ไม่ได้ผูกกับผู้ใช้รายใดรายหนึ่งโดยเฉพาะ แต่เนื่องจากคุณลักษณะ `[Authorize]` กำหนดให้ต้องล็อกอินก่อนจึงจะเห็นรายการสิ่งที่ต้องทำ เราจึงสามารถกรองคิวรีฐานข้อมูลตามผู้ใช้ที่ล็อกอินเข้าระบบได้

อันดับแรก ฉีด `UserManager<ApplicationUser>` เข้าไปยัง `TodoController`:

**Controllers/TodoController.cs**

```csharp
[Authorize]
public class TodoController : Controller
{
    private readonly ITodoItemService _todoItemService;
    private readonly UserManager<ApplicationUser> _userManager;

    public TodoController(ITodoItemService todoItemService,
        UserManager<ApplicationUser> userManager)
    {
        _todoItemService = todoItemService;
        _userManager = userManager;
    }

    // ...
}
```

เราต้องเพิ่มคำสั่ง `using` ไว้ตอนต้นของโค้ด:

```csharp
using Microsoft.AspNetCore.Identity;
```

คลาส `UserManager` เป็นส่วนหนึ่งของระบบอัตลักษณ์ใน ASP.NET Core ซึ่งเราสามารถใช้เพื่อรับค่าผู้ใช้ปัจจุบันในแอคชัน `Index` ได้:

```csharp
public async Task<IActionResult> Index()
{
    var currentUser = await _userManager.GetUserAsync(User);
    if (currentUser == null) return Challenge();

    var items = await _todoItemService
        .GetIncompleteItemsAsync(currentUser);

    var model = new TodoViewModel()
    {
        Items = items
    };

    return View(model);
}
```

โค้ดใหม่ที่ตอนต้นของเมธอดแอคชันนี้ใช้ `UserManager` เพื่อเรียกดูผู้ใช้ปัจจุบันจากคุณสมบัติ `User` ที่อยู่ในแอคชัน:

```csharp
var currentUser = await _userManager.GetUserAsync(User);
```

หากมีผู้ใช้ล็อกอินอยู่ คุณสมบัติ `User` จะมีวัตถุขนาดเบา (lightweight object) ที่มีข้อมูลบางส่วนของผู้ใช้ ซึ่ง `UserManager` จะใช้เพื่อเรียกดูรายละเอียดทั้งหมดของผู้ใช้ขึ้นมาจากฐานข้อมูลผ่านเมธอด `GetUserAsync()`

ค่าของ `currentUser` ไม่ควรเป็นค่าว่าง (null) ได้ เนื่องจากคุณสมบัติ `[Authorize]` ถูกใช้อยู่กับ controller อย่างไรก็ตาม เราควรตรวจสอบเพื่อป้องกันไว้ก่อนอยู่เสมอด้วยการใช้เมธอด `Challenge()` เพื่อบังคับให้ผู้ใช้ต้องล็อกอินอีกครั้งหากข้อมูลของผู้ใช้มีไม่ครบ:

```csharp
if (currentUser == null) return Challenge();
```

เนื่องจากเราจะส่งพารามิเตอร์ `ApplicationUser` ให้กับ `GetIncompleteItemsAsync()` เราจึงจำเป็นต้องอัพเดตอินเตอร์เฟส `ITodoItemService`:

**Services/ITodoItemService.cs**

```csharp
public interface ITodoItemService
{
    Task<TodoItem[]> GetIncompleteItemsAsync(
        ApplicationUser user);
    
    // ...
}
```

เนื่องจากเราได้แก้ไขอินเตอร์เฟส `ITodoItemService` เราจึงจำเป็นต้องอัพเดตลายเซ็นของเมธอด `GetIncompleteItemsAsync()` ใน `TodoItemService` ด้วย:

**Services/TodoItemService**

```csharp
public async Task<TodoItem[]> GetIncompleteItemsAsync(
    ApplicationUser user)
```

ขั้นตอนต่อไปคือการอัพเดตคิวรีฐานข้อมูลและเพิ่มตัวกรองเพื่อให้แสดงเฉพาะรายการที่ถูกสร้างโดยผู้ใช้ปัจจุบัน ซึ่งก่อนจะทำเช่นนั้นได้ เราต้องเพิ่มคุณสมบัติใหม่เข้าไปในฐานข้อมูล

### อัพเดตฐานข้อมูล

เราจำเป็นต้องเพิ่มคุณสมบัติใหม่เข้าไปในโมเดลเอนทิตี `TodoItem` เพื่อให้แต่ละรายการสามารถ "จำ" ผู้ใช้ที่เป็นเจ้าของได้:

**Models/TodoItem.cs**

```csharp
public string UserId { get; set; }
```

เนื่องจากเราได้อัพเดตโมเดลเอนทิตีที่ถูกใช้โดยบริบทฐานข้อมูลแล้ว เราจึงจำเป็นต้องโยกย้ายฐานข้อมูลดโดยใช้คำสั่ง `dotnet ef` ในเทอร์มินัล:

```
dotnet ef migrations add AddItemUserId
```

คำสั่งนี้จะสร้างการโยกย้ายใหม่ในชื่อ `AddItemUserId` ซึ่งจะเพิ่มคอลัมน์ใหม่ในตาราง `Items` ให้สอดคล้องกับความเปลี่ยนแปลงที่เราทำไว้กับโมเดล `TodoItem`

รัน `dotnet ef` อีกครั้งเพื่อนำไปใช้กับฐานข้อมูล:

```
dotnet ef database update
```

### อัพเดตคลาสบริการ

หลังจากได้ปอัพเดตฐานข้อมูลและบริบทของฐานข้อมูลเรียบร้อยแล้ว เราสามารถอัพเดตเมธอด `GetIncompleteItemsAsync()` ใน `TodoItemService` และเพิ่มอีกวรรคหนึ่งเข้าไปยังคำสั่ง `Where`:

**Services/TodoItemService.cs**

```csharp
public async Task<TodoItem[]> GetIncompleteItemsAsync(
    ApplicationUser user)
{
    return await _context.Items
        .Where(x => x.IsDone == false && x.UserId == user.Id)
        .ToArrayAsync();
}
```

หลังจากรันแอปพลิเคชันแล้วลงทะเบียนหรือล็อกอินแล้ว เราจะพบรายการสิ่งที่ต้องทำที่ว่างเปล่าอีกครั้ง แต่โชคร้ายที่รายการใด ๆ ที่เคยเพิ่มไว้ก่อนหน้านี้จะหายไปกลายเป็นอากาศธาตุ เนื่องจากเรายังไม่ได้อัพเดตแอคชัน `AddItem` ให้รับรู้ถึงการกำหนดผู้ใช้

### อัพเดตแอคชัน AddItem และ MarkDone

เราจำเป็นต้องใช้ `UserManager` เพื่อรับค่าผู้ใช้ปัจจุบันในเมธอดแอคชัน `AddItem` และ `MarkDone` เหมือนกับที่ทำไว้กับ `Index`

ทั้งสองเมธอดที่ถูกอัพเดตไว้จะเป็นดังนี้:

**Controllers/TodoController.cs**

```csharp
[ValidateAntiForgeryToken]
public async Task<IActionResult> AddItem(TodoItem newItem)
{
    if (!ModelState.IsValid)
    {
        return RedirectToAction("Index");
    }

    var currentUser = await _userManager.GetUserAsync(User);
    if (currentUser == null) return Challenge();

    var successful = await _todoItemService
        .AddItemAsync(newItem, currentUser);

    if (!successful)
    {
        return BadRequest("Could not add item.");
    }

    return RedirectToAction("Index");
}

[ValidateAntiForgeryToken]
public async Task<IActionResult> MarkDone(Guid id)
{
    if (id == Guid.Empty)
    {
        return RedirectToAction("Index");
    }

    var currentUser = await _userManager.GetUserAsync(User);
    if (currentUser == null) return Challenge();

    var successful = await _todoItemService
        .MarkDoneAsync(id, currentUser);
    
    if (!successful)
    {
        return BadRequest("Could not mark item as done.");
    }

    return RedirectToAction("Index");
}
```

ในตอนนี้ เมธอดบริการทั้งสองจะต้องรับค่าพารามิเตอร์ `ApplicationUser` ให้อัพเดตข้อกำหนดของอินเตอร์เฟสใน `ITodoItemService` ดังนี้:

```csharp
Task<bool> AddItemAsync(TodoItem newItem, ApplicationUser user);

Task<bool> MarkDoneAsync(Guid id, ApplicationUser user);
```

และท้ายที่สุด ให้อัพเดตการทำงานของเมธอดบริการ `TodoItemService` ในเมธอด `AddItemAsync` ให้กำหนดคุณสมบัติ `UserId` ขณะสร้าง `new TodoItem`:

```csharp
public async Task<bool> AddItemAsync(
    TodoItem newItem, ApplicationUser user)
{
    newItem.Id = Guid.NewGuid();
    newItem.IsDone = false;
    newItem.DueAt = DateTimeOffset.Now.AddDays(3);
    newItem.UserId = user.Id;

    // ...
}
```

ประโยค `Where` ในเมธอด `MarkDoneAsync` ยังจำเป็นต้องตรวจสอบ ID ของผู้ใช้ เพื่อไม่ให้ผู้ไม่หวังดีสามารถกำหนดว่างานของผู้ใช้รายอื่นเสร็จแล้วได้จากการเดา ID ของผู้ใช้เหล่านั้น:

```csharp
public async Task<bool> MarkDoneAsync(
    Guid id, ApplicationUser user)
{
    var item = await _context.Items
        .Where(x => x.Id == id && x.UserId == user.Id)
        .SingleOrDefaultAsync();

    // ...
}
```

เรียบร้อยแล้ว! ลองใช้แอปพลิเคชันกับสองผู้ใช้ที่แตกต่างกันดู จะพบว่ารายการสิ่งที่ต้องทำถูกจำกัดไว้เป็นส่วนตัวสำหรับแต่ละผู้ใช้เท่านั้น
