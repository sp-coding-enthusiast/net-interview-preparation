# Service Lifetimes and Pitfalls (ASP.NET Core DI)

One of the most common .NET Lead and Senior Engineer interview topics is:

> "What problems can occur if we choose the wrong service lifetime?"

Many production outages happen not because of business logic bugs but because services are registered with incorrect lifetimes.

---

# Overview of Service Lifetimes

ASP.NET Core provides three service lifetimes:

```csharp
services.AddTransient<TService, TImplementation>();
services.AddScoped<TService, TImplementation>();
services.AddSingleton<TService, TImplementation>();
```

| Lifetime | Created |
|-----------|----------|
| Transient | Every time requested |
| Scoped | Once per HTTP request |
| Singleton | Once per application |

---

# 1. Transient Lifetime

## Definition

A new instance is created every time the service is requested.

```csharp
services.AddTransient<IEmailService, EmailService>();
```

---

## Example

```csharp
public class OrderService
{
    public OrderService(IEmailService emailService)
    {
    }
}
```

Every injection creates a new EmailService object.

---

## Good Use Cases

- EmailService
- PdfGenerator
- TaxCalculator
- ValidationService
- Utility classes

These services do not maintain state.

---

# Pitfall #1: Excessive Object Creation

Suppose:

```csharp
services.AddTransient<HeavyService>();
```

HeavyService:

```csharp
public class HeavyService
{
    public HeavyService()
    {
        Thread.Sleep(5000);
    }
}
```

Now every request creates a new expensive object.

Result:

```text
High CPU
High Memory
Slow Response Time
```

---

## Real Production Impact

1000 requests:

```text
1000 HeavyService objects
```

created unnecessarily.

Application becomes slow.

---

# Pitfall #2: Losing State

Suppose:

```csharp
public class CartService
{
    public int ItemCount { get; set; }
}
```

Registered as:

```csharp
services.AddTransient<CartService>();
```

Controller:

```csharp
cart.ItemCount++;
```

Next injection:

```text
ItemCount = 0
```

because a new object is created.

State is lost.

---

# When NOT To Use Transient

Avoid for:

- Caching
- Database Context
- Shared State
- Expensive Initialization

---

# 2. Scoped Lifetime

## Definition

One instance per HTTP request.

```csharp
services.AddScoped<IOrderService, OrderService>();
```

---

## Example

Request:

```text
GET /orders
```

During this request:

```text
Controller
Service
Repository
```

share the same instance.

When request ends:

```text
Instance destroyed
```

---

## Good Use Cases

- DbContext
- Repository
- Unit Of Work
- User Context
- Business Services

---

# Pitfall #3: Using Scoped For Background Jobs

Suppose:

```csharp
public class EmailBackgroundWorker
{
    private readonly AppDbContext _db;

    public EmailBackgroundWorker(AppDbContext db)
    {
        _db = db;
    }
}
```

Background services are not tied to HTTP requests.

But DbContext is Scoped.

Result:

```text
ObjectDisposedException
```

or

```text
Cannot resolve scoped service
```

---

## Correct Approach

Create a scope manually.

```csharp
using var scope = _scopeFactory.CreateScope();

var db = scope.ServiceProvider
              .GetRequiredService<AppDbContext>();
```

---

# Pitfall #4: Memory Growth Due To Long Requests

Suppose:

```text
Large File Upload
```

takes:

```text
20 minutes
```

Scoped services remain alive for entire request.

Result:

```text
Higher Memory Consumption
```

until request completes.

---

# Pitfall #5: Injecting Scoped Into Singleton

Most popular interview question.

Example:

```csharp
services.AddSingleton<CacheService>();
services.AddScoped<AppDbContext>();
```

Constructor:

```csharp
public CacheService(AppDbContext db)
{
}
```

Application startup:

```text
Runtime Error
```

Error:

```text
Cannot consume scoped service
from singleton
```

---

## Why?

Lifetime comparison:

```text
Singleton
|
|----------------------|
|                      |
Application Lifetime
```

```text
Scoped
|
Request Lifetime
```

Scoped service dies before Singleton.

Singleton would hold an invalid reference.

---

# 3. Singleton Lifetime

## Definition

One object for the entire application lifetime.

```csharp
services.AddSingleton<ICacheService, CacheService>();
```

---

## Example

Application starts:

```text
Create CacheService
```

All requests use same object.

Application stops:

```text
Destroy CacheService
```

---

## Good Use Cases

- Memory Cache
- Configuration
- Logging
- Feature Flags
- Static Lookup Data

---

# Pitfall #6: Shared Mutable State

Suppose:

```csharp
public class CounterService
{
    public int Count;
}
```

Singleton:

```csharp
services.AddSingleton<CounterService>();
```

Users:

```text
User A -> Count++
User B -> Count++
User C -> Count++
```

All share same variable.

Result:

```text
Unexpected Results
```

---

## Real Production Issue

Imagine:

```csharp
CurrentUserId
CurrentTenant
CurrentOrder
```

stored in Singleton.

Now users can see each other's data.

This becomes a major security issue.

---

# Pitfall #7: Thread Safety Problems

Singletons are shared by all requests.

Example:

```csharp
public class CounterService
{
    public int Count;

    public void Increment()
    {
        Count++;
    }
}
```

Multiple requests:

```text
Request A
Request B
Request C
```

execute simultaneously.

Expected:

```text
1
2
3
```

Actual:

```text
1
1
2
```

because of race conditions.

---

## Fix

Use synchronization.

```csharp
private readonly object _lock = new();

lock(_lock)
{
    Count++;
}
```

Or use:

```csharp
Interlocked.Increment(ref Count);
```

---

# Pitfall #8: Memory Leak With Singleton

Suppose:

```csharp
public class CacheService
{
    private List<Order> _orders = new();
}
```

Singleton lives forever.

Every request:

```csharp
_orders.Add(order);
```

After months:

```text
Millions of objects
```

stored in memory.

Result:

```text
High RAM Usage
Out Of Memory
Application Crash
```

---

# Pitfall #9: Singleton Holding Large Resources

Example:

```csharp
public class ReportService
{
    private byte[] report = new byte[500000000];
}
```

Singleton:

```text
500 MB allocated
```

for application lifetime.

Even when unused.

Result:

```text
Memory Waste
```

---

# Real-World Lifetime Choices

## DbContext

```csharp
services.AddDbContext<AppDbContext>();
```

Lifetime:

```text
Scoped
```

Reason:

One database session per request.

---

## Repository

```csharp
services.AddScoped<IRepository, Repository>();
```

Reason:

Works with DbContext.

---

## Email Service

```csharp
services.AddTransient<IEmailService, EmailService>();
```

Reason:

Stateless.

---

## Memory Cache

```csharp
services.AddSingleton<ICacheService, CacheService>();
```

Reason:

Shared application-wide.

---

# Interview Question

## What happens if Singleton depends on Scoped?

Answer:

```text
Runtime Exception
Cannot consume scoped service from singleton
```

---

## What happens if DbContext is Singleton?

Answer:

```text
Concurrency Issues
Data Corruption
Thread Safety Problems
Memory Growth
```

---

## What happens if CacheService is Transient?

Answer:

```text
Cache recreated every request
No caching benefit
Poor performance
```

---

# Easy Way To Remember

Think of vehicle rentals:

## Transient

```text
Rent a new bike for every trip
```

---

## Scoped

```text
Rent one car for the entire day
```

---

## Singleton

```text
Buy one car and keep it forever
```

---

# Senior-Level Interview Answer

Choosing the correct DI lifetime is critical for application performance and stability. Transient services are best for lightweight stateless operations. Scoped services are ideal for request-specific data such as DbContext. Singleton services are suitable for shared application-wide resources like caching and configuration. Common pitfalls include excessive object creation with Transient, injecting Scoped services into Singletons, thread-safety issues in Singletons, and memory leaks caused by long-lived objects retaining unnecessary data.