## ดาวน์โหลด SDK
 ให้สืบค้นด้วย "download .net core" แล้วทำตามขั้นตอนที่ระบุบนหน้าดาวน์โหลดของไมโครซอฟต์เพื่อดาวน์โหลด .NET Core SDK หลังจากติดตั้ง SDK เสร็จแล้ว ให้เปิดเทอร์มินัล (หรือเพาเวอร์เชลล์บนวินโดวส์) แล้วใช้เครื่องมือบนบรรทัดคำสั่ง (เรียกอีกอย่างว่า **CLI**) 
 `dotnet` เพื่อให้มั่นใจว่าทุกอย่างทำงานได้:

```
dotnet --version

2.1.104
```

คุณสามารถเรียกดูข้อมูลเพิ่มเติมเกี่ยวกับแพลตฟอร์มของคุณได้ด้วยแฟลก `--info`:

```
dotnet --info

.NET Command Line Tools (2.1.104)

Product Information:
 Version:            2.1.104
 Commit SHA-1 hash:  48ec687460

Runtime Environment:
 OS Name:     Mac OS X
 OS Version:  10.13

(more details...)
```

ถ้าผลลัพธ์เป็นดังที่แสดงอยู่ด้านบน แสดงว่าคุณพร้อมไปต่อแล้ว!
