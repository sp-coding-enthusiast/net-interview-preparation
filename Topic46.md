# 45. Transient Services in ASP.NET Core Dependency Injection (DI)

Transient is one of the **three built-in service lifetimes** in ASP.NET Core.

A Transient service creates a **new object every time** it is requested from the Dependency Injection (DI) container.

This makes it ideal for:

- Lightweight services
- Stateless services
- Helper classes
- Validation services
- Formatting services

> **Interview Tip:** Transient is best used when a service does not maintain any state and creating a new instance is inexpensive.

---

# What is a Transient Service?

## Definition

A **Transient** service is created **every time** it is requested from the DI container.

No instance is shared.

Every injection receives a **new object**.

---

# Layman Example: Disposable Paper Cup

Imagine visiting a coffee shop.

Every time you order coffee,

you receive a **new paper cup**.

```
Order 1

↓

Cup 1
```

---

```
Order 2

↓

Cup 2
```

---

```
Order 3

↓

Cup 3
```

The cups are never reused.

This is exactly how a Transient service works.

---

# Another Example: Taxi Ride

Each time you book a ride,

a new taxi is assigned.

```
Home → Office

↓

Taxi A
```

---

```
Office → Mall

↓

Taxi B
```

---

```
Mall → Home

↓

Taxi C
```

Every trip gets a different taxi.

---

# Registration

Register a Transient service using:

```csharp
builder.Services.AddTransient<IEmailService, EmailService>();
```

or

```csharp
builder.Services.AddTransient<EmailService>();
```

---

# How Transient Works

```
Application

↓

HTTP Request

↓

Controller

↓

Request EmailService

↓

Create New Object
```

---

Another request

```
HTTP Request

↓

Controller

↓

Request EmailService

↓

Create Another New Object
```

Even inside the **same HTTP request**, every resolution creates a new instance.

---

# Visual Flow

```
Request

↓

Controller

↓

EmailService

↓

Instance A
```

---

Later in the same request

```
Controller

↓

EmailService

↓

Instance B
```

Different object.

---

# Example

## Service

```csharp
public class EmailService
{
    public Guid Id { get; } = Guid.NewGuid();
}
```

---

Register

```csharp
builder.Services.AddTransient<EmailService>();
```

---

Controller

```csharp
public class HomeController : Controller
{
    private readonly EmailService _service;

    public HomeController(EmailService service)
    {
        _service = service;
    }

    public IActionResult Index()
    {
        return Content(_service.Id.ToString());
    }
}
```

Each time `EmailService` is resolved, a new `Guid` is generated.

---

# Demonstration

Suppose

```csharp
builder.Services.AddTransient<ProductService>();
```

Another service requests it twice.

```csharp
public class OrderService
{
    public OrderService(
        ProductService service1,
        ProductService service2)
    {
    }
}
```

Result

```
service1

↓

ABC123
```

```
service2

↓

XYZ999
```

Two different objects.

---

# Real-World Example

An online shopping application

```
Order Request

↓

ValidationService

↓

New Object
```

Next request

```
Order Request

↓

ValidationService

↓

Another New Object
```

Validation is stateless, so a new instance is perfectly fine.

---

# When Should You Use Transient?

Good candidates include:

- Email service
- SMS sender
- PDF generator
- Excel exporter
- Validation service
- Mapping service
- Data formatter
- Utility/helper service

These services usually do not store data between calls.

---

# Why Transient Is Useful

Suppose you generate a PDF.

```
Generate PDF

↓

Return PDF

↓

Object Destroyed
```

The object has completed its work.

There is no reason to reuse it.

---

# Advantages

- No shared state.
- Safe for concurrent requests because each consumer gets its own instance.
- Easy to reason about.
- Reduces the risk of unintended data sharing.

---

# Disadvantages

- More object creation.
- Higher memory allocations.
- Increased garbage collection for frequently resolved services.
- Not suitable for expensive-to-create objects if reused frequently.

---

# Transient vs Scoped

Transient

```
Request

↓

Service A

↓

Instance 1
```

```
Service B

↓

Instance 2
```

Different instances.

---

Scoped

```
Request

↓

Service A

↓

Instance 1
```

```
Service B

↓

Same Instance 1
```

Shared within the request.

---

# Transient vs Singleton

Transient

```
Request 1

↓

Instance A
```

```
Request 2

↓

Instance B
```

---

Singleton

```
Request 1

↓

Instance A
```

```
Request 2

↓

Same Instance A
```

---

# Lifetime Comparison

| Lifetime | Object Created | Shared |
|-----------|----------------|--------|
| Transient | Every resolution | No |
| Scoped | Once per HTTP request | Within the request |
| Singleton | Once per application | Across all requests |

---

# Internal Working

```
Request Arrives
       │
       ▼
Controller Needs Service
       │
       ▼
DI Container
       │
       ▼
Create New Object
       │
       ▼
Inject Service
       │
       ▼
Use Service
       │
       ▼
Eligible for Garbage Collection
```

---

# Common Mistakes

## Using Transient for Expensive Objects

Bad

```csharp
builder.Services.AddTransient<LargeCacheService>();
```

If object creation is expensive and the service can be safely shared,

Transient may not be the right choice.

---

## Expecting State to Persist

Bad

```csharp
public class CounterService
{
    public int Count;
}
```

Registered as Transient.

Every resolution creates a new object.

```
Count = 0
```

again.

State is not preserved.

---

## Using Transient for Entity Framework `DbContext`

Bad

```csharp
builder.Services.AddTransient<AppDbContext>();
```

This can create multiple `DbContext` instances within the same request, making transaction management and change tracking difficult.

Correct

```csharp
builder.Services.AddDbContext<AppDbContext>();
```

`DbContext` is Scoped by default.

---

## Assuming Transient Improves Performance

Creating new objects repeatedly increases allocations.

Transient is not automatically faster than Scoped or Singleton.

Choose the lifetime based on the service's behavior.

---

# Best Practices

- Use Transient for lightweight, stateless services.
- Avoid storing state inside Transient services.
- Keep Transient services inexpensive to create.
- Prefer Scoped for request-based business logic.
- Prefer Singleton only for shared, thread-safe services.

---

# Good vs Bad Examples

| Good Transient Services | Poor Transient Choices |
|--------------------------|------------------------|
| Email Service | `DbContext` |
| Validation Service | Large in-memory cache |
| PDF Generator | Request-wide business service |
| Formatter | Configuration service |
| Helper Utility | Shared cache manager |

---

# Easy Way to Remember

Think of **a disposable tissue**.

```
Use Once

↓

Throw Away
```

Every time you need one,

you take a **new tissue**.

That's exactly how a Transient service behaves.

---

# Frequently Asked Interview Questions

### What is a Transient service?

A Transient service is created every time it is requested from the Dependency Injection container.

---

### Is a Transient service shared?

No.

Each resolution receives a new instance.

---

### When should Transient be used?

For lightweight, stateless services such as validators, formatters, helper utilities, email senders, and PDF generators.

---

### Can multiple components in the same request receive different Transient instances?

Yes.

Each resolution creates a new object unless the same instance is explicitly passed around.

---

### Is Transient always better than Scoped?

No.

Transient increases object creation. Scoped is often a better choice for request-specific business logic because it allows components within the same request to share a single instance.

---

# Easy Comparison

```
Transient

↓

New Object

↓

Every Time
```

```
Scoped

↓

One Object

↓

Per Request
```

```
Singleton

↓

One Object

↓

Entire Application
```

---

# Interview Answer (2-Minute Version)

**What is a Transient service in ASP.NET Core?**

A Transient service is a Dependency Injection lifetime where a new instance of the service is created every time it is requested from the DI container. It is best suited for lightweight, stateless services that do not need to share data between components or requests, such as validation services, email senders, and helper utilities. Because a new object is created for every resolution, Transient services avoid shared state but result in more object allocations than Scoped or Singleton services. Therefore, they should be used for inexpensive, short-lived operations rather than services that maintain request-specific state or are costly to create.