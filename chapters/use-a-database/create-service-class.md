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

สังเกตได้ว่าคุณเคยเห็นรูปแบบของการทำ dependency injection มาแล้วในบท *พื้นฐานของ MVC* เพียงแต่ในครั้งนี้สิ่งที่ถูกฉีดเข้าไปกลับกลายเป็น `ApplicationDbContext` แทน โดย `ApplicationDbContext` นี้ได้ถูกเพิ่มเข้าไปในคอนเทนเนอร์บริการเรียบร้อยแล้วในเมธอด `ConfigureServices` ดังนั้นจึงพร้อมสำหรับการฉีดเข้าไปในที่นี้

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

### อัพเดตคอนเทนเนอร์บริการ

เนื่องจากคลาส `FakeTodoItemService` ถูกลบไปแล้ว เราจึงจำเป็นต้องอัพเดตบรรทัดใน `ConfigureServices` ที่เชื่อมโยงอินเตอร์เฟส `ITodoItemService` ให้เป็นดังนี้:

```csharp
services.AddScoped<ITodoItemService, TodoItemService>();
```

`AddScoped` เพิ่มบริการของเราลงในคอนเทนเนอร์บริการให้มีวงจรชีวิตแบบ **scoped** นั่นหมายความว่าอินสแตนซ์ใหม่ของคลาส `TodoItemService` จะถูกสร้างขึ้นมาระหว่างการร้องขอเว็บในแต่ละครั้ง ซึ่งวิธีนี้เป็นสิ่งจำเป็นสำหรับคลาสบริการต่าง ๆ ที่ต้องสื่อสารกับฐานข้อมูล

> การเพิ่มคลาสบริการที่ต้องสื่อสารกับ Entity Framework Core (และฐานข้อมูลของคุณ) ด้วยวงจรชีวิตแบบ singleton (หรือแบบอื่น ๆ) อาจทำให้เกิดปัญหาได้ เนื่องจากวิธีที่ Entity Framework Core ใช้จัดการการเชื่อมต่อกับฐานข้อมูลอยู่ในเบื้องหลังสำหรับแต่ละการร้องขอ เพื่อหลีกเลี่ยงปัญหาดังกลา่ว ขอให้ใช้วงจรชีวิตแบบ scoped ทุกครั้งสำหรับบริการที่ต้องสื่อสารกับ Entity Framework Core

`TodoController` ที่ต้องพึ่งพา `ITodoItemService` ที่ถูกฉีดเข้ามาจะไม่ได้รับรู้ถึงการเปลี่ยนแปลงในคลาสบริการนี้เลย แต่ในเบื้องหลังแล้วจะเป็นการเปลี่ยนไปใช้ Entity Framework Core และได้สื่อสารกับฐานข้อมูลจริง!

### มาทดสอบกัน

ให้เริ่มการทำงานของแอปพลิเคชันแล้วเปิดไปที่ `http://localhost:5000/todo` ในตอนนี้ รายการปลอม ๆ จะหายไปแล้ว แต่แอปพลิเคชันของเราได้คิวรีข้อมูลไปยังฐานข้อมูลจริง ๆ แต่เนื่องจากยังไม่มีข้อมูลสิ่งที่ต้องทำใด ๆ ถูกบันทึกเอาไว้ รายการของเราจึงยังว่างเปล่าอยู่ในตอนนี้

ในบทต่อไป เราจะได้เพิ่มศักยภาพให้กับแอปพลิเคชันของเรา โดยเริ่มจากการเพิ่มความสามารถในการสร้างรายการสิ่งที่ต้องทำขึ้นมา
