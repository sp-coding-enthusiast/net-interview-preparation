# 41. Explain Built-in Dependency Injection (DI) in ASP.NET Core

Dependency Injection (DI) is one of the **core features** of ASP.NET Core.

It allows objects to receive the dependencies they need **automatically**, instead of creating them manually.

DI makes applications:

- Loosely coupled
- Easier to test
- Easier to maintain
- More scalable
- More reusable

> **Interview Tip:** Dependency Injection is one of the most frequently asked ASP.NET Core interview topics for Senior Developer, Lead Engineer, and Solution Architect roles.

---

# What is Dependency Injection?

## Definition

Dependency Injection is a design pattern in which the framework creates and provides the required objects (dependencies) to a class instead of the class creating them itself.

In simple words:

> **"Don't create your dependencies. Ask for them."**

---

# Layman Example: Restaurant

Imagine you order coffee in a restaurant.

### Without DI

You go to:

- Buy coffee beans
- Buy milk
- Buy sugar
- Make coffee yourself

```
Customer

↓

Buys Ingredients

↓

Makes Coffee
```

---

### With DI

You simply order.

```
Customer

↓

Waiter

↓

Coffee Served
```

The restaurant prepares everything.

ASP.NET Core works exactly like the restaurant.

---

# Another Example: Car Driver

Without DI

```
Driver

↓

Build Engine

↓

Install Engine

↓

Drive
```

---

With DI

```
Driver

↓

Receives Car

↓

Drive
```

The driver doesn't build the engine.

The engine is injected.

---

# What is a Dependency?

Suppose we have

```csharp
public class ProductService
{
}
```

Now

```csharp
public class ProductController
{
    private readonly ProductService _service;

    public ProductController()
    {
        _service = new ProductService();
    }
}
```

Here,

```
ProductController

↓

depends on

↓

ProductService
```

So,

**ProductService** is the dependency.

---

# Problem Without DI

```csharp
public class ProductController
{
    private readonly ProductService _service =
        new ProductService();
}
```

Problems:

- Tight coupling
- Difficult to test
- Difficult to replace implementation
- Difficult to mock
- Harder to maintain

---

# With DI

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

Now,

ASP.NET Core creates `ProductService` automatically.

---

# Who Creates the Object?

ASP.NET Core has a built-in **Dependency Injection Container**.

```
Application

↓

DI Container

↓

Creates Objects

↓

Injects Them

↓

Controller
```

The controller never calls `new ProductService()`.

---

# Built-in DI Container

```
Controller

↓

Needs Service

↓

DI Container

↓

Creates Service

↓

Injects Service
```

---

# Registering Services

Before a service can be injected, it must be registered.

```csharp
builder.Services.AddSingleton<ProductService>();
```

or

```csharp
builder.Services.AddScoped<ProductService>();
```

or

```csharp
builder.Services.AddTransient<ProductService>();
```

---

# Constructor Injection

The most common type of DI.

```csharp
public class ProductController : ControllerBase
{
    private readonly ProductService _service;

    public ProductController(ProductService service)
    {
        _service = service;
    }
}
```

ASP.NET Core automatically injects the service.

---

# Complete Example

## Step 1 - Service

```csharp
public class ProductService
{
    public string GetProducts()
    {
        return "Products";
    }
}
```

---

## Step 2 - Register

```csharp
builder.Services.AddScoped<ProductService>();
```

---

## Step 3 - Inject

```csharp
public class ProductController : ControllerBase
{
    private readonly ProductService _service;

    public ProductController(ProductService service)
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

# Interface-Based DI (Recommended)

Instead of depending on a concrete class,

depend on an interface.

---

Interface

```csharp
public interface IProductService
{
    string GetProducts();
}
```

---

Implementation

```csharp
public class ProductService : IProductService
{
    public string GetProducts()
    {
        return "Products";
    }
}
```

---

Registration

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
```

---

Injection

```csharp
public ProductController(IProductService service)
{
    _service = service;
}
```

Benefits:

- Loose coupling
- Easier unit testing
- Easy to replace implementations

---

# How DI Works Internally

```
Application Starts
        │
        ▼
Register Services
        │
        ▼
DI Container Stores Registrations
        │
        ▼
Request Arrives
        │
        ▼
Controller Requested
        │
        ▼
DI Container Creates Dependencies
        │
        ▼
Controller Created
        │
        ▼
Controller Executes
```

---

# Types of Dependency Injection

ASP.NET Core mainly supports:

## 1. Constructor Injection (Recommended)

```csharp
public ProductController(ProductService service)
```

Most commonly used.

---

## 2. Method Injection

```csharp
public IActionResult Get(
    [FromServices] ProductService service)
{
    return Ok();
}
```

Injects a service into a specific action.

---

## 3. Property Injection

Not supported automatically by the built-in DI container.

Requires third-party containers or manual assignment.

---

# Middleware with DI

Custom middleware can also receive services.

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(
        HttpContext context,
        ILogger<LoggingMiddleware> logger)
    {
        logger.LogInformation("Request received");

        await _next(context);
    }
}
```

The `ILogger` is injected automatically.

---

# Real-World Example

E-commerce Application

```
ProductsController

↓

IProductService

↓

ProductRepository

↓

DbContext

↓

SQL Server
```

Every dependency is resolved automatically by the DI container.

---

# Benefits of Built-in DI

- Reduces tight coupling
- Improves testability
- Simplifies maintenance
- Supports inversion of control (IoC)
- Built into ASP.NET Core (no external library required)
- Automatically manages service lifetimes

---

# Common Mistakes

### Creating Services Manually

Bad

```csharp
var service = new ProductService();
```

Always prefer constructor injection.

---

### Injecting Concrete Classes Everywhere

Prefer interfaces.

```csharp
IProductService
```

instead of

```csharp
ProductService
```

---

### Forgetting to Register the Service

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
```

If you don't register it, ASP.NET Core throws:

```
InvalidOperationException:
Unable to resolve service...
```

---

### Injecting Too Many Dependencies

If a constructor has many parameters, it may indicate the class has too many responsibilities.

```csharp
public ProductController(
    ILogger logger,
    ProductService service,
    EmailService email,
    SmsService sms,
    PaymentService payment,
    CacheService cache)
```

Consider refactoring.

---

# Best Practices

- Prefer constructor injection.
- Depend on interfaces instead of concrete classes.
- Register services with the appropriate lifetime.
- Keep classes focused on a single responsibility.
- Avoid using `new` for services managed by DI.
- Let the DI container manage object creation.

---

# DI Pipeline

```
Application Starts
        │
        ▼
Register Services
        │
        ▼
DI Container
        │
        ▼
Incoming Request
        │
        ▼
Controller Requested
        │
        ▼
Resolve Dependencies
        │
        ▼
Create Controller
        │
        ▼
Execute Action
        │
        ▼
Return Response
```

---

# Frequently Asked Interview Questions

### What is Dependency Injection?

Dependency Injection is a design pattern where the framework provides the required dependencies to a class instead of the class creating them itself.

---

### Why use Dependency Injection?

It promotes loose coupling, improves testability, enhances maintainability, and simplifies object creation.

---

### Which type of injection is most commonly used in ASP.NET Core?

Constructor Injection.

---

### Does ASP.NET Core have a built-in DI container?

Yes.

ASP.NET Core includes a built-in Dependency Injection container, so no external framework is required for most applications.

---

### What happens if a service is not registered?

ASP.NET Core throws an `InvalidOperationException` indicating that it cannot resolve the requested service.

---

# Easy Way to Remember

Think of DI as a **Restaurant**.

```
You

↓

Order Food

↓

Kitchen Prepares

↓

Waiter Serves
```

You don't cook the food yourself.

Similarly,

your controller doesn't create its own dependencies.

The DI container provides them.

---

# Interview Answer (2-Minute Version)

**What is the built-in Dependency Injection (DI) feature in ASP.NET Core?**

Dependency Injection is a built-in feature of ASP.NET Core that automatically creates and injects the required dependencies into classes. Instead of manually instantiating objects using the `new` keyword, services are registered with the DI container and resolved when needed. The most common approach is constructor injection, where dependencies are provided through the class constructor. ASP.NET Core supports interface-based dependency injection, making applications loosely coupled, easier to unit test, and simpler to maintain. The framework also manages service lifetimes such as Singleton, Scoped, and Transient, allowing efficient object lifecycle management.