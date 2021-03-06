# พื้นฐานของ MVC
ในบทนี้ คุณจะได้ศึกษาเกี่ยวกับระบบ MVC ใน ASP.NET Core. **MVC** (Model-View-Controller) เป็นแบบแผนสำหรับการสร้างเว็บแอปพลิเคชันที่ใช้อยู่ในเว็บเฟรมเวิร์กเกือบทั้งหมด (ดังตัวอย่างที่ดีเช่น Ruby on Rails และ Express) รวมทั้งเฟรมเวิร์กจาวาสคริปต์ที่ทำงานเบื้องหน้าเช่น Angular นอกจากนี้ โมบายแอปบน iOS และแอนดรอยด์ต่างก็ใช้รูปแบบใดรูปแบบหนึ่งของ MVC เช่นกัน

MVC มีองค์ประกอบสามส่วนดังที่ปรากฏอยู่ในชื่อ: models, views และ controllers โดย **Controllers** ทำหน้าที่รองรับคำขอต่าง ๆ ที่มาจากไคลเอนต์หรือเว็บเบราว์เซอร์ แล้วตัดสินใจว่าจะให้โค้ดส่วนใดทำงาน  **Views** เป็นเทมเพลต (โดยทั่วไปจะเป็น HTML ที่ทำงานร่วมกับภาษาเทมเพลตเช่น Handlebars, Pug และ Razor) ที่นำข้อมูลต่าง ๆ เข้ามารวมไว้ด้วยกันแล้วจึงแสดงผลไปยังผู้ใช้ และ **Models** ที่คอยเก็บข้อมูลที่จะถูกเพิ่งเข้าไปใน view รวมถึงข้อมูลที่ถูกป้อนเข้ามาโดยผู้ใช้ด้วย

แบบแผนโดยทั่วไปของโค้ด MVC เป็นดังนี้:

* Controller รับคำขอเข้ามา แล้วค้นหาข้อมูลขึ้นมาจากฐานข้อมูล
* Controller สร้าง model ที่มีข้อมูลดังกล่าวแล้วจึงนำไปเชื่อมกับ view
* View ถูกสร้างขึ้น แล้วถูกแสดงผลบนเบราว์เซอร์ของผู้ใช้
* ผู้ใช้กดปุ่มหรือส่งข้อมูลบนฟอร์ม ซึ่งจะส่งคำขอใหม่ไปให้กับ controller แล้วจึงเริ่มวงจรนี้อีกครั้ง

หากคุณเคยทำงานกับ MVC ในภาษาอื่นมาแล้ว คุณจะรู้สึกคุ้นเคยกับ MVC ใน ASP.NET Core ได้ทันที หากคุณยังใหม่กับ MVC บทนี้จะช่วยให้คุณทราบถึงหลักการพื้นฐานและช่วยให้คุณเริ่มงานต่อไปได้

## เรากำลังจะสร้างอะไร
แบบฝึกหัดในแบบ "Hello World" ของ MVC นี้จะเป็นการสร้างแอปพลิเคชันรายการสิ่งที่ต้องทำ (to-do list) ซึ่งเป็นโปรเจกต์ที่ดีเนื่องจากมีขอบเขตที่เล็กและไม่ซับซ้อน แต่ขณะเดียวกับก็ครอบคลุมถึงหลักการและองค์ประกอบแต่ละด้านของ MVC ที่ประยุกต์ใช้ในแอปพลิเคชันใหญ่ ๆ ได้

ในหนังสือเล่มนี้ คุณจะได้สร้างแอปสิ่งที่ต้องทำที่ผู้ใช้สามารถเพิ่มกิจกรรมต่าง ๆ ลงในรายการสิ่งที่ต้องทำของตน พร้อมทั้งสามารถทำสัญลักษณ์รายการเมื่อกิจกรรมแล้วเสร็จได้ ซึ่งโดยรายละเอียดแล้ว สิ่งที่คุณจะสร้างมีดังนี้:

* เว็บแอปพลิเคชันเซอร์ฟเวอร์ (บางครั้งเรียกว่า "ส่วนหลัง") โดยใช้ ASP.NET Core ภาษา C# และแบบแผน MVC
* ฐานข้อมูลสำหรับใช้เก็บรายการกิจกรรมสิ่งที่ต้องทำของผู้ใช้ด้วยระบบฐานข้อมูล SQLite และระบบที่เรียกว่า Entity Framework Core
* หน้าเว็บต่าง ๆ และส่วนต่อประสานผู้ใช้ที่จะโต้ตอบกับผู้ใช้ผ่านทางเว็บเบราว์เซอร์ด้วย HTML, CSS และ JavaScript (เรียกว่าเป็น "ส่วนหน้า")
* แบบฟอร์มล็อกอินและตรวจสอบการรักษาความปลอดภัยเพื่อรให้ผู้ใช้แต่ละคนสามารถเก็ํบรายการสิ่งที่ต้องทำของตนไว้เป็นส่วนตัวได้

ดูดีใช่ไหม? มาเริ่มสร้างกันเลย! ถ้าคุณยังไม่ได้สร้างโปรเจกต์ ASP.NET Core ขึ้นมาใหม่ด้วยคำสั่ง `dotnet new mvc` ขอให้กลับไปทำตามขั้นตอนในบทที่แล้วเสียก่อน เมื่อเสร็จแล้วคุณควรสามารถสร้างและรันโปรเจกต์ให้แสดงหน้าต้อนรับตั้งต้นขึ้นมาได้
