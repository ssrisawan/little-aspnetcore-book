# อัตลักษณ์และการรักษาความปลอดภัย

ความปลอดภัยเป็นหนึ่งในประเด็นหลักที่ต้องคำนึงถึงในเว็บแอปพลิเคชันและ API สมัยใหม่ การเก็บรักษาข้อมูลของผู้ใช้และลูกค้าให้พ้นจากเงื้อมมือของผู้ไม่หวังดีเป็นสิ่งสำคัญยิ่ง หัวข้อนี้จึงครอบคลุมหลายด้านและเกี่ยวข้องกับสิ่งต่าง ๆ เช่น:

* การทำให้ข้อมูลนำเข้าปลอดภัยเพื่อป้องกันการโจมตีแบบ SQL injection
* การป้องกันการโจมตีแบบข้ามโดเมน (CSRF) ในฟอร์มต่าง ๆ
* การใช้ HTTPS (การเชื่อมต่อแบบเข้ารหัสลับ) เพื่อให้ไม่สามารถดักดูข้อมูลได้ระหว่างถูกส่งผ่านอินเทอร์เน็ต
* การทำให้ผู้ใช้สามารถเข้าระบบได้โดยใช้รหัสผ่านหรือวิธีการอื่น ๆ
* การออกแบบระบบรีเซ็ตรหัสผ่าน การกู้คืนบัญชีผู้ใช้ และกระบวนการพิสูจน์ตัวจริงแบบหลายปัจจัย (multi-factor authentication)

ASP.NET Core ช่วยให้การสร้างสิ่งเหล่านี้ขึ้นมาทำได้ง่ายยิ่งขึ้น สองรายการข้างต้น (การป้องกันการโจมตีแบบ SQL injection และแบบข้ามโดเมน) ถูกสร้างไว้ในระบบแล้ว และเราสามารถเพิ่มโค้ดเพียงไม่กี่บรรทัดเพื่อเพิ่มการรองรับ HTTPS ในบทนี้จึงเน้นไปที่ประเด็นด้านการตรวจสอบอัตลักษณ์หรือ **identity**: การจัดการบัญชีผู้ใช้ การพิสูจน์ตัวจริง (การล็อกอิน) ของผู้ใช้อย่างปลอดภัย และการตัดสินใจเกี่ยวกับการกำหนดสิทธิให้หลังจากผู้ใช้พิสูจน์ตัวจริงแล้ว

> การพิสูจน์ตัวจริงและการกำหนดสิทธิเป็นสองแนวคิดที่แตกต่างกัน แต่ด้วยชื่อที่ใกล้เคียงกันจึงมักทำให้เกิดความสับสนได้โดยง่าย **การพิสูจน์ตัวจริง (authentication)** จัดการเกี่ยวกับการล็อกอินเข้าระบบของผู้ใช้ ขณะที่ **การกำหนดสิทธิ (authorization)** จัดการเกี่ยวกับสิทธิของผู้ใช้รายนั้น ๆ ว่าสามารถทำอะไรได้บ้าง *หลังจาก* ได้ล็อกอินเข้าระบบแล้ว เราอาจเปรียบได้ว่าการพิสูจน์ตัวจริงก็เหมือนกับการถามว่า "เรารู้จักผู้ใช้คนนี้หรือไม่?" ขณะที่การกำหนดสิทธิเป็นการถามว่า "ผู้ใช้คนนี้ได้รับอนุญาติให้ทำ *X* หรือไม่?"

เทมเพลต MVC + Individual Authentication ที่เราใช้ขึ้นโครงโปรเจกต์ไว้ก่อนหน้านี้ได้รวมเอาคลาสต่าง ๆ ที่ถูกสร้างบนรากฐานระบบอัตลักษณ์ของ ASP.NET Core หรือ ASP.NET Core Identity เอาไว้แล้ว โดยระบบนี้เป็นส่วนหนึ่งของ ASP.NET Core และดำเนินการเกี่ยวกับอัตลักษณ์และการพิสูจน์ตัวจริงของผู้ใช้ โดยค่าตั้งต้นแล้ว เทมเพลตนี้ช่วยให้ล็อกอินได้โดยใช้อีเมลและรหัสผ่าน 

## ASP.NET Core Identity คืออะไร?

ASP.NET Core Identity เป็นระบบอัตลักษณ์ที่มาพร้อมกับ ASP.NET Core โดยเป็นแพคเกจ NuGet ที่สามารถติดตั้งลงในโปรเจกต์ใด ๆ ก้ได้ เช่นเดียวกับทุกอย่างในระบบนิเวศของ ASP.NET (และได้รวมอยู่ในโปรเจกต์อยู่แล้วหากใช้เทมเพลตตั้งต้น)

ASP.NET Core Identity ช่วยจัดการเกี่ยวกับการจัดเก็บบัญชีผู้ใช้ การทำแฮชและจัดเก็บรหัสผ่าน รวมทั้งการจัดการบทบาทต่าง ๆ สำหรับผู้ใช้ ระบบนี้รองรับการล็อกอินด้วยอีเมลและรหัสผ่าน การพิสูจน์ตัวจริงแบบหลายปัจจัย การล็อกอินโดยผู้ให้บริการเช่นกูเกิลและเฟสบุ๊ก รวมไปถึงการเชื่อมต่อกับบริการอื่น ๆ โดยใช้โปรโตคอลเช่น OAuth 2.0 และ OpenID Connect

view สำหรับการลงทะเบียนและล็อกอินที่มาพร้อมกับเทมเพลต MVC + Individual Authentication ใช้ศักยภาพของ ASP.NET Core Identity และพร้อมใช้งานอยู่แล้ว! ลองลงทะเบียนเพื่อสร้างบัญชีผู้ใช้ใหม่แล้วล็อกอินเข้าระบบดู
