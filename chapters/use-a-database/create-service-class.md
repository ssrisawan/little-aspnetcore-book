## สร้างคลาสบริการ

ในบท *พื้นฐานของ MVC* เราได้สร้าง `FakeTodoItemService` ที่ได้กำหนดรายการสิ่งที่ต้องทำไว้โดยตรงในโค้ด แต่ในตอนนี้เนื่องจากเรามีบริบทฐานข้อมูลแล้ว เราสามารถสร้างคลาสบริการขึ้นมาใหม่ซึ่งจะใช้ Entity Framework Core เพื่อรับรายการต่าง ๆ มาจากฐานข้อมูลจริง

ให้ลบไฟล์ `FakeTodoItemService.cs` แล้วสร้างไฟล์ขึ้นมาใหม่ดังนี้:

**Services/TodoItemService.cs**

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using AspNetCoreTodo.Data;
using AspNetCoreTodo.Models;
using Microsoft.EntityFrameworkCore;

namespace AspNetCoreTodo.Services
{
    public class TodoItemService : ITodoItemService
    {
        private readonly ApplicationDbContext _context;

        public TodoItemService(ApplicationDbContext context)
        {
            _context = context;
        }

        public async Task<TodoItem[]> GetIncompleteItemsAsync()
        {
            var items = await _context.Items
                .Where(x => x.IsDone == false)
                .ToArrayAsync();
            return items;
        }
    }
}
```

You'll notice the same dependency injection pattern here that you saw in the *MVC basics* chapter, except this time it's the `ApplicationDbContext` that's getting injected. The `ApplicationDbContext` is already being added to the service container in the `ConfigureServices` method, so it's available for injection here.

เรามาดูที่โค้ดของเมธอด `GetIncompleteItemsAsync` ให้ใกล้ชิดกันขึ้นอีกหน่อย อันดับแรก เมธอดนี้ใช้คุณสมบัติ `Items` ของบริบทเพื่อเข้าถึงรายการสิ่งที่ต้องทำทั้งหมดใน `DbSet`:

```csharp
var items = await _context.Items
```

จากนั้น เมธอด `Where` จะถูกใช้เพื่อกรองเฉพาะรายการที่ยังไม่แล้วเสร็จขึ้นมา:

```csharp
.Where(x => x.IsDone == false)
```

เมธอด `Where` นี้เป็นความสามารถของ C# ที่เรียกว่า LINQ (**l**anguage **in**tegrated **q**uery) ซึ่งได้รับแรงบันดาลใจมาจากการเขียนโปรแกรมเชิงฟังก์ชัน (functional programming) และทำให้สามารถแสดงถึงการคิวรีฐานข้อมูลไว้ในโค้ดได้โดยง่าย ในที่นี้ Entity Framework Core จะดำเนินการอยู่เบื้องหลัง เพื่อแปลเมธอด `Where` ให้อยู่ในรูปคำสั่งเช่น `SELECT * FROM Items WHERE IsDone = 0` หรือให้เป็นเอกสารคิวรีที่ทำหน้าที่เช่นเดียวกันในฐานข้อมูล NoSQL

ท้ายที่สุด เมธอด `ToArrayAsync` จะบอก Entity Framework Core ให้รับเอนทิตีทั้งหมดที่สอดคล้องกับตัวกรองและคืนค่ากลับมาเป็นอาร์เรย์ โดยเมธอด `ToArrayAsync` นี้ทำงานแบบไม่ประสานจังหวะ (คืนค่าเป็น `Task`) ดังนั้นจึงจำเป็นต้องกำหนดให้รอหรือ  `await` เพื่อรับค่าดังกล่าว

เพื่อให้เมธอดสั้นลงได้อีกเล้กน้อย คุณสามารถลบตัวแปร `items` ที่ถูกใช้เป็นตัวกลางเก็บข้อมูลออกได้ แล้วให้คืนค่าผลลัพธ์ของการคิวรีโดยตรง (ซึ่งจะให้ผลเหมือนกัน):

```csharp
public async Task<TodoItem[]> GetIncompleteItemsAsync()
{
    return await _context.Items
        .Where(x => x.IsDone == false)
        .ToArrayAsync();
}
```

### Update the service container

Because you deleted the `FakeTodoItemService` class, you'll need to update the line in `ConfigureServices` that is wiring up the `ITodoItemService` interface:

```csharp
services.AddScoped<ITodoItemService, TodoItemService>();
```

`AddScoped` adds your service to the service container using the **scoped** lifecycle. This means that a new instance of the `TodoItemService` class will be created during each web request. This is required for service classes that interact with a database.

> Adding a service class that interacts with Entity Framework Core (and your database) with the singleton lifecycle (or other lifecycles) can cause problems, because of how Entity Framework Core manages database connections per request under the hood. To avoid that, always use the scoped lifecycle for services that interact with Entity Framework Core.

The `TodoController` that depends on an injected `ITodoItemService` will be blissfully unaware of the change in services classes, but under the hood it'll be using Entity Framework Core and talking to a real database!

### Test it out

Start up the application and navigate to `http://localhost:5000/todo`. The fake items are gone, and your application is making real queries to the database. There doesn't happen to be any saved to-do items, so it's blank for now.

In the next chapter, you'll add more features to the application, starting with the ability to create new to-do items.
