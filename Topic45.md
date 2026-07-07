# 44. Scoped Services in ASP.NET Core Dependency Injection (DI)

Scoped is the **most commonly used service lifetime** in ASP.NET Core applications.

A Scoped service is created **once per HTTP request** and shared throughout that request.

This makes it ideal for:

- Business logic
- Database operations
- Repository pattern
- Entity Framework `DbContext`
- Unit of Work

> **Interview Tip:** If you're unsure which lifetime to choose for a business service, **Scoped** is usually the safest and most appropriate choice.

---

# What is a Scoped Service?

## Definition

A **Scoped** service is created **once for each HTTP request**.

Every component that requests the service during the same request receives the **same instance**.

A new HTTP request gets a **new instance**.

---

# Layman Example: Hotel Room

Imagine staying at a hotel.

### Day 1

```
Check In

↓

Room 101

↓

Sleep

↓

Check Out
```

---

### Day 2

```
Check In

↓

Room 205

↓

Sleep

↓

Check Out
```

You keep the **same room** during your stay.

The next stay gets a **different room**.

That's exactly how a Scoped service works.

---

# Another Example: Food Delivery

Suppose you order food.

```
Order Created

↓

Restaurant

↓

Delivery Partner

↓

Delivered
```

The same delivery partner handles that order.

The next order gets a different delivery partner.

Each order is like an HTTP request.

---

# Registration

Scoped services are registered using:

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
```

or

```csharp
builder.Services.AddScoped<ProductService>();
```

---

# How Scoped Works

```
HTTP Request 1

↓

Create ProductService

↓

Controller A

↓

Controller B

↓

Repository

↓

Same Instance

↓

Request Ends

↓

Destroy Instance
```

---

Second request

```
HTTP Request 2

↓

Create New ProductService

↓

New Instance
```

---

# Visual Flow

```
Request 1

↓

ProductService

↓

Instance A
```

---

```
Request 2

↓

ProductService

↓

Instance B
```

Different requests never share the same Scoped instance.

---

# Example

## Service

```csharp
public class ProductService
{
    public Guid Id { get; } = Guid.NewGuid();
}
```

---

Register

```csharp
builder.Services.AddScoped<ProductService>();
```

---

Controller

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
        return Ok(_service.Id);
    }
}
```

---

Request 1

```
Id

↓

ABC123
```

---

Another component in the **same request**

```
ABC123
```

Same instance.

---

Request 2

```
XYZ999
```

New instance.

---

# Real-World Example

E-commerce Application

```
HTTP Request

↓

OrderController

↓

OrderService

↓

OrderRepository

↓

DbContext

↓

SQL Server
```

The same `OrderService` and `DbContext` instance are reused throughout that request.

---

# Why Scoped Is Useful

Imagine a customer places an order.

```
Validate Order

↓

Calculate Price

↓

Save Order

↓

Update Inventory

↓

Create Invoice
```

All operations use the **same** `DbContext`.

Benefits:

- One unit of work
- Consistent entity tracking
- Efficient database interaction

---

# Scoped vs Transient

Transient

```
Request

↓

Service A

↓

New Object

↓

Service B

↓

Another New Object
```

---

Scoped

```
Request

↓

Service A

↓

Same Object

↓

Service B

↓

Same Object
```

---

# Scoped vs Singleton

Scoped

```
Request 1

↓

Object A
```

```
Request 2

↓

Object B
```

---

Singleton

```
Request 1

↓

Object A
```

```
Request 2

↓

Same Object A
```

---

# Why Entity Framework Uses Scoped

`DbContext` is registered as Scoped by default.

```
Request

↓

DbContext

↓

Track Changes

↓

SaveChanges()

↓

Dispose
```

Each request gets its own `DbContext`, preventing cross-request data leakage and concurrency issues.

---

# What Happens Internally?

```
Application Starts
       │
       ▼
DI Container Ready
       │
       ▼
HTTP Request Arrives
       │
       ▼
Create Scoped Instance
       │
       ▼
Share Instance Within Request
       │
       ▼
Request Completes
       │
       ▼
Dispose Scoped Instance
```

---

# Multiple Services Sharing the Same Scoped Instance

```
Controller
     │
     ▼
OrderService
     │
     ▼
OrderRepository
     │
     ▼
DbContext
```

All use the same Scoped `DbContext` during that request.

---

# Common Use Cases

Scoped is ideal for:

- Business services
- Repository classes
- Entity Framework `DbContext`
- Unit of Work
- Domain services
- Request-specific processing

---

# Common Mistakes

## Registering `DbContext` as Singleton

Bad

```csharp
builder.Services.AddSingleton<AppDbContext>();
```

Problems:

- Not thread-safe
- Shared entity tracking
- Memory growth
- Concurrency issues

Correct

```csharp
builder.Services.AddDbContext<AppDbContext>();
```

`AddDbContext()` registers it as Scoped by default.

---

## Storing User Data in Singleton Instead of Scoped

Bad

```csharp
builder.Services.AddSingleton<UserService>();
```

Current user information becomes shared across requests.

Use Scoped instead if the data is request-specific.

---

## Injecting Scoped into Singleton

Bad

```csharp
public class CacheService
{
    public CacheService(ProductService service)
    {
    }
}
```

where:

```csharp
CacheService -> Singleton

ProductService -> Scoped
```

This causes a lifetime mismatch.

---

## Assuming Scoped Lives Forever

Scoped services are destroyed when the request ends.

They are **not** shared across requests.

---

# Scoped in Middleware

Custom middleware itself is typically created once, but you can inject Scoped services into the `InvokeAsync` method.

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
        ProductService service)
    {
        await _next(context);
    }
}
```

The `ProductService` is resolved per request.

---

# Best Practices

- Use Scoped for request-specific business logic.
- Use Scoped for repositories and `DbContext`.
- Avoid storing request-specific data in Singleton services.
- Do not inject Scoped services directly into Singletons.
- Keep Scoped services focused on a single responsibility.

---

# Lifetime Comparison

| Feature | Transient | Scoped | Singleton |
|----------|-----------|--------|-----------|
| Created | Every resolution | Once per HTTP request | Once per application |
| Shared | No | Within the same request | Across all requests |
| Thread Safety Required | Usually no | Usually no (per request) | Yes |
| Typical Use | Stateless utilities | Business logic, repositories, `DbContext` | Configuration, cache, logging |

---

# Request Lifecycle

```
Request Starts
       │
       ▼
Create Scoped Service
       │
       ▼
Controller
       │
       ▼
Repository
       │
       ▼
DbContext
       │
       ▼
Controller Returns Response
       │
       ▼
Dispose Scoped Service
```

---

# Easy Way to Remember

Think of **a movie ticket**.

```
Buy Ticket

↓

Watch Movie

↓

Ticket Expires
```

The ticket is valid only for that visit.

Similarly, a Scoped service is valid only for that HTTP request.

---

# Frequently Asked Interview Questions

### What is a Scoped service?

A Scoped service is created once per HTTP request and shared by all components within that request.

---

### Which lifetime is used for Entity Framework `DbContext`?

Scoped.

---

### Can multiple controllers in the same request share a Scoped service?

Yes, if they resolve the same service within the same request scope, they receive the same instance.

---

### Can a Singleton depend on a Scoped service?

No.

This creates a lifetime mismatch and is not supported directly.

---

### When should Scoped be used?

For request-specific business logic, repositories, and database operations.

---

# Interview Answer (2-Minute Version)

**What is a Scoped service in ASP.NET Core?**

A Scoped service is a Dependency Injection lifetime where one instance of the service is created for each HTTP request. During that request, every component that depends on the service receives the same instance. When the request completes, the instance is disposed. Scoped services are ideal for request-specific business logic, repositories, and Entity Framework `DbContext` because they maintain consistent state throughout a request without sharing data across different users or requests. Compared to Transient, Scoped reduces unnecessary object creation, and unlike Singleton, it avoids concurrency and shared-state issues.