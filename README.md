# The Little ASP.NET Core Book

*by Nate Barbettini*

Copyright &copy; 2018. All rights reserved.

ISBN: 978-1-387-75615-5

Released under the Creative Commons Attribution 4.0 license. You are free to share, copy, and redistribute this book in any format, or remix and transform it for any purpose (even commercially). You must give appropriate credit and provide a link to the license.

สำหรับข้อมูลเพิ่มเติม โปรดไปที่ https://creativecommons.org/licenses/by/4.0/

## เกริ่นนำ
ขอบคุณที่เลือกหนังสือ The Little ASP.NET Core ขึ้นมาอ่าน! ผมเขียนหนังสือสั้น ๆ เล่มนี้เพื่อช่วยให้นักพัฒนาและผู้ที่สนใจในการเขียนโปรแกรมบนเว็บให้ได้เรียนรู้เกี่ยวกับ ASP.NET Core 2.0 ซึ่งเป็นเฟรมเวิร์กสำหรับสร้างเว็บแอปพลิเคชันและ API

หนังสือ The Little ASP.NET Core ถูกวางโครงสร้างให้ทดลองทำตามเป็นขั้นตอน คุณจะได้สร้างแอปพลิเคชันตั้งแต่เริ่มจนจบและได้เรียนรู้เกี่ยวกับ:

* พื้นฐานเกี่ยวกับรูปแบบ MVC (Model-View-Controller)
* โค้ดส่วนหน้า (HTML, CSS, JavaScript) ทำงานร่วมกับโค้ดส่วนหลังอย่างไร
* Dependency injection คืออะไรและมีประโยชน์อย่างไร
* การอ่านและเขียนข้อมูลจากฐานข้อมูล
* การเพิ่มส่วนล็อกอิน การลงทะเบียน และการรักษาความปลอดภัย
* การนำส่ง (deploy) แอปพลิเคชันขึ้นไปยังเว็บ

ไม่ต้องกังวลหากคุณยังไม่รู้อะไรเกี่ยวกับ ASP.NET Core (หรือหัวข้อใด ๆ ข้างต้น) เพื่อเริ่มต้นใช้งาน

## Before you begin

โค้ดที่เสร็จสมบูรณ์ของแอปพลิเคชันที่คุณกำลังจะสร้างขึ้นสามารถเข้าถึงได้บน GitHub:

https://www.github.com/nbarbettini/little-aspnetcore-todo

คุณสามารถดาวน์โหลดมาทดลองก่อนได้หากต้องการดูงานที่เสร็จสมบูรณ์ หรืออาจนำมาใช้เพื่อเปรียบเทียบกับโค้ดที่คุณเขียนก็ได้

หนังสือเล่มนี้ได้รับการปรับปรุงเพื่อแก้บั๊กและเพิ่มเนื้อหาอย่างสม่ำเสมอ หากคุณกำลังอ่านฉบับที่เป็น PDF หนังสืออิเล็กทรอนิกส์ หรือฉบับที่พิมพ์เป็นเล่ม สามารถตรวจสอบที่เว็บไซต์อย่างเป็นทางการ ([littleasp.net/book](http://www.littleasp.net/book)) เพื่อดูว่ามีรุ่นที่ใหม่กว่าหรือไม่ โดยข้อมูลรุ่นและบันทึกการเปลี่ยนแปลงอยู่ที่หน้าสุดท้ายของหนังสือเล่มนี้

### อ่านหนังสือเล่นนี้ในภาษาของคุณ

Thanks to some fantastic multilingual contributors, the Little ASP.NET Core Book has been translated into other languages:

* [**ASP.NET Core El Kitabı**](https://sahinyanlik.gitbooks.io/kisa-asp-net-core-kitabi/) (ภาษาตุรกี)
 	 
* [**简明 ASP.NET Core 手册**](https://windsting.github.io/little-aspnetcore-book/book/) (ภาษาจีน)


## หนังสือเล่มนี้เหมาะกับใคร
ถ้าคุณยังใหม่กับการเขียนโปรแกรม หนังสือเล่มนี้จะแนะนำคุณเกี่ยวกับแบบแผนและหลักการต่าง ๆ ที่ถูกใช้เพื่อสร้างเว็บแอปพลิเคชันสมัยใหม่ คุณจะได้เรียนรู้เกี่ยวกับการสร้างเว็บแอป (และรู้ว่าชิ้นส่วนต่าง ๆ ประกอบเข้าด้วยกันอย่างไร) ด้วยการสร้างบางอย่างขึ้นมาตั้งแต่ต้น และแม้ว่าหนังสือเล่มเล็ก ๆ นี้จะไม่สามารถครอบคลุมทุกอย่างที่คุณต้องรู้เกี่ยวกับการเขียนโปรแกรมได้ แต่คุณสามารถใช้เป็นจุดเริ่มต้นเพื่อการเรียนรู้ในขั้นสูงต่อไป

หากคุณเขียนโค้ดอยู่แล้วด้วยภาษาเบื้องหลัง เช่น Node Python Ruby Go หรือ Java คุณจะสัมผัสได้ถึงไอเดียต่าง ๆ ที่คุ้นเคย เช่น MVC, view templates และ dependency injection แม้ว่าโค้ดในหนังสือเล่มนี้ใช้ภาษา C# ก็จะดูไม่แตกต่างจากสิ่งที่คุณรู้อยู่แล้วมากนัก

หากคุณเป็นนักพัฒนา ASP.NET MVC คุณจะรู้สึกเหมือนอยู่บ้าน! ASP.NET Core เพิ่มเครื่องมือใหม่ ๆ และใช้สิ่งทีต่าง ๆ ่คุณรู้จักอยู่แล้วซ้ำอีกครั้ง (และทำให้เข้าใจง่ายขึ้น) โดยความแตกต่างบางส่วนจะถูกกล่าวถึงไว้ด้านล่างนี้

ไม่ว่าคุณจะมีประสบการณ์กับการเขียนโปรแกรมบนเว็บมาก่อนหรือไม่ หนังสือเล่มนี้จะสอนคุณทุกอย่างที่จำเป็นต่อการสร้างเว็บแอปพลิเคชันอย่างง่าย ๆ และมีประโยชน์ด้วย ASP.NET Core คุณจะได้เรียนรู้ถึงการสร้างกระบวนการทำงานต่าง ๆ โดยใช้โค้ดทั้งส่วนหลัง (backend) และส่วนหน้า (frontend) การทำงานกับฐานข้อมูล และการนำส่งแอปขึ้นไปยังเซอร์ฟเวอร์

## ASP.NET Core คืออะไร
ASP.NET Core เป็นเว็บเฟรมเวิร์กที่ถูกสร้างขึ้นโดยไมโครซอฟต์เพื่อสร้างเว็บแอปพลิเคชัน API และไมโครเซอร์วิส โดยใช้แพทเทิร์นที่ใช้กันอย่างแพร่หลายเช่น MVC (Model-View-Controller), dependency injection และไปป์ไลน์ของคำขอที่ประกอบขึ้นมาจากมิดเดิลแวร์ และเนื่องจาก ASP.NET Core เป็นซอฟต์แวร์โอเพนซอร์สที่ถูกเผยแพร่ภายใต้ไลเซนส์ Apache 2.0 ทำให้สามารถใช้ซอร์สโค้ดได้อย่างอิสระ และสังคมผู้ใช้งานยังสามารถช่วยกันแก้ไขบั๊ก รวมทั้งพัฒนาความสามารถใหม่ ๆ ได้อีกด้วย

ASP.NET Core ทำงานอยู่บน .NET runtime ของไมโครซอฟต์ในลักษณะเดียวกันกับ Java Virtual Machine (JVM) และตัวแปลภาษา Ruby คุณสามารถเขียนแอปพลิเคชัน ASP.NET Core ได้ในหลายภาษา (C#, Visual Basic, F#) โดย C# เป็นตัวเลือกที่ได้รับความนิยมมากที่สุด และเป็นภาษาที่ถูกใช้ในหนังสือเล่มนี้ คุณสามารถสร้างและรันแอปพลิเคชัน ASP.NET Core ได้บนวินโดวส์ แมค และลินุกซ์

## ทำไมต้องใช้เฟรมเวิร์กใหม่?
ปัจจุบันเรามีเว็บเฟรมเวิร์กให้เลือกใช้อย่างหลากหลาย: Node/Express, Spring, Ruby on Rails, Django, Laravel และอื่น ๆ อีกมากมาย แล้ว ASP.NET Core มีจุดเด่นอย่างไร

* **ความเร็ว** ASP.NET Core ทำงานได้รวดเร็ว เพราะโค้ด .NET ถูกคอมไพล์ไว้แล้ว จึงทำงานได้เร็วกว่าโค้ดของภาษาที่ต้องใช้ตัวแปลภาษาเช่น JavaScript และ Ruby มาก ASP.NET Core ยังถูกปรับแต่งให้ทำงานได้ดีกับงานแบบหลายเธรดและแบบอะซิงโครนัส จึงเป็นเรื่องปกติหากจะพบว่าโปรแกรมสามารถทำงานได้เร็วกว่าโค้ดที่เขียนด้วย Node.js ราว 5-10 เท่า

* **ระบบสภาพแวดล้อม** ASP.NET Core อาจยังใหม่ แต่ .NET นั้นอยู่มานานแล้ว ดังนั้นจึงมีแพคเกจจำนวนมากอยู่บน (ตัวจัดการแพคเกจของ .NET package manager; คล้ายกับ npm, Ruby gems, และ Maven) เช่นแพคเกจสำหรับแยกข้อความ JSON, การเชื่อมต่อฐานข้อมูล, การสร้าง PDF, ไปจนถึงอะไรก็ตามที่คุณอยากได้

* **ความปลอดภัย** ทีมงานที่ไมโครซอฟต์ให้ความสำคัญกับความปลอดภัยเป็นอย่างมาก ASP.NET Core ก็ถูกสร้างขึ้นโดยคำนึงถึงความปลอดภัยตั้งแต่แรก โดยช่วยรองรับงานต่าง ๆ เช่นการทำให้ข้อมูลนำเข้าปลอดภัยและป้องกันการโจมตีแบบปลอมแปลงแบบโจมตีคำขอข้ามไซต์ (cross-site request forgery: CSRF) คุณจึงไม่ต้องดำเนินการด้วยตนเอง นอกจากนี้ยังมีข้อดีของการกำหนดชนิดข้อมูลแบบ static ด้วยคอมไพเลอร์ .NET ซึ่งเปรียบได้กับการเปิดใช้การตรวจสอบข้อผิดพลาดขั้นสูงสุดไว้ตลอดเวลา จึงช่วยลดโอกาสเกิดความผิดพลาดกับชนิดของตัวแปรหรือข้อมูลโดยไม่ได้ตั้งใจได้

## .NET Core และมาตรฐาน .NET (.NET Standard)
ตลอดหนังสือเล่มนี้ คุณจะได้เรียนรู้เกี่ยวกับ ASP.NET Core (ซึ่งเป็นเว็บเฟรมเวิร์ก) และอาจมีการกล่าวถึงรันไทม์ของ .NET ซึ่งเป็นไลบรารีสนับสนุนสำหรับรันโค้ด .NET เป็นระยะ หากถึงตอนนี้เริ่มดูเหมือนเป็นภาษาต่างดาวสำหรับคุณแล้วก็ไม่เป็นไร ข้ามไปบทต่อไปเลยก็ได้!

คุณอาจเคยได้ยินเกี่ยวกับ .NET Core และมาตรฐาน .NET การตั้งชื่อดูจะทำให้เกิดความสับสน แต่สามารถอธิบายให้เข้าใจง่าย ๆ ได้ดังนี้:

**มาตรฐาน .NET** เป็นอินเตอร์เฟสที่ไม่ขึ้นกับแพลตฟอร์มที่กำหนดความสามารถและ API ไว้ ต้องทำความเข้าใจว่ามาตรฐาน .NET ไม่ได้เป็นตัวโค้ดหรือการทำงานจริงใด ๆ แต่เป็นเพียงข้อกำหนด API เท่านั้น ปัจจุบันมีมาตรฐาน .NET หลาย "เวอร์ชัน" หรือระดับที่แสดงถึงจำนวนของ API ที่มีให้ใช้งานได้ (หรือความครอบคลุมด้านต่าง ๆ ของ API) ตัวอย่างเช่น มาตรฐาน .NET 2.0 มี API ให้ใช้งานได้มากกว่ามาตรฐาน .NET 1.5 ซึ่งก็มี APIs มากกว่ามาตรฐาน .NET 1.0

**.NET Core** เป็นรันไทม์ .NET ที่สามารถติดตั้งได้ทั้งบนวินโดวส์ แมค และลินุกซ์ โดยได้นำ API ที่ถูกกำหนดไว้ในอินเตอร์เฟสมาตรฐาน .NET ที่มาพร้อมกับโค้ดที่เหมาะสมกับแต่ละระบบปฏิบัติการ นี่คือสิ่งที่เราต้องติดตั้งลงในเครื่องเพื่อสร้างและรันแอปพลิเคชัน ASP.NET Core

และเพื่อเป็นการเปรียบเทียบ **.NET Framework** เป็นการนำมาตรฐาน .NET มาใช้อีกรูปแบบหนึ่งสำหรับทำงานบนวินโดวส์โดยเฉพาะ ซึ่งเคยเป็นรันไทม์ .NET เพียงหนึ่งเดียวก่อนการมาถึงของ .NET Core ที่นำ .NET ไปสู่แมคและลินุกซ์ ASP.NET Core ก็สามารถทำงานบน .NET Framework ที่ทำงานได้เฉพาะบนวินโดวส์ก็ได้ แต่จะไม่นำมากล่าวถึงในที่นี้มากนัก

หากคุณยังสับสนกับชื่อต่าง ๆ เหล่านี้ ไม่ต้องกังวลไป! เราจะเริ่มเขียนโค้ดจริงกันเร็ว ๆ นี้

## หมายเหตุถึงนักพัฒนา ASP.NET 4
หากคุณเคยใช้ ASP.NET รุ่นก่อนหน้านี้มาแล้ว ก็สามารถข้ามไปบทต่อไปได้เลย

ASP.NET Core เป็นการเขียน ASP.NET ขึ้นมาใหม่ตั้งแต่ต้น โดยมุ่งเน้นไปที่การทำให้เฟรมเวิร์กทันสมัย และเพื่อให้แยกตัวออกจาก System.Web, IIS และวินโดวส์ได้ในที่สุด หากคุณจำ OWIN/Katana จาก ASP.NET 4 ได้ แสดงว่าคุณมาได้ครึ่งทางแล้ว: โครงการ Katana ได้กลายมาเป็น ASP.NET 5 และในที่สุดก็ถูกเปลี่ยนชื่อมาเป็น ASP.NET Core นั่นเอง

สืบเนื่องมาจาก Katana ทำให้คลาส `Startup`กลายเป็นกลไกสำคัญในการทำงาน และไม่มีทั้ง `Application_Start` และ `Global.asax` อีกต่อไป ในตอนนี้ ทั้งไปป์ไลน์จะถูกขับเคลื่อนด้วยมิดเมิลแวร์ และไม่มีการแยกระหว่าง MVC และ Web API อีกแล้ว: controller สามารถคืนค่าเป็น view, รหัสสถานะ หรือข้อมูลก็ได้ และ Dependency injection ถูกรวมเอาไว้แล้ว จึงไม่มีความจำเป็นต้องติดตั้งและกำหนดค่าให้คอนเทนเนอร์เช่น StructureMap และ Ninject หากไม่ต้องการ นอกจากนี้ เฟรมเวิร์กทั้งหมดได้ถูกปรับแต่งให้ทำงานได้อย่างรวดเร็วและมีประสิทธิภาพอีกด้วย

เอาล่ะ เกริ่นนำกันมาพอแล้ว มาเริ่มลงลึกกับ ASP.NET Core กันเถอะ!
