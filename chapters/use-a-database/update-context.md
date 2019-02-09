## ปรับแก้บริบท

ในตอนนี้ยังไม่มีอะไรน่าสนใจในบริบทฐานข้อมูลมากนัก:

**Data/ApplicationDbContext.cs**

```csharp
public class ApplicationDbContext 
             : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        // ...
    }
}
```

อันดับแรก ให้เพิ่มคุณสมบัติ `DbSet` เข้าไปยัง `ApplicationDbContext` ที่ด้านล่างของคอนสตรัคเตอร์:

```csharp
public ApplicationDbContext(
    DbContextOptions<ApplicationDbContext> options)
    : base(options)
{
}

public DbSet<TodoItem> Items { get; set; }

// ...
```

เนื่องจาก `DbSet` ใช้แทนตารางหรือชุดข้อมูลในฐานข้อมูล การกำหนดคุณสมบัติ `DbSet<TodoItem>` แล้วตั้งชื่อว่า `Items` นั้น จึงเป็นการบอก Entity Framework Core ว่าเราต้องการเก็บเอนทิตี `TodoItem` ไว้ในตารางชื่อ `Items` นั่นเอง

ในตอนนี้ เราได้ปรับแก้คลาสบริบทเรียบร้อยแล้ว แต่ยังมีปัญหาเล็ก ๆ อีกปัญหาหนึ่ง: บริบทกับฐานข้อมูลยังไม่สอดคล้องกัน เนื่องจากยังไม่มีตาราง `Items` ในฐานข้อมูล (การปรับแก้โค้ดของคลาสบริบทไม่ได้ส่งผลให้เกิดการเปลี่ยนแปลงใด ๆ ขึ้นกับตัวฐานข้อมูล)

เพื่อปรับแก้ฐานข้อมูลให้สอดคล้องกับความเปลี่ยนแปลงที่ได้กระทำไว้กับบริบท เราจำเป็นต้องสร้าง **การย้ายข้อมูล (migration)**

> หากคุณมีฐานข้อมูลอยู่แล้ว ให้ลองค้นหา "scaffold-dbcontext existing database" จากบนเว็บ แล้วอ่านเอกสารของไมโครซอฟต์เกี่ยวกับการใช้เครื่องมือ `Scaffold-DbContext` เพื่อทำวิศวกรรมย้อนรอยโครงสร้างฐานข้อมูลของคุณให้กลายเป็น `DbContext` และคลาส model ต่าง ๆ ได้โดยอัตโนมัติ
