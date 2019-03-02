## กำหนดสิทธิด้วยบทบาท

บทบาท (Role) เป็นแนวทางที่ใช้กันทั่วไปเพื่อรองรับการกำหนดสิทธิและการอนุญาตในเว็บแอปพลิเคชัน ตัวอย่างเช่นการนิยมสร้างบทบาทผู้ดูแลระบบ (Administrator) ที่ให้ผู้ใช้แอดมินให้มีสิทธิหรืออำนาจเหนือกว่าผู้ใช้ทั่วไป

ในโปรเจกต์นี้ เราจะเพิ่มหน้าจัดการผู้ใช้ที่สามารถเห็นได้โดยผู้ดูแลระบบเท่านั้น หากผู้ใช้ทั่วไปพยายามเข้าถึงหน้านี้ก็จะพบกับข้อความแสดงความผิดพลาด

### เพิ่มหน้าสำหรับจัดการผู้ใช้

อันดับแรก สร้าง controller ขึ้นมาใหม่:

**Controllers/ManageUsersController.cs**

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using AspNetCoreTodo.Models;
using Microsoft.EntityFrameworkCore;

namespace AspNetCoreTodo.Controllers
{
    [Authorize(Roles = "Administrator")]
    public class ManageUsersController : Controller
    {
        private readonly UserManager<ApplicationUser>
            _userManager;
        
        public ManageUsersController(
            UserManager<ApplicationUser> userManager)
        {
            _userManager = userManager;
        }

        public async Task<IActionResult> Index()
        {
            var admins = (await _userManager
                .GetUsersInRoleAsync("Administrator"))
                .ToArray();

            var everyone = await _userManager.Users
                .ToArrayAsync();

            var model = new ManageUsersViewModel
            {
                Administrators = admins,
                Everyone = everyone
            };

            return View(model);
        }
    }
}
```

กำหนดคุณสมบัติ `Roles` ในคุณลักษณะ `[Authorize]` เพื่อให้แน่ใจว่าผู้ใช้จะต้องล็อกอินไว้แล้ว  **และ** ได้รับมอบหมายบทบาทผู้ดูแลระบบไว้จึงจะสามารถเรียกใช้หน้านี้ได้

อันดับต่อไปให้สร้าง view model:

**Models/ManageUsersViewModel.cs**

```csharp
using System.Collections.Generic;

namespace AspNetCoreTodo.Models
{
    public class ManageUsersViewModel
    {
        public ApplicationUser[] Administrators { get; set; }

        public ApplicationUser[] Everyone { get; set;}
    }
}
```

ท้ายที่สุด สร้างโฟลเดอร์ `Views/ManageUsers` และ view สำหรับแอคชัน `Index`:

**Views/ManageUsers/Index.cshtml**

```html
@model ManageUsersViewModel

@{
    ViewData["Title"] = "Manage users";
}

<h2>@ViewData["Title"]</h2>

<h3>Administrators</h3>

<table class="table">
    <thead>
        <tr>
            <td>Id</td>
            <td>Email</td>
        </tr>
    </thead>
    
    @foreach (var user in Model.Administrators)
    {
        <tr>
            <td>@user.Id</td>
            <td>@user.Email</td>
        </tr>
    }
</table>

<h3>Everyone</h3>

<table class="table">
    <thead>
        <tr>
            <td>Id</td>
            <td>Email</td>
        </tr>
    </thead>
    
    @foreach (var user in Model.Everyone)
    {
        <tr>
            <td>@user.Id</td>
            <td>@user.Email</td>
        </tr>
    }
</table>
```

เริ่มแอปพลิเคชันแล้วไปที่เส้นทาง `/ManageUsers` ขณะล็อกอินเข้าระบบเป็นผู้ใช้ทั่วไป จะพบกับหน้าปฏิเสธการเข้าถึงดังนี้:

![Access denied error](access-denied.png)

นั่นเป็นเพราะผู้ใช้ต่าง ๆ ไม่ได้รับมอบหมายบทบาทผู้ดูแลระบบโดยอัตโนมัติ


### สร้างบัญชีผู้ดูแลระบบเพื่อทำการทดสอบ

ด้วยเหตุผลด้านความปลอดภัย จะไม่มีใครลงทะเบียนสร้างบัญชีใหม่เป็นผู้ดูแลระบบด้วยตนเองได้ อันที่จริง บทบาทผู้ดูแลระบบยังไม่มีในฐานข้อมูลด้วยซ้ำ!

เราสามารถเพิ่มบทบาทผู้ดูแลระบบรวมทั้งบัญชีผู้ดูแลระบบเพื่อทำการทดสอบเข้าไปยังฐานข้อมูลตั้งแต่เริ่มรันแอปพลิเคชันเป็นครั้งแรก การเพิ่มข้อมูลในการรันครั้งแรกเข้าไปยังฐานข้อมูลเช่นนี้เรียกว่าการกำหนดค่าตั้งต้น (initialize) หรือ **การซีด (seeding)** ฐานข้อมูล

สร้างคลาสใหม่ในชื่อ `SeedData` ไว้ในไดเรกทอรีรากของโปรเจกต์:

**SeedData.cs**

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using AspNetCoreTodo.Models;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;

namespace AspNetCoreTodo
{
    public static class SeedData
    {
        public static async Task InitializeAsync(
            IServiceProvider services)
        {
            var roleManager = services
                .GetRequiredService<RoleManager<IdentityRole>>();
            await EnsureRolesAsync(roleManager);

            var userManager = services
                .GetRequiredService<UserManager<ApplicationUser>>();
            await EnsureTestAdminAsync(userManager);
        }
    }
}
```

เมธอด `InitializeAsync()` ใช้ `IServiceProvider` (ชุดของบริการต่าง ๆ ที่ถูกกำหนดไว้ในเมธอด `Startup.ConfigureServices()`) เพื่อรับค่า `RoleManager` และ `UserManager` จากระบบอัตลักษณ์ของ ASP.NET Core


เพิ่มสองเมธอดด้านล่างนี้ไว้ใต้เมธอด `InitializeAsync()` โดยเมธอดแรกคือ `EnsureRolesAsync()`:

```csharp
private static async Task EnsureRolesAsync(
    RoleManager<IdentityRole> roleManager)
{
    var alreadyExists = await roleManager
        .RoleExistsAsync(Constants.AdministratorRole);
    
    if (alreadyExists) return;

    await roleManager.CreateAsync(
        new IdentityRole(Constants.AdministratorRole));
}
```

เมธอดนี้ตรวจสอบว่ามีบทบาท `Administrator` ในฐานข้อมูลแล้วหรือไม่ หากยังไม่มีก็จะสร้างขึ้นมาให้ แทนที่จะต้องพิมพ์ `"Administrator"` บ่อยครั้ง ให้สร้างคลาสเล็ก ๆ ชื่อ `Constants` เพื่อเก็บค่านั้น:

**Constants.cs**

```csharp
namespace AspNetCoreTodo
{
    public static class Constants
    {
        public const string AdministratorRole = "Administrator";
    }
}
```

> ถ้าต้องการ คุณสามารถอัพเดต `ManageUsersController` เพื่อใช้ค่าคงที่นี้ได้เช่นกัน

อันดับต่อไปคือเมธอด `EnsureTestAdminAsync()`:

**SeedData.cs**

```csharp
private static async Task EnsureTestAdminAsync(
    UserManager<ApplicationUser> userManager)
{
    var testAdmin = await userManager.Users
        .Where(x => x.UserName == "admin@todo.local")
        .SingleOrDefaultAsync();

    if (testAdmin != null) return;

    testAdmin = new ApplicationUser
    {
        UserName = "admin@todo.local",
        Email = "admin@todo.local"
    };
    await userManager.CreateAsync(
        testAdmin, "NotSecure123!!");
    await userManager.AddToRoleAsync(
        testAdmin, Constants.AdministratorRole);
}
```

หากยังไม่มีผู้ใช้ที่มีชื่อผู้ใช้เป็น `admin@todo.local` ในฐานข้อมูล เมธอดนี้จะสร้างบัญชีนี้ขึ้นมาพร้อมกำหนดรหัสผ่านชั่วคราวให้ หลังจากล็อกอินเป็นครั้งแรกแล้วควรเปลี่ยนรหัสผ่านสำหรับบัญชีนี้เป็นอย่างอื่นที่ปลอดภัยกว่าทันที!

อันดับต่อไป เราต้องกำหนดให้แอปพลิเคชันรันกลไกนี้ขณะเริ่มการทำงาน ให้แก้ไข `Program.cs` และอัพเดต `Main()` ให้เรียกใช้เมธอดใหม่ `InitializeDatabase()`:

**Program.cs**

```csharp
public static void Main(string[] args)
{
    var host = BuildWebHost(args);
    InitializeDatabase(host);
    host.Run();
}
```

จากนั้น ให้เพิ่มเมธอดใหม่เข้าไปในคลาสโดยวางไว้ใต้ `Main()`:

```csharp
private static void InitializeDatabase(IWebHost host)
{
    using (var scope = host.Services.CreateScope())
    {
        var services = scope.ServiceProvider;

        try
        {
            SeedData.InitializeAsync(services).Wait();
        }
        catch (Exception ex)
        {
            var logger = services
                .GetRequiredService<ILogger<Program>>();
            logger.LogError(ex, "Error occurred seeding the DB.");
        }
    }
}
```

เพิ่มคำสั่ง `using` นี้ไว้ตอนต้นของไฟล์:

```csharp
using Microsoft.Extensions.DependencyInjection;
```

เมธอดนี้จะรับชุดของบริการที่ `SeedData.InitializeAsync()` จำเป็นต้องใช้ แล้วจึงรันเมธอดเพื่อซีดฐานข้อมูล หากเกิดข้อผิดพลาดใด ๆ บันทึกความผิดพลาดดังกล่าวจะถูกจัดเก็บไว้

> เนื่องจาก `InitializeAsync()` จะคืนค่า `Task` จึงต้องใช้เมธอด `Wait()` เพื่อให้แน่ใจว่าต้องดำเนินการให้แล้วเสร็จก่อนเริ่มแอปพลิเคชัน โดยทั่วไปเรามักใช้ `await` เพื่อดำเนินการนี้ แต่ด้วยเหตุผลทางเทคนิคเราไม่สามารถใช้ `await` ในคลาส `Program` ได้ นี่เป็นข้อยกเว้นที่พบไม่บ่อยนัก คุณควรใช้ `await` ในสถานการณ์อื่นทั้งหมด!

หลังจากเริ่มแอปพลิเคชันในครั้งต่อไป บัญชี `admin@todo.local` จะถูกสร้างขึ้นและกำหนดให้มีบทบาทเป็นผู้ดูแลระบบ ลองล็อกอินด้วยบัญชีนี้แล้วเปิดไปที่ `http://localhost:5000/ManageUsers` ก็จะพบรายชื่อผู้ใช้ทั้งหมดที่ลงทะเบียนไว้ในแอปพลิเคชัน

> เพื่อเพิ่มความท้าทาย ลองใส่คุณสมบัติด้านการดูแลระบบเพิ่มเติมลงในหน้านี้ ตัวอย่างเช่นคุณอาจเพิ่มปุ่มเพื่อให้ผู้ดูแลระบบสามารถลบบัญชีผู้ใช้ได้

### ตรวจสอบการกำหนดสิทธิใน view

คุณลักษณะ `[Authorize]` ช่วยให้สามารถตรวจสอบการกำหนดสิทธิใน controller หรือในเมธอดแอคชันได้โดยง่าย แล้วถ้าต้องการตรวจสอบสิทธิใน view ล่ะ? ตัวอย่างเช่น หากสามารถแสดงลิงก์เพื่อจัดการผู้ใช้ "Manage users" ในแถบนำทางหากผู้ใช้ที่ล็อกอินอยู่เป็นผู้ดูแลระบบได้ก็คงจะดี

เราสามารถฉีด `UserManager` เข้าไปใน view ได้โดยตรงเพื่อทำการตรวจสอบสิทธิ เพื่อให้ view ของเราไม่รกและเป็นระเบียบเรียบร้อย เราจะสร้าง view เฉพาะส่วนขึ้นมาใหม่ให้เพิ่มรายการลงใน navbar ในเลย์เอาต์:

**Views/Shared/_AdminActionsPartial.cshtml**

```html
@using Microsoft.AspNetCore.Identity
@using AspNetCoreTodo.Models

@inject SignInManager<ApplicationUser> signInManager
@inject UserManager<ApplicationUser> userManager

@if (signInManager.IsSignedIn(User))
{
    var currentUser = await userManager.GetUserAsync(User);

    var isAdmin = currentUser != null
        && await userManager.IsInRoleAsync(
            currentUser,
            Constants.AdministratorRole);

    if (isAdmin)
    {
        <ul class="nav navbar-nav navbar-right">
            <li>
                <a asp-controller="ManageUsers" 
                   asp-action="Index">
                   Manage Users
                </a>
            </li>
        </ul>
    }
}
```

> การตั้งชื่อ view เฉพาะส่วนสำหรับใช้งานร่วมกันนั้นนิยมเริ่มด้วยเส้นใต้หรือ `_` แต่ไม่ได้กำหนดไว้เป็นข้อบังคับ

view เฉพาะส่วนแรกนี้ใช้ `SignInManager` เพื่อตรวจสอบว่าหากผู้ใช้ยังไม่ได้ล็อกอิน ก็สามารถข้ามโค้ดที่เหลือของ view ไปได้ แต่ถ้า **มี** ผู้ใช้ล็อกอินแล้ว `UserManager` จะถูกใช้เพื่อเรียกดูรายละเอียดและตรวจสอบสิทธิด้วย `IsInRoleAsync()` หากการตรวจสอบทั้งหมดได้สำเร็จและผู้ใช้เป็นผู้ดูแลระบบ ลิงก์ **Manage users** จะถูกเพิ่มเข้าไปยัง navbar

เพื่อเพิ่ม view เฉพาะส่วนนี้เข้าไปยังเลย์เอาต์หลัก ให้แก้ไข `_Layout.cshtml` แล้วเพิ่มไปยังส่วนของ navbar:

**Views/Shared/_Layout.cshtml**

```html
<div class="navbar-collapse collapse">
    <ul class="nav navbar-nav">
        <!-- existing code here -->
    </ul>
    @await Html.PartialAsync("_LoginPartial")
    @await Html.PartialAsync("_AdminActionsPartial")
</div>
```

เมื่อเราล็อกอินด้วยบัญชีผู้ดูแลระบบแล้ว เราจะพบรายการใหม่ปรากฎขึ้นที่ด้านบนขวาดังนี้:

![Manage Users link](manage-users.png)

