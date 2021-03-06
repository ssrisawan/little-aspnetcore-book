# การเพิ่มแพคเกจภายนอก
หนึ่งในข้อดีของการใช้ระบบนิเวศ (ecosystem) ที่เติบโตเต็มที่แล้วเช่น .NET คือการมีแพคเกจและปลั๊กอินจากภายนอกเป็นจำนวนมาก และเช่นเดียวกับระบบจัดการแพคเกจอื่น ๆ คุณสามารถดาวน์โหลดและติดตั้งแพคเกจ .NET ต่าง ๆ สำหรับงานแต่ละด้านหรือสำหรับปัญหาใด ๆ ที่คุณต้องแก้ไขได้เกือบทั้งหมด

NuGet เป็นทั้งเครื่องมือจัดการแพคเกจและเป็นคลังเก็บแพคเกจอย่างเป็นทางการอีกด้วย (อยู่ที่ https://www.nuget.org) คุณสามารถค้นหาแพคเกจ NuGet ที่ต้องการได้จากบนเว็บแล้วติดตั้งลงในเครื่องของคุณผ่านทางเทอร์มินัล (หรือ GUI หากใช้ Visual Studio)

## ติดตั้งแพคเกจ Humanizer
ในตอนท้ายของบทที่แล้ว แอปพลิเคชันสิ่งที่ต้องทำแสดงรายการสิ่งที่ต้องทำดังนี้:

![Dates in ISO 8601 format](iso8601.png)

คอลัมน์วันครบกำหนดแสดงวันที่ในรูปแบบที่เข้าใจง่ายสำหรับคอมพิวเตอร์ (เรียกว่า ISO 8601) แต่ซับซ้อนเกินไปสำหรับมนุษย์ จะดีกว่านี้ไหมหากให้แสดงข้อความที่เข้าใจง่ายเช่น "อีก X วันนับจากนี้"?

เราอาจเขียนโค้ดให้แปลงวันที่ ISO 8601 ให้อยู่ในรูปแบบสตริงที่เข้าใจง่ายสำหรับมนุษย์ด้วยตนเองก็ได้ แต่ยังมีวิธีที่เร็วกว่านี้

แพคเกจ Humanizer บน NuGet ช่วยแก้ปัญหานี้ด้วยการให้บริการเมธอดที่สามารถทำให้ "เหมาะกับมนุษย์" หรือช่วยเขียนรูปแบบของเวลา วันที่ ระยะเวลา และอื่น ๆ ขึ้นมาใหม่ได้ โปรเจกต์ที่ยอดเยี่ยมและเป็นประโยชน์นี้ ถูกเผยแพร่ภายใต้ไลเซนส์โอเพนซอร์สที่ให้อิสระเปิดกว้างอย่าง MIT อีกด้วย

ให้รันคำสั่งนี้ในเทอร์มินัลเพื่อติดตั้งแพคเกจลงในโปรเจกต์ของคุณ:

```
dotnet add package Humanizer
```

หากลองดูในไฟล์โปรเจกต์ `AspNetCoreTodo.csproj` คุณจะพบบรรทัด `PackageReference` ถูกเพิ่มขึ้นมาใหม่โดยอ้างถึง `Humanizer`

## การใช้ Humanizer ภายใน view

เพื่อใช้แพคเกจในโค้ด เรามักต้องเพิ่มคำสั่ง `using` เพื่อนำเข้าแพคเกจที่ตอนต้นของไฟล์

เนื่องจาก Humanizer ถูกนำมาใช้เขียนรูปแบบของวันที่ขึ้นมาใหม่ขณะสร้าง view เราจึงสามารถเรียกใช้ใน view ได้โดยตรง โดยขั้นแรกจะต้องเพิ่มคำสั่ง `@using` ที่ตอนต้นของ view:

**Views/Todo/Index.cshtml**

```html
@model TodoViewModel
@using Humanizer

// ...
```

จากนั้นให้แก้ไขบรรทัดที่แสดงคุณสมบัติ `DueAt` โดยเปลี่ยนไปใช้เมธอด `Humanize` ของแพคเกจ Humanizer:

```html
<td>@item.DueAt.Humanize()</td>
```

ในตอนนี้ วันที่ต่าง ๆ สามารถอ่านได้ง่ายขึ้นมากแล้ว:

![Human-readable dates](friendly-dates.png)

ใน NuGet มีแพคเกจสำหรับแทบทุกงานตั้งแต่การแจกแจง XML ไปจนถึงแมชชีนเลิร์นนิง แม้แต่การโพสต์ข้อความไปยังทวิตเตอร์ อันที่จริง ภายใต้ ASP.NET Core เองนั้นก็เป็นเพียงการรวมเอาแพคเกจ NuGet ต่าง ๆ ที่ถูกเพิ่มลงในโปรเจกต์ของคุณไว้ด้วยกันเท่านั้น

> ไฟล์โปรเจกต์ที่ถูกสร้างขึ้นด้วยคำสั่ง `dotnet new mvc` ได้รวมการอ้างอิงไปยังแพคเกจ `Microsoft.AspNetCore.All` ซึ่งเป็น "เมทาแพคเกจ" ที่มีการอ้างอิงไปยังแพคเกจต่าง ๆ ของ ASP.NET Core ที่ใช้บ่อยเอาไว้แล้ว ด้วยวิธีนี้คุณจึงไม่จำเป็นต้องอ้างอิงแพคเกจอีกนับร้อยไว้ในไฟล์โปรเจกต์ของคุณ

ในบทถัดไป คุณจะได้ใช้แพคเกจ NuGet อีกชุดหนึ่ง (เป็นระบบที่เรียกว่า Entity Framework Core) เพื่อเขียนโค้ดสำหรับสื่อสารกับฐานข้อมูล
