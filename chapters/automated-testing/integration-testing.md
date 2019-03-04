## การทดสอบทั้งระบบ (Integration testing)

เมื่อเปรียบเทียบกับการทดสอบหน่วยย่อยแล้ว การทดสอบทั้งระบบจะครอบคลุมด้านต่าง ๆ กว้างกว่ามาก เพราะเป็นการทำให้ทุกระดับชั้นของทั้งแอปพลิเคชันได้ทำงานไม่ได้เป็นแยกออกมาเพียงแค่คลาสหรือเมธอดหนึ่ง ๆ เท่านั้น การทดสอบทั้งระบบจึงช่วยให้มั่นใจได้ว่าส่วนประกอบในแอปพลิเคชันของเราทำงานร่วมกันได้อย่างถูกต้อง: การเลือกเส้นทาง, controllers, บริการต่าง ๆ, โค้ดของฐานข้อมูล, และอื่น ๆ อีกมาก

การทดสอบทั้งระบบทำได้ช้ากว่าและซับซ้อนกว่าการทดสอบหน่วยย่อยมาก ดังนั้นเรามักพบว่าในแต่ละโปรเจกต์จะมีการทดสอบหน่วยย่อยบ่อยครั้งขณะที่การทดสอบทั้งระบบอาจถูกกระทำเพียงไม่กี่ครั้ง

เพื่อเป็นการทดสอบระดับชั้นทั้งหมด (รวมทั้งการเลือกเส้นทาง controller) การทดสอบทัั้งระบบมักสร้างคำขอ HTTP ไปยังแอปพลิเคชันในลักษณะเดียวกันกับการเรียกด้วยเว็บเบราว์เซอร์

เพื่อทำการทดสอบทั้งระบบ เราสามารถเริ่มการทำงานของแอปพลิเคชันแล้วส่งคำขอไปยัง http://localhost:5000 ด้วยตนเองได้ อย่างไรก็ตาม ASP.NET Core มีทางเลือกที่ดีกว่า: คลาส `TestServer` ซึ่งคลาสนี้จะเป็นเซอร์ฟเวอร์ให้แอปพลิเคชันของเราตลอดช่วงเวลาของการทดสอบ และจะหยุดทำงานโดยอัตโนมัติหลังการทดสอบเสร็จสิ้น

### สร้างโปรเจกต์เพื่อการทดสอบ

หากยังอยู่ใต้ไดเรกทอรีของโปรเจกต์ ให้ `cd` ขึ้นไปหนึ่งระดับเพื่อไปที่ไดเรกทอรีราก `AspNetCoreTodo` แล้วใช้คำสั่งนี้เพื่อขึ้นโครงโปรเจกต์ขึ้นมาใหม่:

```
dotnet new xunit -o AspNetCoreTodo.IntegrationTests
```

ในตอนนี้ โครงสร้างไดเรกทอรีของเราควรอยู่ในรูปแบบดังนี้:

```
AspNetCoreTodo/
    AspNetCoreTodo/
        AspNetCoreTodo.csproj
        Controllers/
        (etc...)

    AspNetCoreTodo.UnitTests/
        AspNetCoreTodo.UnitTests.csproj

    AspNetCoreTodo.IntegrationTests/
        AspNetCoreTodo.IntegrationTests.csproj
```

> คุณอาจจัดเก็บทั้งการทดสอบหน่วยย่อยและการทดสอบทั้งระบบไว้ในไดเรกทอรีเดียวกันก็ได้ แต่สำหรับโปรเจกต์ขนาดใหญ่ มักจัดเก็บแยกกันไว้เพื่อให้ง่ายต่อการรันการทดสอบแยกจากกัน

เนื่องจากโปรเจกต์ทดสอบจะใช้คลาสต่าง ๆ ที่ถูกเขียนไว้ในโปรเจกต์หลัก เราจึงจำเป็นต้องเพิ่มการอ้างอิงไปยังโปรเจกต์หลัก:

```
dotnet add reference ../AspNetCoreTodo/AspNetCoreTodo.csproj
```

เรายังจำเป็นต้องเพิ่มแพคเกจ NuGet `Microsoft.AspNetCore.TestHost` ด้วย:

```
dotnet add package Microsoft.AspNetCore.TestHost
```

ให้ลบไฟล์ `UnitTest1.cs` ที่ถูกสร้างขึ้นด้วย `dotnet new` ให้ลบไฟล์ `UnitTest1.cs` ที่ถูกสร้างขึ้นมาโดยอัตโนมัติ ในตอนนี้เราก็พร้อมที่จะเขียนการทดสอบแบบทั้งระบบแล้ว

### เขียนการทดสอบทั้งระบบ

มีบางอย่างที่จำเป็นต้องกำหนดค่าให้กับเซอร์ฟเวอร์สำหรับการทดสอบแต่ละครั้ง แทนที่จะใส่โค้ดสำหรับการตั้งค่านี้ให้รกรุงรังในการทดสอบ เราสามารถเก็บการตั้งค่านี้แยกไว้ในอีกคลาสหนึ่ง ให้สร้างคลาสใหม่ในชื่อ `TestFixture`:

**AspNetCoreTodo.IntegrationTests/TestFixture.cs**

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.TestHost;
using Microsoft.Extensions.Configuration;

namespace AspNetCoreTodo.IntegrationTests
{
    public class TestFixture : IDisposable  
    {
        private readonly TestServer _server;

        public HttpClient Client { get; }

        public TestFixture()
        {
            var builder = new WebHostBuilder()
                .UseStartup<AspNetCoreTodo.Startup>()
                .ConfigureAppConfiguration((context, config) =>
                {
                    config.SetBasePath(Path.Combine(
                        Directory.GetCurrentDirectory(),
                        "..\\..\\..\\..\\AspNetCoreTodo"));
                    
                    config.AddJsonFile("appsettings.json");
                });

            _server = new TestServer(builder);

            Client = _server.CreateClient();
            Client.BaseAddress = new Uri("http://localhost:8888");
        }

        public void Dispose()
        {
            Client.Dispose();
            _server.Dispose();
        }
    }
}
```

คลาสนี้ช่วยจัดการเกี่ยวกับการตั้งค่า `TestServer` และช่วยคงความเป็นระเบียบเรียบร้อยให้กับการทดสอบ

ในตอนนี้เราก็พร้อมที่จะเขียนการทดสอบทั้งระบบ (จริง ๆ) แล้ว ให้สร้างคลาสใหม่ชื่อ `TodoRouteShould`:

**AspNetCoreTodo.IntegrationTests/TodoRouteShould.cs**

```csharp
using System.Net;
using System.Net.Http;
using System.Threading.Tasks;
using Xunit;

namespace AspNetCoreTodo.IntegrationTests
{
    public class TodoRouteShould : IClassFixture<TestFixture>
    {
        private readonly HttpClient _client;

        public TodoRouteShould(TestFixture fixture)
        {
            _client = fixture.Client;
        }

        [Fact]
        public async Task ChallengeAnonymousUser()
        {
            // Arrange
            var request = new HttpRequestMessage(
                HttpMethod.Get, "/todo");

            // Act: request the /todo route
            var response = await _client.SendAsync(request);

            // Assert: the user is sent to the login page
            Assert.Equal(
                HttpStatusCode.Redirect,
                response.StatusCode);

            Assert.Equal(
                "http://localhost:8888/Account" +
                "/Login?ReturnUrl=%2Ftodo",
                response.Headers.Location.ToString());
        }
    }
}
```

การทดสอบนี้จะสร้างคำขอแบบไม่ระบุผู้ใช้ (ไม่ได้ล็อกอิน) ไปยังเส้นทาง `/todo` และยืนยันว่าเบราว์เซอร์จะถูกเปลี่ยนเส้นทางใหม่ให้ไปยังหน้าล็อกอิน

กรณีนี้เป็นตัวอย่างที่ดีสำหรับการทดสอบทั้งระบบ เนื่องจากจะต้องทำงานเกี่ยวพันกับหลายองค์ประกอบของแอปพลิเคชัน: ระบบเลือกเส้นทาง, controller และข้อเท็จจริงที่ว่า controller นั้น ๆ ถูกระบุให้เป็น `[Authorize]` และอื่น ๆ อีกมาก และนี่ยังเป็นการทดสอบที่ดีเนื่องจากจะช่วยให้มั่นใจได้ว่าเราจะไม่เผลอลบคุณลักษณะ `[Authorize]` โดยไม่ได้ตั้งใจซึ่งจะส่งผลให้ทุกคนสามารถเข้าถึง view สิ่งที่ต้องทำได้

## รันการทดสอบ

รันการทดสอบในเทอร์มินัลด้วย `dotnet test` หากทุกอย่างทำงานได้อย่างถูกต้อง เราจะพบกับข้อความแสดงความสำเร็จดังนี้:

```
Starting test execution, please wait...
 Discovering: AspNetCoreTodo.IntegrationTests
 Discovered:  AspNetCoreTodo.IntegrationTests
 Starting:    AspNetCoreTodo.IntegrationTests
 Finished:    AspNetCoreTodo.IntegrationTests

Total tests: 1. Passed: 1. Failed: 0. Skipped: 0.
Test Run Successful.
Test execution time: 2.0588 Seconds
```


## สรุป

การทดสอบเป็นเรื่องใหญ่และมีสิ่งที่ต้องเรียนรู้อีกมาก ในบทนี้ยังไม่ได้กล่าวถึงการทดสอบ UI หรือการทดสอบโค้ดส่วนหน้า (จาวาสคริปต์) ซึ่งคงต้องใช้หนังสืออีกเป็นเล่ม ๆ เพื่อให้ครอบคลุมประเด็นเหล่านี้ อย่างไรก็ตาม ในตอนนี้เราควรมีความรู้พื้นฐานและทักษะที่จำเป็นต่อการเรียนรู้เพิ่มเติมเกี่ยวกับการทดสอบและสามารถเขียนการทดสอบต่าง ๆ สำหรับแอปพลิเคชันของเราได้แล้ว

เอกสารของ ASP.NET Core (https://docs.asp.net) และ Stack Overflow เป็นแหล่งทรัพยากรชั้นดีสำหรับการเรียนรู้เพิ่มเติมและค้นหาคำตอบได้หากติดปัญหาใด ๆ
