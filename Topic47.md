# 46. Constructor Injection in ASP.NET Core

Constructor Injection is the **most common and recommended** way to use Dependency Injection (DI) in ASP.NET Core.

Instead of creating objects using the `new` keyword, ASP.NET Core automatically provides the required dependencies through the class constructor.

> **Interview Tip:** More than 90% of dependency injection in ASP.NET Core applications uses **Constructor Injection**.

---

# What is Constructor Injection?

## Definition

Constructor Injection is a technique where the DI container supplies the required dependencies through the constructor when creating an object.

Instead of:

```csharp
public class ProductController
{
    private readonly ProductService _service = new ProductService();
}
```

ASP.NET Core injects the dependency automatically.

```csharp
public class ProductController
{
    private readonly ProductService _service;

    public ProductController(ProductService service)
    {
        _service = service;
    }
}
```

---

# Layman Example: Pizza Delivery

Imagine you order a pizza.

Without Constructor Injection

```
You

↓

Go To Restaurant

↓

Cook Pizza

↓

Eat
```

You do everything yourself.

---

With Constructor Injection

```
Restaurant

↓

Makes Pizza

↓

Delivers Pizza

↓

You Eat
```

You simply receive what you need.

The DI container is like the restaurant.

---

# Why Use Constructor Injection?

Without DI

```csharp
public class ProductController
{
    private ProductService _service =
        new ProductService();
}
```

Problems:

- Tight coupling
- Hard to test
- Hard to replace implementations
- Difficult to maintain

---

With Constructor Injection

```csharp
public class ProductController
{
    private readonly IProductService _service;

    public ProductController(
        IProductService service)
    {
        _service = service;
    }
}
```

Benefits:

- Loose coupling
- Easier testing
- Better maintainability
- Supports interfaces

---

# Complete Example

## Step 1 - Interface

```csharp
public interface IProductService
{
    string GetProducts();
}
```

---

## Step 2 - Implementation

```csharp
public class ProductService : IProductService
{
    public string GetProducts()
    {
        return "Product List";
    }
}
```

---

## Step 3 - Register Service

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
```

---

## Step 4 - Inject into Controller

```csharp
public class ProductController : ControllerBase
{
    private readonly IProductService _service;

    public ProductController(
        IProductService service)
    {
        _service = service;
    }

    [HttpGet]
    public IActionResult Get()
    {
        return Ok(_service.GetProducts());
    }
}
```

---

# How It Works

```
Application Starts
        │
        ▼
Register Services
        │
        ▼
HTTP Request
        │
        ▼
Controller Requested
        │
        ▼
DI Container Creates ProductService
        │
        ▼
Injects into Constructor
        │
        ▼
Controller Executes
```

---

# Multiple Dependencies

A constructor can receive multiple services.

```csharp
public ProductController(
    IProductService productService,
    ILogger<ProductController> logger,
    IConfiguration configuration)
{
}
```

ASP.NET Core resolves all registered dependencies automatically.

---

# Constructor Injection in Middleware

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }
}
```

Here, `RequestDelegate` is injected through the constructor.

---

# Best Practices

- Prefer constructor injection over other forms.
- Depend on interfaces, not concrete classes.
- Keep the number of constructor parameters reasonable.
- Use `readonly` fields for injected services.

---

# Common Mistakes

### Creating Services Manually

Bad

```csharp
var service = new ProductService();
```

Use DI instead.

---

### Forgetting Registration

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
```

If omitted, ASP.NET Core throws:

```
InvalidOperationException
```

---

### Too Many Constructor Parameters

```csharp
public ProductController(
    IService1 s1,
    IService2 s2,
    IService3 s3,
    IService4 s4,
    IService5 s5,
    IService6 s6)
```

This often indicates the class has too many responsibilities.

---

# Interview Questions

### What is Constructor Injection?

Constructor Injection is a Dependency Injection technique where the framework provides dependencies through the constructor when creating an object.

---

### Why is Constructor Injection preferred?

Because it promotes loose coupling, improves testability, and ensures required dependencies are available when the object is created.

---

### Can ASP.NET Core automatically resolve constructor dependencies?

Yes, provided the services are registered with the DI container.

---

# Interview Answer (2-Minute Version)

**What is Constructor Injection in ASP.NET Core?**

Constructor Injection is the recommended Dependency Injection pattern in ASP.NET Core. The DI container automatically provides the required dependencies through a class constructor instead of the class creating them manually. This reduces coupling, improves maintainability, simplifies unit testing, and supports interface-based programming. It is the most commonly used form of dependency injection in ASP.NET Core applications.

---

# 47. Method Injection in ASP.NET Core

Method Injection is a form of Dependency Injection where a dependency is provided **only to a specific method** instead of the entire class.

Unlike Constructor Injection, the service is available **only while that method executes**.

---

# What is Method Injection?

## Definition

Method Injection allows ASP.NET Core to inject a service into a controller action or another method by using attributes such as **`[FromServices]`**.

---

# Layman Example: Borrowing a Calculator

Imagine you're solving a math problem.

You don't buy a calculator.

You borrow one for a few minutes.

```
Need Calculator

↓

Borrow

↓

Use

↓

Return
```

You only use it when required.

Method Injection works the same way.

---

# Why Use Method Injection?

Suppose a controller has one action that needs an email service.

Without Method Injection

```csharp
public class OrderController
{
    private readonly IEmailService _email;

    public OrderController(IEmailService email)
    {
        _email = email;
    }
}
```

Even actions that never send emails receive the service.

---

With Method Injection

```csharp
public IActionResult SendEmail(
    [FromServices] IEmailService email)
{
    email.Send();

    return Ok();
}
```

Only that action receives the dependency.

---

# Complete Example

## Step 1 - Interface

```csharp
public interface IEmailService
{
    void Send();
}
```

---

## Step 2 - Implementation

```csharp
public class EmailService : IEmailService
{
    public void Send()
    {
        Console.WriteLine("Email Sent");
    }
}
```

---

## Step 3 - Register

```csharp
builder.Services.AddTransient<IEmailService, EmailService>();
```

---

## Step 4 - Action Method

```csharp
[HttpPost]
public IActionResult SendEmail(
    [FromServices] IEmailService email)
{
    email.Send();

    return Ok();
}
```

ASP.NET Core resolves `IEmailService` only for this action.

---

# How It Works

```
HTTP Request
        │
        ▼
Action Selected
        │
        ▼
DI Container Resolves Service
        │
        ▼
Inject into Method
        │
        ▼
Execute Method
```

---

# When Should You Use Method Injection?

Good use cases:

- Rarely used services
- Optional services
- Admin-only endpoints
- Reporting actions
- Export functionality

---

# Constructor vs Method Injection

| Constructor Injection | Method Injection |
|------------------------|------------------|
| Service available to entire class | Service available only to one method |
| Preferred approach | Use for specific scenarios |
| Dependencies are mandatory | Dependencies can be action-specific |
| Most common | Less common |

---

# Example Comparison

Constructor Injection

```csharp
public ProductController(
    IProductService service)
{
}
```

Every action can use `service`.

---

Method Injection

```csharp
public IActionResult Export(
    [FromServices] IReportService report)
{
}
```

Only the `Export` action can use `report`.

---

# Best Practices

- Prefer Constructor Injection for required dependencies.
- Use Method Injection only for dependencies needed by a small number of actions.
- Avoid mixing both approaches unnecessarily.
- Register services in the DI container before using `[FromServices]`.

---

# Common Mistakes

### Forgetting `[FromServices]`

Without it, ASP.NET Core treats the parameter as request data instead of a service.

---

### Using Method Injection Everywhere

If most methods require the same dependency, Constructor Injection is cleaner and easier to maintain.

---

### Forgetting Service Registration

An unregistered service results in a runtime exception.

---

# Interview Questions

### What is Method Injection?

Method Injection provides dependencies directly to a method instead of the constructor, typically using the `[FromServices]` attribute.

---

### When should Method Injection be used?

When only one or two actions require a specific service, making Constructor Injection unnecessary.

---

### Which is preferred?

Constructor Injection is the preferred and most commonly used approach.

Method Injection is reserved for action-specific dependencies.

---

# Easy Way to Remember

### Constructor Injection

```
Buy a Laptop

↓

Use It Every Day
```

You need it all the time.

---

### Method Injection

```
Borrow a Projector

↓

Use Once

↓

Return
```

You only need it temporarily.

---

# Interview Answer (2-Minute Version)

**What is Method Injection in ASP.NET Core?**

Method Injection is a Dependency Injection technique where services are injected directly into a method rather than the constructor. In ASP.NET Core, this is commonly done using the `[FromServices]` attribute on controller action parameters. It is useful when only a specific action requires a service, helping keep constructors smaller and avoiding unnecessary dependencies. While Method Injection is useful for specialized scenarios, Constructor Injection remains the recommended approach for dependencies used throughout a class.