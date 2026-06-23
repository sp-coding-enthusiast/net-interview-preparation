# Constructor Injection vs Method Injection

Dependency Injection (DI) is a technique where dependencies are provided from outside instead of being created inside a class.

The two most common DI approaches are:

1. Constructor Injection
2. Method Injection

---

# What Problem Are We Solving?

Suppose an OrderService needs an EmailService.

Without DI:

```csharp
public class OrderService
{
    private EmailService _emailService = new EmailService();
}
```

Problems:

- Tight coupling
- Difficult unit testing
- Hard to replace implementations
- Violates Dependency Inversion Principle

---

# 1. Constructor Injection

## Definition

Dependencies are provided through the constructor when the object is created.

---

## Example

```csharp
public interface IEmailService
{
    void SendEmail(string message);
}

public class EmailService : IEmailService
{
    public void SendEmail(string message)
    {
        Console.WriteLine(message);
    }
}
```

Service:

```csharp
public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public void PlaceOrder()
    {
        _emailService.SendEmail("Order Placed");
    }
}
```

Registration:

```csharp
services.AddScoped<IEmailService, EmailService>();
services.AddScoped<OrderService>();
```

ASP.NET Core automatically injects EmailService into OrderService.

---

# Layman Example

Imagine buying a car.

The car requires:

- Engine
- Battery
- Wheels

Constructor Injection means:

> The car cannot be created unless all required parts are provided.

Example:

```text
Car(
    Engine,
    Battery,
    Wheels
)
```

No missing components.

The car is fully ready at creation time.

---

# Advantages of Constructor Injection

## 1. Mandatory Dependencies

The object cannot exist without required dependencies.

Example:

```csharp
public OrderService(IEmailService emailService)
```

Compiler forces the dependency.

---

## 2. Easy Unit Testing

Production:

```csharp
var service = new OrderService(new EmailService());
```

Unit Test:

```csharp
var service = new OrderService(new MockEmailService());
```

Very easy to replace dependencies.

---

## 3. Immutability

Dependencies are usually readonly.

```csharp
private readonly IEmailService _emailService;
```

Cannot be changed accidentally.

---

## 4. Clear Design

By looking at constructor:

```csharp
public OrderService(
    IRepository repository,
    ILogger logger,
    IEmailService email)
```

you immediately know what the class depends on.

---

## 5. Recommended by ASP.NET Core

ASP.NET Core DI container is designed primarily for constructor injection.

---

# Disadvantages of Constructor Injection

## Constructor Explosion

Suppose:

```csharp
public OrderService(
    IRepository repository,
    ILogger logger,
    IEmailService email,
    ICacheService cache,
    INotificationService notification,
    IUserContext context,
    IConfiguration config)
```

Now constructor becomes huge.

This often indicates:

```text
Class has too many responsibilities
```

Possible violation of:

```text
Single Responsibility Principle (SRP)
```

---

# 2. Method Injection

## Definition

Dependencies are passed directly into a method when needed.

---

## Example

```csharp
public class OrderService
{
    public void PlaceOrder(
        IEmailService emailService)
    {
        emailService.SendEmail("Order Placed");
    }
}
```

Dependency is provided only when method is called.

---

# Layman Example

Imagine a taxi.

Constructor Injection:

```text
Buy a car with GPS permanently installed.
```

Method Injection:

```text
Rent GPS only when needed.
```

GPS is supplied only for that trip.

---

# Example

```csharp
public class ReportService
{
    public void GenerateReport(
        ILogger logger)
    {
        logger.Log("Generating Report");
    }
}
```

Logger is needed only during report generation.

---

# Advantages of Method Injection

## 1. Dependency Used Only When Needed

Example:

```csharp
public void ExportPdf(IPdfGenerator pdfGenerator)
{
}
```

No need to keep PdfGenerator permanently.

---

## 2. Reduces Constructor Size

Instead of:

```csharp
public ReportService(
    ILogger logger,
    IPdfGenerator pdf,
    IExcelGenerator excel)
```

you can pass generators when required.

---

## 3. Useful For Optional Dependencies

Example:

```csharp
public void Process(
    IAuditService? auditService = null)
{
}
```

Dependency becomes optional.

---

# Disadvantages of Method Injection

## 1. Dependency Is Hidden

Consider:

```csharp
public class OrderService
{
    public void PlaceOrder(
        IEmailService email)
    {
    }
}
```

Looking at constructor:

```csharp
public OrderService()
{
}
```

You cannot tell what dependencies are required.

---

## 2. Repeated Passing

Every call:

```csharp
service.PlaceOrder(emailService);
```

must pass dependency manually.

This becomes repetitive.

---

## 3. Risk of Null Values

Example:

```csharp
service.PlaceOrder(null);
```

Runtime failure:

```text
NullReferenceException
```

---

## 4. Harder Maintenance

Developers must remember to provide dependencies every time.

Mistakes become common.

---

# Real ASP.NET Core Example

## Constructor Injection

Controller:

```csharp
public class OrderController : ControllerBase
{
    private readonly IOrderService _service;

    public OrderController(
        IOrderService service)
    {
        _service = service;
    }
}
```

ASP.NET Core automatically injects dependency.

This is the most common pattern.

---

## Method Injection

ASP.NET Core supports method injection using attributes.

Example:

```csharp
public IActionResult CreateOrder(
    [FromServices] IEmailService emailService)
{
    emailService.SendEmail("Order Created");

    return Ok();
}
```

Dependency is injected only into that action method.

---

# Interview Question

## Which injection type is preferred in ASP.NET Core?

Answer:

```text
Constructor Injection
```

Reason:

- Dependencies are explicit
- Better testability
- Better design
- Built-in DI container optimized for it

---

# When To Use Constructor Injection?

Use when:

```text
Dependency is required for class to function
```

Examples:

- Repository
- Logger
- DbContext
- Cache
- Business Services

---

# When To Use Method Injection?

Use when:

```text
Dependency is needed only for a specific operation
```

Examples:

- PDF Generator
- Excel Export Service
- Temporary Validators
- Optional Services

---

# Comparison Table

| Feature | Constructor Injection | Method Injection |
|-----------|----------------------|------------------|
| Dependency Visibility | High | Low |
| Testability | Excellent | Good |
| ASP.NET Core Preferred | Yes | No |
| Mandatory Dependencies | Yes | No |
| Optional Dependencies | Difficult | Easy |
| Maintenance | Easier | Harder |
| Null Risk | Low | Higher |
| Usage Frequency | Most Common | Less Common |

---

# Easy Way To Remember

## Constructor Injection

Think of building a house.

Before moving in, the house must have:

- Foundation
- Walls
- Roof

Without them, the house cannot exist.

Dependencies are mandatory.

---

## Method Injection

Think of ordering food delivery.

You only provide:

```text
Delivery Address
```

when placing an order.

Dependency is supplied only when needed.

---

# Senior-Level Interview Answer

Constructor Injection is the preferred dependency injection technique in ASP.NET Core because it makes dependencies explicit, supports immutability, improves testability, and ensures required dependencies are available when the object is created. Method Injection is useful when a dependency is required only for a specific operation or is optional. In enterprise applications, Constructor Injection is used for most services, while Method Injection is reserved for specialized scenarios.