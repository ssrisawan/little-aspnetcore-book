## ปรับแต่งเลย์เอาต์

ไฟล์เลย์เอาต์ที่ `Views/Shared/_Layout.cshtml` มีโค้ด HTML "พื้นฐาน" สำหรับแต่ละ view ซึ่งรวมทั้งแถบนำทางหรือ navbar ที่ถูกวางตำแหน่งไว้ที่ด้านบนของแต่ละหน้าด้วย

ในการเพิ่มรายการใหม่ลงในแถบนำทาง ให้หาโค้ด HTML สำหรับรายการของ navbar ที่มีอยู่แล้ว:

**Views/Shared/_Layout.cshtml**

```html
<ul class="nav navbar-nav">
    <li><a asp-area="" asp-controller="Home" asp-action="Index">
        Home
    </a></li>
    <li><a asp-area="" asp-controller="Home" asp-action="About">
        About
    </a></li>
    <li><a asp-area="" asp-controller="Home" asp-action="Contact">
        Contact
    </a></li>
</ul>
```

เพิ่มรายการของคุณขึ้นมาใหม่โดยให้ชี้ไปที่ controller `Todo` แทนที่จะเป็น `Home`:

```html
<li>
    <a asp-controller="Todo" asp-action="Index">My to-dos</a>
</li>
```

แอตทริบิวต์ `asp-controller` และ `asp-action` ในส่วนย่อย `<a>` เรียกว่า **ตัวช่วยแท็ก (tag helper)** ก่อนที่ view จะถูกสร้างขึ้นมา ASP.NET Core จะแทนที่ตัวช่วยแท็กเหล่านี้ให้ด้วยแอตทริบิวต์ของ HTML  ซึ่งในที่นี้ ค่า URL ที่จะไปยังเส้นทาง `/Todo/Index` จะถูกสร้างขึ้นมาและเพิ่มไปยังส่วนย่อย `<a>` เป็นแอตทริบิวต์ `href` นั่นหมายความว่าคุณไม่ต้องกำหนดเส้นทางไปยัง `TodoController` ลงไปโดยตรงในโค้ด แต่ ASP.NET Core จะช่วยสร้างขึ้นมาให้คุณโดยอัตโนมัติ

> หากคุณเคยใช้ Razor ใน ASP.NET 4.x มาก่อนหน้านี้ อาจสังเกตว่ามีไวยากรณ์บางส่วนเปลี่ยนไป แทนที่จะใช้ `@Html.ActionLink()` เพื่อสร้างลิงก์ไปยังแอคชัน ในตอนนี้ ตัวช่วยแท็กเป็นวิธีที่แนะนำสำหรับการสร้างลิงก์ต่าง ๆ ใน view ของคุณ นอกจากนี้ ตัวช่วยแท็กยังมีประโยชน์สำหรับการทำงานกับฟอร์มอีกด้วย (ดังที่จะได้เห็นในบทต่อ ๆ ไป) คุณสามารถเรียนรู้เพิ่มเติมเกี่ยวกับตัวช่วยแท็กได้จากเอกสารที่ https://docs.asp.net
