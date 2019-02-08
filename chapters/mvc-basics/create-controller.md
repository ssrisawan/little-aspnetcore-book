## การสร้าง controller

ขณะนี้ โปรเจกต์ของเรามี controller บางตัวอยู่ในไดเรกทอรี Controllers อยู่แล้ว เช่น `HomeController` ที่ทำหน้าที่สร้างหน้าเว็บตั้งต้นที่แสดงให้เห็นเมื่อเข้าไปที่ `http://localhost:5000` ในเบื้องต้น เราจะยังไม่ดำเนินการใด ๆ กับ controller ที่มีอยู่แล้วเหล่านี้

สร้าง controller ใหม่สำหรับปฏิบัติการกับรายการสิ่งที่ต้องทำ โดยให้ตั้งชื่อว่า `TodoController` แล้วให้เพิ่มโค้ดต่อไปนี้:

**Controllers/TodoController.cs**

``` csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;

namespace AspNetCoreTodo.Controllers
{
    public class TodoController : Controller
    {
        // Actions go here
    }
}
```

เส้นทางต่าง ๆ (route) ที่รองรับโดย controller เรียกว่า **action** และถูกแทนด้วยเมธอดในคลาส controller ตัวอย่างเช่น `HomeController` มีสามเมธอด action (`Index`, `About` และ `Contact`) ที่ ASP.NET Core จับคู่เข้ากับเส้นทาง URL ต่อไปนี้:

```
localhost:5000/Home         -> Index()
localhost:5000/Home/About   -> About()
localhost:5000/Home/Contact -> Contact()
```

ใน ASP.NET Core มีธรรมเนียมปฏิบัติ (รูปแบบต่าง ๆ ที่ใช้บ่อย) อยู่จำนวนหนึ่ง เช่นจากชื่อ `FooController` จะกลายเป็น `/Foo` และชื่อ action `Index` สามารถละไว้จาก URL ได้ เป็นต้น โดยผู้ใช้สามารถปรับแต่งพฤติกรรมเหล่านี้ได้ตามต้องการ อย่างไรก็ตาม ในขั้นตอนนี้ เราจะใช้ธรรมเนียมปฏิบัติดังกล่าวข้างต้นกันก่อน

ให้เพิ่ม action ในชื่อ `Index` เข้าไปยัง `TodoController` และแทนที่หมายเหตุ  `// Actions go here` ด้วยโค้ดต่อไปนี้:

```csharp
public class TodoController : Controller
{
    public IActionResult Index()
    {
        // Get to-do items from database

        // Put items into a model

        // Render view using the model
    }
}
```

เมธอด action สามารถคืนค่าเป็น view, ข้อมูล JSON หรือรหัสสถานะ HTTP เช่น `200 OK` และ `404 Not Found` ซึ่งชนิดของค่าที่ถูกส่งคืนจาก `IActionResult` มีความยืดหยุ่นและสามารถส่งคืนค่าต่าง ๆ ดังกล่าวข้างต้นได้

ตามแนวปฏิบัติที่ดีแล้ว controller ต่าง ๆ ควรทำงานน้อยที่สุดที่เป็นไปได้ เช่นในกรณีนี้ controller จะมีหน้าที่รับผิดชอบต่อการนำรายการสิ่งที่ต้องทำมาจากฐานข้อมูล แล้วนำรายการต่าง ๆ มาอยู่ใน model ที่สามารถเข้าใจได้โดย view จากนั้นจึงส่ง view กลับไปยังเบราว์เซอร์ของผู้ใช้

ก่อนที่จะเริ่มเขียนโค้ดส่วนที่เหลือของ controller ได้นั้น คุณจำเป็นต้องสร้าง model and และ view ขึ้นมาเสียก่อน
