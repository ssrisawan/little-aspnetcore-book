# The Little ASP.NET Core Book

*by Nate Barbettini*

Copyright &copy; 2018. All rights reserved.

ISBN: 978-1-387-75615-5

Released under the Creative Commons Attribution 4.0 license. You are free to share, copy, and redistribute this book in any format, or remix and transform it for any purpose (even commercially). You must give appropriate credit and provide a link to the license.

สำหรับข้อมูลเพิ่มเติม โปรดไปที่ https://creativecommons.org/licenses/by/4.0/

## เกริ่นนำ
ขอบคุณที่เลือกหนังสือ The Little ASP.NET Core ขึ้นมาอ่าน! ผมเขียนหนังสือสั้น ๆ เล่นนี้เพื่อช่วยให้นักพัฒนาและผู้ที่สนใจในการเขียนโปรแกรมบนเว็บให้ได้เรียนรู้เกี่ยวกับ ASP.NET Core 2.0 ซึ่งเป็นเฟรมเวิร์กสำหรับสร้างเว็บแอปพลิเคชันและ API

หนังสือ The Little ASP.NET Core ถูกวางโครงสร้างให้ทดลองทำตามเป็นขั้นตอน คุณจะได้สร้างแอปพลิเคชันตั้งแต่เริ่มจนจบและได้เรียนรู้เกี่ยวกับ:

* พื้นฐานเกี่ยวกับรูปแบบ MVC (Model-View-Controller)
* โค้ดส่วนหน้า (HTML, CSS, JavaScript) ทำงานร่วมกับโค้ดส่วนหลังอย่างไร
* Dependency injection คืออะไรและมีประโยชน์อย่างไร
* การอ่านและเขียนข้อมูลจากฐานข้อมูล
* การเพิ่มส่วนล็อกอิน การลงทะเบียน และการรักษาความปลอดภัย
* การส่ง (deploy) แอปพลิเคชันขึ้นไปยังเว็บ

ไม่ต้องกังวลหากคุณยังไม่รู้อะไรเกี่ยวกับ ASP.NET Core (หรือหัวข้อใด ๆ ข้างต้น) เพื่อเริ่มต้นใช้งาน

## Before you begin

โค้ดที่เสร็จสมบูรณ์ของแอปพลิเคชันที่คุณกำลังจะสร้างขึ้นสามารถเข้าถึงได้บน GitHub:

https://www.github.com/nbarbettini/little-aspnetcore-todo

คุณสามารถดาวน์โหลดมาทดลองก่อนได้หากต้องการดูงานที่เสร็จสมบูรณ์ หรืออาจนำมาใช้เพื่อเปรียบเทียบกับโค้ดที่คุณเขียนก็ได้

หนังสือเล่มนี้ได้รับการปรับปรุงเพื่อแก้บั๊กและเพิ่มเนื้อหาอย่างสม่ำเสมอ หากคุณกำลังอ่านฉบับที่เป็น PDF หนังสืออิเล็กทรอนิกส์ หรือฉบับที่พิมพ์เป็นเล่ม สามารถตรวจสอบที่เว็บไซต์อย่างเป็นทางการ ([littleasp.net/book](http://www.littleasp.net/book)) เพื่อดูว่ามีรุ่นที่ใหม่กว่าหรือไม่ โดยข้อมูลรุ่นและบันทึกการเปลี่ยนแปลงอยู่ที่หน้าสุดท้ายของหนังสือเล่มนี้

### อ่านหนังสือเล่นนี้ในภาษาของคุณ

Thanks to some fantastic multilingual contributors, the Little ASP.NET Core Book has been translated into other languages:

* [**ASP.NET Core El Kitabı**](https://sahinyanlik.gitbooks.io/kisa-asp-net-core-kitabi/) (Turkish)
 	 
* [**简明 ASP.NET Core 手册**](https://windsting.github.io/little-aspnetcore-book/book/) (ภาษาจีน)


## หนังสือเล่มนี้เหมาะกับใคร
ถ้าคุณยังใหม่กับการเขียนโปรแกรม หนังสือเล่มนี้จะแนะนำคุณเกี่ยวกับแบบแผนและหลักการต่าง ๆ ที่ถูกใช้เพื่อสร้างเว็บแอปพลิเคชันสมัยใหม่ คุณจะได้เรียนรู้เกี่ยวกับการสร้างเว็บแอป (และรู้ว่าชิ้นส่วนต่าง ๆ ประกอบเข้าด้วยกันอย่างไร) ด้วยการสร้างบางอย่างขึ้นมาตั้งแต่ต้น และแม้ว่าหนังสือเล่มเล็ก ๆ นี้จะไม่สามารถครอบคลุมทุกอย่างที่คุณต้องรู้เกี่ยวกับการเขียนโปรแกรมได้ แต่คุณสามารถใช้เป็นจุดเริ่มต้นเพื่อการเรียนรู้ในขั้นสูงต่อไป

หากคุณเขียนโค้ดอยู่แล้วด้วยภาษาเบื้องหลัง เช่น Node Python Ruby Go หรือ Java คุณจะสัมผัสได้ถึงไอเดียต่าง ๆ ที่คุ้นเคย เช่น MVC, view templates และ dependency injection แม้ว่าโค้ดในหนังสือเล่มนี้ใช้ภาษา C# ก็จะดูไม่แตกต่างจากสิ่งที่คุณรู้อยู่แล้วมากนัก

หากคุณเป็นนักพัฒนา ASP.NET MVC คุณจะรู้สึกเหมือนอยู่บ้าน! ASP.NET Core เพิ่มเครื่องมือใหม่ ๆ และใช้สิ่งทีต่าง ๆ ่คุณรู้จักอยู่แล้วซ้ำอีกครั้ง (และทำให้เข้าใจง่ายขึ้น) ส่วนที่แตกต่างกันบางส่วนถูกแสดงไว้ดัง I'll point out some of the differences below.

ไม่ว่าคุณจะมีประสบการณ์กับการเขียนโปรแกรมบนเว็บมาก่อนหรือไม่ หนังสือเล่มนี้จะสอนคุณทุกอย่างที่จำเป็นต่อการสร้างเว็บแอปพลิเคชันอย่างง่าย ๆ และมีประโยชน์ด้วย ASP.NET Core คุณจะได้เรียนรู้ถึงการสร้างกระบวนการทำงานต่าง ๆ โดยใช้โค้ดทั้งส่วนหลัง (backend) และส่วนหน้า (frontend) การทำงานกับฐานข้อมูล and how to deploy the app to the world.

## ASP.NET Core คืออะไร
ASP.NET Core เป็นเว็บเฟรมเวิร์กที่ถูกสร้างขึ้นโดยไมโครซอฟต์เพื่อสร้างเว็บแอปพลิเคชัน API และไมโครเซอร์วิส โดยใช้แพทเทิร์นที่ใช้กันอย่างแพร่หลายเช่น MVC (Model-View-Controller), dependency injection, and a request pipeline comprised of middleware. It's open-source under the Apache 2.0 license, which means the source code is freely available, and the community is encouraged to contribute bug fixes and new features.

ASP.NET Core ทำงานอยู่บน .NET runtime ของไมโครซอฟต์ในลักษณะเดียวกันกับ Java Virtual Machine (JVM) และตัวแปลภาษา Ruby คุณสามารถเขียนแอปพลิเคชัน ASP.NET Core ได้ในหลายภาษา (C#, Visual Basic, F#) โดย C# เป็นตัวเลือกที่ได้รับความนิยมมากที่สุด และเป็นภาษาที่ถูกใช้ในหนังสือเล่มนี้ คุณสามารถสร้างและรันแอปพลิเคชัน ASP.NET Core ได้บนวินโดวส์ แมค และลินุกซ์

## Why do we need another web framework?
ปัจจุบันเรามีเว็บเฟรมเวิร์กให้เลือกใช้อย่างหลากหลายอยู่แล้ว: Node/Express, Spring, Ruby on Rails, Django, Laravel และอื่น ๆ อีกมากมาย แล้ว ASP.NET Core มีจุดเด่นอย่างไร

* **ความเร็ว** ASP.NET Core ทำงานได้รวดเร็ว เพราะโค้ด .NET ถูกคอมไพล์ไว้แล้ว จึงทำงานได้เร็วกว่าโค้ดของภาษาที่ต้องใช้ตัวแปลภาษาเช่น JavaScript และ Ruby มาก ASP.NET Core is also optimized for multithreading and asynchronous tasks. It's common to see a 5-10x speed improvement over code written in Node.js.

* **Ecosystem.** ASP.NET Core may be new, but .NET has been around for a long time. There are thousands of packages available on NuGet (the .NET package manager; think npm, Ruby gems, or Maven). There are already packages available for JSON deserialization, database connectors, PDF generation, or almost anything else you can think of.

* **ความปลอดภัย** ีมงานที่ไมโครซอฟต์ให้ความสำคัญกับความปลอดภัยเป็นอย่างมาก and ASP.NET Core is built to be secure from the ground up. It handles things like sanitizing input data and preventing cross-site request forgery (CSRF) attacks, so you don't have to. You also get the benefit of static typing with the .NET compiler, which is like having a very paranoid linter turned on at all times. This makes it harder to do something you didn't intend with a variable or chunk of data.

## .NET Core and .NET Standard
Throughout this book, you'll be learning about ASP.NET Core (the web framework). I'll occasionally mention the .NET runtime, the supporting library that runs .NET code. If this already sounds like Greek to you, just skip to the next chapter!

You may also hear about .NET Core and .NET Standard. The naming gets confusing, so here's a simple explanation:

**.NET Standard** is a platform-agnostic interface that defines features and APIs. It's important to note that .NET Standard doesn't represent any actual code or functionality, just the API definition. There are different "versions" or levels of .NET Standard that reflect how many APIs are available (or how wide the API surface area is). For example, .NET Standard 2.0 has more APIs available than .NET Standard 1.5, which has more APIs than .NET Standard 1.0.

**.NET Core** is the .NET runtime that can be installed on Windows, Mac, or Linux. It implements the APIs defined in the .NET Standard interface with the appropriate platform-specific code on each operating system. This is what you'll install on your own machine to build and run ASP.NET Core applications.

And just for good measure, **.NET Framework** is a different implementation of .NET Standard that is Windows-only. This was the only .NET runtime until .NET Core came along and brought .NET to Mac and Linux. ASP.NET Core can also run on Windows-only .NET Framework, but I won't touch on this too much.

หากคุณยังสับสนกับชื่อต่าง ๆ เหล่านี้ ไม่ต้องกังวลไป! เราจะเริ่มเขียนโค้ดจริงกันเร็ว ๆ นี้

## A note to ASP.NET 4 developers
If you haven't used a previous version of ASP.NET, skip ahead to the next chapter.

ASP.NET Core เป็นการเขียน ASP.NET ขึ้นมาใหม่ตั้งแต่ต้น โดยมุ่งเน้นไปที่การทำให้เฟรมเวิร์กทันสมัย และเพื่อให้แยกตัวออกจาก System.Web, IIS และวินโดวส์ได้ในที่สุด หากคุณจำ OWIN/Katana จาก ASP.NET 4 ได้ แสดงว่าคุณมาได้ครึ่งทางแล้ว: โครงการ Katana ได้กลายมาเป็น ASP.NET 5 และในที่สุดก็ถูกเปลี่ยนชื่อมาเป็น ASP.NET Core นั่นเอง

Because of the Katana legacy, the `Startup` class is front and center, and there's no more `Application_Start` or `Global.asax`. The entire pipeline is driven by middleware, and there's no longer a split between MVC and Web API: controllers can simply return views, status codes, or data. Dependency injection comes baked in, so you don't need to install and configure a container like StructureMap or Ninject if you don't want to. And the entire framework has been optimized for speed and runtime efficiency.

เอาล่ะ เกริ่นนำกันมาพอแล้ว มาเริ่มลงลึกกับ ASP.NET Core กันเถอะ!
