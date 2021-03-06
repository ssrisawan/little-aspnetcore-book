## ใช้กล่องเลือกเพื่อระบุว่างานเสร็จแล้ว

ตอนนี้เราสามารถเพิ่มสิ่งที่ต้องทำเข้าไปยังรายการของเราได้แล้ว แต่ในที่สุดเราคงต้องการระบุให้ได้ว่างานใดทำเสร็จแล้วบ้าง ใน view  `Views/Todo/Index.cshtml` จะมีกล่องเลือกถูกสร้างขึ้นสำหรับแต่ละรายการสิ่งที่ต้องทำ:

```html
<input type="checkbox" class="done-checkbox">
```

การคลิกที่กล่องเลือกยังไม่ทำให้เกิดอะไรขึ้น (ในตอนนี้) และเช่นเดียวกับในบทที่แล้ว เราจะเพิ่มกิจกรรมนี้โดยใช้ฟอร์มและแอคชัน นอกจากนี้ เรายังจะได้เพิ่มโค้ดจาวาสคริปต์อีกเล็กน้อยด้วย


### เพิ่มองค์ประกอบของฟอร์มเข้าไปใน view

อันดับแรก ให้อัพเดต view โดยให้ครอบกล่องข้อความด้วยองค์ประกอบ `<form>` จากนั้น ให้เพิ่มองค์ประกอบที่มีค่า ID ของรายการดังกล่าวโดยซ่อนเอาไว้:

**Views/Todo/Index.cshtml**

```html
<td>
    <form asp-action="MarkDone" method="POST">
        <input type="checkbox" class="done-checkbox">
        <input type="hidden" name="id" value="@item.Id">
    </form>
</td>
```

เมื่อลูป `foreach` ทำงานใน view เพื่อแสดงรายการสิ่งที่ต้องทำ ก็จะเกิดสำเนาของฟอร์มนี้ขึ้นมาในแต่ละแถว โดยมีอินพุตที่เป็น ID ของรายการสิ่งที่ต้องทำที่ถูกซ่อนไว้รวมอยู่ด้วย ทำให้โค้ดของ controller สามารถบอกได้ว่ากล่องเลือกกล่องใดถูกคลิกเลือก (ไม่เช่นนั้น เราจะทำได้เพียงบอกว่ามี *บาง* กล่องถูกเลือกไว้เท่านั้น แต่ไม่สามารถบอกได้ว่ากล่องใดบเป็นกล่องที่ถูกเลือก)

หากเรารันแอปพลิเคชันในตอนนี้ กล่องเลือกเหล่านี้จะยังไม่ทำให้เกิดผลใด ๆ เนื่องจากยังไม่มีป่ม submit เพื่อให้เบราว์เซอร์สร้างคำขอ POST จากข้อมูลในฟอร์ม เราสามารถเพิ่มปุ่ม submit ลงไปใต้กล่องเลือกแต่ละกล่องก็ได้ แต่คงเป็นประสบการณ์ใช้งานที่ไม่ดีนักสำหรับผู้ใช้ ทางที่ดีแล้ว การคลิกที่กล่องเลือกควรเป็นการส่งฟอร์มโดยอัตโนมัติ ซึ่งเราสามารถทำได้โดยใช้จาวาสคริปต์


### เพิ่มโค้ดจาวาสคริปต์

มองหาไฟล์ `site.js` ในไดเรกทอรี `wwwroot/js` แล้วเพิ่มโค้ดดังนี้: 

**wwwroot/js/site.js**

```javascript
$(document).ready(function() {

    // Wire up all of the checkboxes to run markCompleted()
    $('.done-checkbox').on('click', function(e) {
        markCompleted(e.target);
    });
});

function markCompleted(checkbox) {
    checkbox.disabled = true;

    var row = checkbox.closest('tr');
    $(row).addClass('done');

    var form = checkbox.closest('form');
    form.submit();
}
```

อันดับแรก เราจะใช้ jQuery (ซึ่งเป็นไลบรารีตัวหนึ่งของจาวาสคริปต์) เพื่อเพิ่มโค้ดเข้าไปยังเหตุการณ์ `click` ของกล่องเลือกทั้งหมดบนหน้าจอที่ใช้คลาส CSS `done-checkbox` เมื่อกล่องเลือกถูกคลิกแล้ว ฟังก์ชัน `markCompleted()` ก็จะทำงาน

ฟังก์ชัน `markCompleted()` จะดำเนินการหลายอย่างดังนี้:
* เพิ่มคุณสมบัติ `disabled` ให้กล่องเลือกเพื่อไม่ให้สามารถคลิกซ้ำได้
* เพิ่มคลาส CSS `done` เข้าไปยังแถวที่วางกล่องเลือกนั้นไว้ เพื่อเปลี่ยนรูปแบบการแสดงผลของแถวนั้น ๆ ตามกฎ CSS ที่กำหนดไว้ใน `style.css`
* ส่งฟอร์ม

นั่นคือทั้งหมดที่ต้องทำสำหรับ view และโค้ดที่ทำงานอยู่เบื้องหน้า ตอนนี้ก็ถึงเวลาเพิ่มแอคชันใหม่กันแล้ว!


### เพิ่มแอคชันเข้าไปยัง controller

ดังที่คุณน่าจะคาดเดาเอาไว้แล้ว ในอันดับต่อไป เราต้องเพิ่มแอคชันชื่อ `MarkDone` เข้าไปใน `TodoController`:

```csharp
[ValidateAntiForgeryToken]
public async Task<IActionResult> MarkDone(Guid id)
{
    if (id == Guid.Empty)
    {
        return RedirectToAction("Index");
    }

    var successful = await _todoItemService.MarkDoneAsync(id);
    if (!successful)
    {
        return BadRequest("Could not mark item as done.");
    }

    return RedirectToAction("Index");
}
```

เรามาไล่ดูแต่ละบรรทัดในเมธอดแอคชันนี้กัน อันดับแรก เมธอดนี้จะรับพารามิเตอร์ `Guid` ชื่อ `id` ในลายเซ็นของเมธอด ซึ่งแตกต่างจากแอคชัน `AddItem` ที่ใช้โมเดลและทำการจับคู่/ตรวจสอบโมเดล โดยพารามิเตอร์ `id` นี้เป็นเพียงค่าง่าย ๆ ไม่ซับซ้อน หากคำขอที่เรียกเข้ามามีฟิลด์ชื่อ `id` แล้ว ASP.NET Core จะพยายามใช้ค่านั้นเป็น guid ซึ่งในที่นี้สามารถทำงานได้เนื่องจากองค์ประกอบที่เราซ่อนไว้ในฟอร์มกล่องเลือกของเรามีชื่อว่า `id`

และเนื่องจากเราไม่ได้ใช้การจับคู่โมเดล จึงไม่มี `ModelState` ให้ตรวจสอบว่าใช้ได้หรือไม่ ดังนั้น เราจึงต้องตรวจสอบค่า guid ดังกล่าวด้วยตนเองเพื่อให้มั่นใจว่าค่าดังกล่าวสามารถใช้งานได้ หากเกิดปัญหาใด ๆ ที่ทำให้ค่าพารามิเตอร์ `id` จากการร้องขอตกหล่นไปหรือไม่สามารถนำมาใช้เป็น guid ได้ จะทำให้ `id` มีค่าเป็น `Guid.Empty` หากเกิดกรณีเช่นนี้ แอคชันจะบอกเบราว์เซอร์ให้เปลี่ยนเส้นทางไปยัง `/Todo/Index` และเรียกหน้านี้ซ้ำอีกครั้ง

อันดับต่อมา controller จำเป็นต้องเรียกใช้ชั้นบริการให้อัพเดตฐานข้อมูล ขั้นตอนนี้จะดำเนินการโดยเมธอดใหม่ที่ชื่อ `MarkDoneAsync` ในอินเตอร์เฟส `ITodoItemService` ซึ่งจะคืนค่าเป็นจริงหรือเท็จขึ้นอยู่กับว่าการอัพเดตนั้นสำเร็จหรือไม่:

```csharp
var successful = await _todoItemService.MarkDoneAsync(id);
if (!successful)
{
    return BadRequest("Could not mark item as done.");
}
```

ท้ายที่สุดแล้ว หากทุกอย่างเรียกร้อยดี เบราว์เซอร์จะถูกส่งต่อไปที่แอคชัน `/Todo/Index` แล้วหน้าดังกล่าวจะถูกโหลดซ้ำอีกครั้ง

หลังจากที่เราได้แก้ไขทั้ง view และ controller เรียบร้อยแล้ว ก็เหลือแค่การเพิ่มเมธอดบริการที่ยังไม่ได้เขียนขึ้นมาเท่านั้น

### เพิ่มเมธอดบริการ

อันดับแรก ให้เพิ่ม `MarkDoneAsync` เข้าไปยังส่วนของการกำหนดอินเตอร์เฟส:

**Services/ITodoItemService.cs**

```csharp
Task<bool> MarkDoneAsync(Guid id);
```

จากนั้น ให้เพิ่มส่วนของการทำงานจริงเข้าไปยัง `TodoItemService`:

**Services/TodoItemService.cs**

```csharp
public async Task<bool> MarkDoneAsync(Guid id)
{
    var item = await _context.Items
        .Where(x => x.Id == id)
        .SingleOrDefaultAsync();

    if (item == null) return false;

    item.IsDone = true;

    var saveResult = await _context.SaveChangesAsync();
    return saveResult == 1; // One entity should have been updated
}
```

เมธอดนี้ใช้ Entity Framework Core และ `Where()` เพื่อค้นหารายการสิ่งที่ต้องทำจากฐานข้อมูลด้วย ID และเมธอด `SingleOrDefaultAsync()` จะคืนค่าเป็นรายการดังกล่าวหรือเป็น `null` หากไม่พบรายการนั้น ๆ

หลังจากที่เราตรวจสอบแล้วว่า `item` ไม่ได้เป็นค่า null ก็เหลือเพียงแค่กำหนดค่าให้กับคุณสมบัติ `IsDone` เท่านั้น:

```csharp
item.IsDone = true;
```

การเปลี่ยนแปลงคุณสมบัตินี้นี้ยังมีผลเฉพาะกับรายการที่อยู่ในเครื่องเท่านั้น จนกว่า `SaveChangesAsync()` จะถูกเรียกใช้เพื่อส่งค่าของการเปลี่ยนแปลงดังกล่าวกลับไปยังฐานข้อมูล โดย `SaveChangesAsync()` จะคืนค่าเป็นตัวเลขที่ระบุว่ามีกี่เอนทิตีที่ได้รับการอัพเดตระหว่างกระบวนการบันทึกข้อมูลดังกล่าว แต่ในที่นี้จะคืนค่าเพียง 1 (รายการได้รับการอัพเดต) หรือ 0 (เกิดความผิดพลาดบางอย่าง) เท่านั้น

### มาทดสอบกัน

ให้รันแอปพลิเคชันแล้วลองคลิกเลือกบางรายการจากบนหน้าจอดู จากนั้นให้โหลดหน้าซ้ำซึ่งจะพบว่ารายการที่ถูกเลือกไปเหล่านั้นหายไปจากบนหน้าจอ สาเหตุที่เป็นเช่นนี้เนื่องจากฟิลเตอร์ `Where()` ในเมธอด `GetIncompleteItemsAsync()` นั่นเอง

ในตอนนี้ แอปพลิเคชันของเรามีเพียงรายการสิ่งที่ต้องทำที่ใช้ร่วมกันสำหรับทุกคนที่เข้ามาใช้แอปพลิเคชัน คงจะเป็นประโยชน์กว่านี้หากสามารถเก็บสิ่งที่ต้องทำของผู้ใช้แต่ละคนแยกจากกันได้ ในบทต่อไป เราจะได้เพิ่มความสามารถของการล็อกอินและการรักษาความปลอดภัยให้โปรเจกต์ของเรา
