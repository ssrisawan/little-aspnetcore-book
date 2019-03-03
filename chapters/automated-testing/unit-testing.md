## การทดสอบหน่วยย่อย (Unit testing)

การทดสอบหน่วยย่อยเป็นการทดสอบขนาดเล็กที่กระทำในช่วงสั้น ๆ เพื่อทดสอบพฤติกรรมของหนึ่งเมธอดหรือหนึ่งคลาส หากโค้ดของเราต้องพึ่งพาเมธอดหรือคลาสอื่น ๆ แล้ว การทดสอบหน่วยย่อยจะ **จำลอง (mocking)** คลาสเหล่านั้นขึ้นมาเพื่อให้การทดสอบมุ่งกระทำไปที่ ส่วนใดส่วนหนึ่งในแต่ละการทดสอบเท่านั้น

ตัวอย่างเช่นคลาส `TodoController` มีสองความขึ้นต่อกัน: `ITodoItemService` และ `UserManager` ขณะที่ `TodoItemService` ยังขึ้นอยู่กับ `ApplicationDbContext` อีกด้วย (เส้นเชื่อมที่ลากจาก `TodoController` > `TodoItemService` > `ApplicationDbContext` เรียกได้ว่าเป็น **กราฟแสดงความขึ้นต่อกัน (dependency graph)**)

หากแอปพลิเคชันรันตามปกติ คอนเทนเนอร์บริการและระบบ dependency injection จะฉีดวัตถุเหล่านี้แต่ละตัวเข้าไปในกราฟแสดงความขึ้นต่อกันขณะที่ `TodoController` หรือ `TodoItemService` ถูกสร้างขึ้น

แต่ในทางกลับกัน ขณะที่เราเขียนการทดสอบหน่วยย่อยนั้น เราต้องทำงานกับกราฟแสดงความขึ้นต่อกันด้วยตนเอง โดยทั่วไปจะใช้สิ่งขึ้นต่อกันในเวอร์ชันเพื่อการทดสอบหรือเวอร์ชันที่ถูกจำลองขึ้นมาเท่านั้น ทำให้เราสามารถแยกเฉพาะตรรกะในคลาสหรือเมธอดที่เรากำลังทดสอบอยู่ออกมาได้ (นี่เป็นเรื่องสำคัญ! หากคุณกำลังทดสอบบริการใด ๆ คุณคงไม่ต้องการ **เขียน** ลงในฐานข้อมูลโดยไม่ได้ตั้งใจ)

### สร้างโปรเจกต์เพื่อการทดสอบ

การสร้างโปรเจกต์แยกออกมาเพื่อการทดสอบโดยเฉพาะเป็นแนวปฏิบัติที่ดี เพราะสามารถแยกส่วนออกมาจากโค้ดของแอปพลิเคชันของเราได้ โปรเจกต์ทดสอบที่สร้างขึ้นมาใหม่นี้ควรถูกจัดเก็บไว้ในระดับเดียวกันกับ (ไม่ใช่ภายใต้) ไดเรกทอรีหลักของโปรเจกต์

หากตอนนี้คุณอยู่ในไดเรกทอรีของโปรเจกต์ ให้ `cd` ย้อนขึ้นไปหนึ่งระดับ (ไดเรกทอรีในระดับรากนี้ก็ถูกตั้งชื่อไว้ว่า `AspNetCoreTodo` เช่นกัน) จากนั้นให้ใช้คำสั่งนี้เพื่อขึ้นโครงโปรเจกต์เพื่อการทดสอบ:

```
dotnet new xunit -o AspNetCoreTodo.UnitTests
```

xUnit.NET เป็นเฟรมเวิร์กเพื่อการทดสอบที่นิยมใช้สำหรับโค้ด .NET และสามารถนำมาใช้เพื่อเขียนได้ทั้งการทดสอบหน่วยย่อยและการทดสอบทั้งระบบ โดยอยู่ในรูปแบบของแพคเกจ NuGet เช่นเดียวกับส่วนอื่น ๆ ที่สามารถติดตั้งลงในโปรเจกต์ใดก็ได้ และเทมเพลต `dotnet new xunit` ได้รวมเอาทุกอย่างที่จำเป็นต้องใช้ไว้ให้แล้ว

ในตอนนี้ โครงสร้างไดเรกทอรีของเราควรอยู่ในรูปแบบดังนี้:

```
AspNetCoreTodo/
    AspNetCoreTodo/
        AspNetCoreTodo.csproj
        Controllers/
        (etc...)

    AspNetCoreTodo.UnitTests/
        AspNetCoreTodo.UnitTests.csproj
```

เนื่องจากโปรเจกต์ทดสอบจะใช้คลาสต่าง ๆ ที่ถูกเขียนไว้ในโปรเจกต์หลัก เราจึงจำเป็นต้องเพิ่มการอ้างอิงไปยังโปรเจกต์ `AspNetCoreTodo`:

```
dotnet add reference ../AspNetCoreTodo/AspNetCoreTodo.csproj
```

ให้ลบไฟล์ `UnitTest1.cs` ที่ถูกสร้างขึ้นมาโดยอัตโนมัติ ในตอนนี้เราก็พร้อมที่จะเขียนการทดสอบแรกแล้ว

> หากคุณใช้ Visual Studio Code คุณอาจต้องปิดและเปิดหน้าต่างของ Visual Studio Code อีกครั้งเพื่อให้การเติมโค้ด (code completion) ทำงานได้ในโปรเจกต์ใหม่

### เขียนการทดสอบบริการ

ลองดูที่ตรรกะในเมธอด `AddItemAsync()` ของ `TodoItemService`:

```csharp
public async Task<bool> AddItemAsync(
    TodoItem newItem, ApplicationUser user)
{
    newItem.Id = Guid.NewGuid();
    newItem.IsDone = false;
    newItem.DueAt = DateTimeOffset.Now.AddDays(3);
    newItem.UserId = user.Id;

    _context.Items.Add(newItem);

    var saveResult = await _context.SaveChangesAsync();
    return saveResult == 1;
}
```

เมธอดนี้ตัดสินใจหรือทำตามเงื่อนไขต่าง ๆ หลายประการเกี่ยวกับรายการที่สร้างขึ้นมาใหม่ (หรืออีกนัยหนึ่งคือดำเนินการตามตรรกะธุรกิจกับรายการใหม่) ก่อนที่จะบันทึกลงในฐานข้อมูลจริง:

* คุณสมบัติ `UserId` ควรได้รับการกำหนดค่าให้เป็น ID ของผู้ใช้
* รายการที่สร้างขึ้นใหม่ควรมีสถานะเป็นยังไม่แล้วเสร็จอยู่เสมอ (`IsDone = false`)
* ชื่อเรื่องของรายการใหม่ควรถูกคัดลอกมาจาก `newItem.Title`
* รายการที่สร้างขึ้นใหม่ควรมีวันครบกำหนดเป็น 3 วันนับจากวันนี้เสมอ

ลองจินตนาการว่าหากคุณหรือใครสักคนได้รีแฟคเตอร์เมธอด `AddItemAsync()` แล้วลืมเกี่ยวกับส่วนของตรรกะทางธุรกิจนี้ ก็อาจทำให้พฤติกรรมของแอปพลิเคชันเปลี่ยนไปโดยไม่ได้คาดคิด! เราสามารถป้องกันเหตุการณ์เช่นนี้ได้โดยเขียนการทดสอบให้ตรวจสอบซ้ำอีกครั้งว่าตรรกะทางธุรกิจนี้ไม่ได้ถูกเปลี่ยนแปลงไป (แม้ว่าส่วนของการทำงานภายในของเมธอดจะถูกแก้ไขก็ตาม)

> ถึงแม้ว่าการเปลี่ยนแปลงตรรกะทางธุรกิจโดยไม่คาดคิดนั้นดูไม่น่าจะเกิดขึ้นได้ แต่การคอยจดจำว่าได้ตัดสินใจหรือทำตามเงื่อนไขอะไรไปในโปรเจกต์ขนาดใหญ่ที่มีความซับซ้อนสูงนั้นอาจยากขึ้นมาก ยิ่งโปรเจกต์มีขนาดใหญ่ขึ้นก็ยิ่งจำเป็นต้องมีระบบตรวจสอบอัตโนมัติเพื่อให้แน่ใจว่าไม่มีสิ่งใดถูกเปลี่ยนแปลงไป!

เพื่อเขียนการทดสอบหน่วยย่อยให้ยืนยันตรรกะใน `TodoItemService` เราต้องสร้างคลาสใหม่ขึ้นในโปรเจกต์ทดสอบ:

**AspNetCoreTodo.UnitTests/TodoItemServiceShould.cs**

```csharp
using System;
using System.Threading.Tasks;
using AspNetCoreTodo.Data;
using AspNetCoreTodo.Models;
using AspNetCoreTodo.Services;
using Microsoft.EntityFrameworkCore;
using Xunit;

namespace AspNetCoreTodo.UnitTests
{
    public class TodoItemServiceShould
    {
        [Fact]
        public async Task AddNewItemAsIncompleteWithDueDate()
        {
            // ...
        }
    }
}
```

> มีหลายวิธีที่สามารถนำมาใช้ตั้งชื่อและจัดโครงสร้างในการทดสอบได้โดยแต่ละวิธีก็มีทั้งข้อดีและข้อเสีย แต่ผู้เขียนนิยมนำหน้าชื่อคลาสต่าง ๆ สำหรับการทดสอบด้วย `Should` เพื่อให้เป็นประโยคที่อ่านแล้วสอดคล้องกับชื่อของเมธอดทดสอบ แต่คุณจะใช้แนวทางอย่างไรก็ได้ตามต้องการ!

คุณลักษณะ `[Fact]` มาจากแพคเกจ xUnit.NET และสามารถใช้เพื่อระบุว่าเมธอดนี้เป็นเมธอดทดสอบได้

`TodoItemService` จำเป็นต้องใช้ `ApplicationDbContext` ซึ่งปกติแล้วจะเชื่อมต่ออยู่กับฐานข้อมูล แต่เราไม่ต้องการนำมาใช้ในการทดสอบเช่นนี้ ทั้งนี้ เราสามารถใช้ผู้ให้บริการฐานข้อมูลแบบอยู่ในหน่วยความจำของ Entity Framework Core ในโค้ดของเราแทนได้ เนื่องจากฐานข้อมูลทั้งหมดอยู่ในหน่วยความจำ จึงจะถูกล้างออกทั้งหมดทุกครั้งที่การทดสอบสิ้นสุดและเริ่มใหม่อีกครั้ง และเนื่องจากผู้ให้บริการนี้เป็นส่วนหนึ่งของ Entity Framework Core ดังนั้น `TodoItemService` จึงไม่สามารถบอกความแตกต่างได้!

ใช้ `DbContextOptionsBuilder` เพื่อกำหนดค่าผู้ให้บริการฐานข้อมูลในหน่วยความจำ แล้วเรียกใช้ `AddItemAsync()`:

```csharp
var options = new DbContextOptionsBuilder<ApplicationDbContext>()
    .UseInMemoryDatabase(databaseName: "Test_AddNewItem").Options;

// Set up a context (connection to the "DB") for writing
using (var context = new ApplicationDbContext(options))
{
    var service = new TodoItemService(context);

    var fakeUser = new ApplicationUser
    {
        Id = "fake-000",
        UserName = "fake@example.com"
    };

    await service.AddItemAsync(new TodoItem
    {
        Title = "Testing?"
    }, fakeUser);
}
```

บรรทัดสุดท้ายจะสร้างรายการสิ่งที่ต้องทำในชื่อ  `Testing?` แล้วจึงบอกกับบริการให้บันทึกลงในฐานข้อมูล (ในหน่วยความจำ)

เพื่อยืนยันว่าตรรกะทางธุรกิจทำงานได้อย่างถูกต้อง ให้เขียนโค้ดเพิ่มเติมไว้ใต้บล็อก `using` ดังนี้:

```csharp
// Use a separate context to read data back from the "DB"
using (var context = new ApplicationDbContext(options))
{
    var itemsInDatabase = await context
        .Items.CountAsync();
    Assert.Equal(1, itemsInDatabase);
    
    var item = await context.Items.FirstAsync();
    Assert.Equal("Testing?", item.Title);
    Assert.Equal(false, item.IsDone);

    // Item should be due 3 days from now (give or take a second)
    var difference = DateTimeOffset.Now.AddDays(3) - item.DueAt;
    Assert.True(difference < TimeSpan.FromSeconds(1));
}
```

การยืนยัน (assert) ครั้งแรกเป็นการตรวจสอบความถูกต้อง: เนื่องจากไม่ควรมีข้อมูลมากกว่าหนึ่งรายการถูกบันทึกไว้ในฐานข้อมูล หากเป็นจริง การทดสอบจะรับรายการที่ถูกบันทึกไว้ขึ้นมาด้วย `FirstAsync` แล้วจึงยืนยันว่าคุณสมบัติต่าง ๆ ถูกกำหนดไว้ด้วยค่าตามที่คาดการณ์ไว้

> โดยทั่วไป ทั้งการทดสอบหน่วยย่อยแและการทดสอบทั้งระบบจะทำตามแบบแผน AAA (Arrange-Act-Assert): วัตถุต่าง ๆ และข้อมูลถูกกำหนดค่าเป็นอันดับแรก จากนั้นจึงดำเนินการบางอย่าง และท้ายที่สุดก็จะตรวจสอบหรือยืนยัน (assert) ว่าการทำงานที่ได้เป็นไปตามที่คาดการณ์ไว้

การยืนยันค่าวันที่และเวลา (datetime) ทำได้ยากกว่าเล็กน้อย เนื่องจากการเปรียบเทียบเวลาที่ต่างกันแม้เพียงมิลลิวินาทีก็จะมีผลเป็นไม่เท่ากันได้ ดังนั้น การทดสอบนี้จึงตรวจสอบว่าค่า `DueAt` ต้องต่างจากค่าที่คาดการณ์ไว้ไม่เกินหนึ่งวินาทีแทนที่จะต้องเท่ากัน

### รันการทดสอบ

ให้รันคำสั่งนี้ในเทอร์มินัล (โปรดตรวจสอบว่าคุณยังอยู่ในไดเรกทอรี `AspNetCoreTodo.UnitTests`):

```
dotnet test
```

คำสั่ง `test` จะสแกนหาการทดสอบต่าง ๆ สำหรับโปรเจกต์ (ซึ่งในกรณีนี้ถูกระบุไว้ด้วยคุณลักษณะ `[Fact]`) แล้วจึงรันการทดสอบทั้งหมดที่พบ ซึ่งจะแสดงผลลัพธ์ที่มีลักษณะดังนี้:

```
Starting test execution, please wait...
 Discovering: AspNetCoreTodo.UnitTests
 Discovered:  AspNetCoreTodo.UnitTests
 Starting:    AspNetCoreTodo.UnitTests
 Finished:    AspNetCoreTodo.UnitTests

Total tests: 1. Passed: 1. Failed: 0. Skipped: 0.
Test Run Successful.
Test execution time: 1.9074 Seconds
```

ในตอนนี้เราก็มีการทดสอบที่ครอบคลุม `TodoItemService` แล้ว เพื่อเพิ่มความท้าทาย ลองเขียนการทดสอบหน่วยย่อยเพิ่มเติมเพื่อยืนยันว่า:

* เมธอด `MarkDoneAsync()` คืนค่าเป็นเท็จ หากได้รับค่า ID ที่ไม่มีอยู่จริง
* เมธอด `MarkDoneAsync()` คืนค่าเป็นจริงเมื่อได้กำหนดให้รายการหนึ่งมีสถานะเป็นเสร็จสมบูรณ์
* เมธอด `GetIncompleteItemsAsync()` คืนค่าเฉพาะรายการต่าง ๆ ที่ผู้ใช้รายหนึ่ง ๆ เป็นเจ้าของเท่านั้น
