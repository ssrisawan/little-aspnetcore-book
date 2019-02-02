## Hello World ในภาษา C# #
ก่อนที่เราจะลงลึกไปกับ ASP.NET Core เรามาลองสร้างและรันแอปพลิเคชัน C# ง่าย ๆ กันก่อน

คุณสามารถดำเนินการทั้งหมดนี้ได้จากบรรทัดคำสั่ง อันดับแรกให้เปิดเทอร์มินัล (หรือเพาเวอร์เชลล์บนวินโดวส์) จากนั้นให้ไปที่ตำแหน่งที่ต้องการใช้เพื่อเก็บโปรเจกต์ของคุณ เช่นที่ไดเรกทอรี Documents:

```
cd Documents
```

ใช้คำสั่ง `dotnet` เพื่อสร้างโปรเจกต์ใหม่:

```
dotnet new console -o CsharpHelloWorld
```

โดยทั่วไป คำสั่ง `dotnet new` จะสร้างโปรเจกต์ .NET ขึ้นมาใหม่ในภาษา C# และพารามิเตอร์ `console` จะเลือกเทมเพลตสำหรับแอปพลิเคชันแบบคอนโซล (โปรแกรมที่แสดงผลออกทางจอภาพ) พารามิเตอร์ `-o CsharpHelloWorld` บอก `dotnet new` ให้สร้างไดเรกทอรีใหม่ในชื่อ `CsharpHelloWorld` สำหรับไฟล์ทั้งหมดของโปรเจกต์ จากนั้นให้เข้าไปในไดเรกทอรีนี้:

```
cd CsharpHelloWorld
```

คำสั่ง `dotnet new console` สร้างโปรแกรมภาษา C# ง่าย ๆ ที่แสดงข้อความ `Hello World!` ขึ้นบนจอภาพ โปรแกรมนี้มีสองไฟล์: ไฟล์โปรเจกต์ (มีส่วนขยายเป็น `.csproj`) และไฟล์โค้ดภาษา C# (ที่มีส่วนขยาย `.cs`) หากคุณเปิดไฟล์แรกในโปรแกรมแก้ไขข้อความหรือแก้ไขโค้ด คุณจะพบข้อความดังนี้:

**CsharpHelloWorld.csproj**

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>

</Project>
```

ไฟล์โปรเจกต์มีพื้นฐานมาจาก XML ทำหน้าที่กำหนดนิยามของข้อมูล (metadata) เกี่ยวกับโปรเจกต์ หลังจากนี้ หากคุณอ้างอิงถึงแพคเกจอื่น ๆ ก็จะปรากฎในนี้ (ลักษณะเดียวกันกับไฟล์ `package.json` สำหรับ npm) You won't have to edit this file by hand very often.

**Program.cs**

```csharp
using System;

namespace CsharpHelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

`static void Main` is the entry point method of a C# program, and by convention it's placed in a class (a type of code structure or module) called `Program`. The `using` statement at the top imports the built-in `System` classes from .NET and makes them available to the code in your class.

From inside the project directory, use `dotnet run` to run the program. You'll see the output written to the console after the code compiles:

```
dotnet run

Hello World!
```

That's all it takes to scaffold and run a .NET program! Next, you'll do the same thing for an ASP.NET Core application.
