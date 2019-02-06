## การฉีด dependency
ย้อนกลับไปที่ `TodoController` เพื่อเพิ่มโค้ดสำหรับทำงานกับ `ITodoItemService` ดังนี้:

```csharp
public class TodoController : Controller
{
    private readonly ITodoItemService _todoItemService;

    public TodoController(ITodoItemService todoItemService)
    {
        _todoItemService = todoItemService;
    }

    public IActionResult Index()
    {
        // Get to-do items from database

        // Put items into a model

        // Pass the view to a model and render
    }
}
```

เนื่องจาก `ITodoItemService` อยู่ในเนมสเปซ `Services` เราจึงต้องเพิ่มคำสั่ง `using` ไว้ตอนต้นของไฟล์ดังนี้:

```csharp
using AspNetCoreTodo.Services;
```

บรรทัดแรกของคลาสเป็นการประกาศตัวแปรแบบ private เพื่อเก็บการอ้างอิงไปยัง `ITodoItemService` ซึ่งตัวแปรนี้ช่วยให้สามารถใช้บริการจากเมธอดแอคชัน `Index` ได้ในภายหลัง (คุณจะได้เห็นวิธีการในไม่กี่นาทีนี้)

บรรทัด `public TodoController(ITodoItemService todoItemService)` เป็นการกำหนด **คอนสตรัคเตอร์ (constructor)** ให้กับคลาส โดยคอนสตรัคเตอร์คือเมธอดพิเศษที่จะถูกเรียกเมื่อต้องการสร้างอินสแตนซ์ของคลาสขึ้นมาใหม่ (ในกรณีนี้คือคลาส `TodoController`) การระบุพารามิเตอร์ `ITodoItemService` ให้กับคอนสตรัคเตอร์นั้นเป็นการประกาศว่าหากต้องการสร้าง `TodoController` คุณจะต้องป้อนวัตถุที่เข้ากันได้กับอินเตอร์เฟส `ITodoItemService` เท่านั้น

> อินเตอร์เฟสเป็นสิ่งที่ยอดเยี่ยมมากเพราะสามารถช่วยให้แยกส่วนของตรรกะในแอปพลิเคชันของคุณออกมาได้ เนื่องจาก controller ต้องพึ่งพาหรือขึ้นอยู่กับอินเตอร์เฟส `ITodoItemService` โดยไม่ได้ระบุว่าต้องเป็นคลาสใด*โดยเฉพาะ* ดังนั้นจึงไม่จำเป็นต้องรู้ว่าคลาสที่ถูกป้อนเข้าไปคือคลาสใด ซึ่งอาจเป็นคลาส `FakeTodoItemService`, คลาสอื่นที่สื่อสารกับฐานข้อมูลจริง ๆ หรืออาจเป็นอย่างอื่นโดยสิ้นเชิง! ขอเพียงแค่ให้เข้ากันได้กับอินเตอร์เฟสที่กำหนดเท่านั้น controller ก็จะสามารถใช้คลาสดังกล่าวได้ จึงช่วยให้สามารถทดสอบส่วนต่าง ๆ ของแอปพลิเคชันแยกจากกันได้โดยง่าย โดยการทดสอบดเช่นนี้จะถูกกล่าวถึงโดยละเอียดอีกครั้งในบท *การทดสอบโดยอัตโนมัติ*

Now you can finally use the `ITodoItemService` (via the private variable you declared) in your action method to get to-do items from the service layer:

```csharp
public IActionResult Index()
{
    var items = await _todoItemService.GetIncompleteItemsAsync();

    // ...
}
```

Remember that the `GetIncompleteItemsAsync` method returned a `Task<TodoItem[]>`? Returning a `Task` means that the method won't necessarily have a result right away, but you can use the `await` keyword to make sure your code waits until the result is ready before continuing on.

The `Task` pattern is common when your code calls out to a database or an API service, because it won't be able to return a real result until the database (or network) responds. If you've used promises or callbacks in JavaScript or other languages, `Task` is the same idea: the promise that there will be a result - sometime in the future.

> If you've had to deal with "callback hell" in older JavaScript code, you're in luck. Dealing with asynchronous code in .NET is much easier thanks to the magic of the `await` keyword! `await` lets your code pause on an async operation, and then pick up where it left off when the underlying database or network request finishes. In the meantime, your application isn't blocked, because it can process other requests as needed. This pattern is simple but takes a little getting used to, so don't worry if this doesn't make sense right away. Just keep following along!

The only catch is that you need to update the `Index` method signature to return a `Task<IActionResult>` instead of just `IActionResult`, and mark it as `async`:

```csharp
public async Task<IActionResult> Index()
{
    var items = await _todoItemService.GetIncompleteItemsAsync();

    // Put items into a model

    // Pass the view to a model and render
}
```

You're almost there! You've made the `TodoController` depend on the `ITodoItemService` interface, but you haven't yet told ASP.NET Core that you want the `FakeTodoItemService` to be the actual service that's used under the hood. It might seem obvious right now since you only have one class that implements `ITodoItemService`, but later you'll have multiple classes that implement the same interface, so being explicit is necessary.

Declaring (or "wiring up") which concrete class to use for each interface is done in the `ConfigureServices` method of the `Startup` class. Right now, it looks something like this:

**Startup.cs**

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // (... some code)

    services.AddMvc();
}
```

The job of the `ConfigureServices` method is adding things to the **service container**, or the collection of services that ASP.NET Core knows about. The `services.AddMvc` line adds the services that the internal ASP.NET Core systems need (as an experiment, try commenting out this line). Any other services you want to use in your application must be added to the service container here in `ConfigureServices`.

Add the following line anywhere inside the `ConfigureServices` method:

```csharp
services.AddSingleton<ITodoItemService, FakeTodoItemService>();
```

This line tells ASP.NET Core to use the `FakeTodoItemService` whenever the `ITodoItemService` interface is requested in a constructor (or anywhere else).

`AddSingleton` adds your service to the service container as a **singleton**. This means that only one copy of the `FakeTodoItemService` is created, and it's reused whenever the service is requested. Later, when you write a different service class that talks to a database, you'll use a different approach (called **scoped**) instead. I'll explain why in the *Use a database* chapter.

That's it! When a request comes in and is routed to the `TodoController`, ASP.NET Core will look at the available services and automatically supply the `FakeTodoItemService` when the controller asks for an `ITodoItemService`. Because the services are "injected" from the service container, this pattern is called **dependency injection**.
