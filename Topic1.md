# Dependency Injection (DI) Lifetimes in ASP.NET Core

## What is Dependency Injection?

Dependency Injection (DI) is a design pattern where objects receive their dependencies from an external container instead of creating them themselves.

### Without DI

```csharp
public class OrderService
{
    private readonly EmailService _emailService;

    public OrderService()
    {
        _emailService = new EmailService();
    }
}
```

Problems:

- Tight coupling
- Difficult unit testing
- Hard to replace implementations
- Poor maintainability

### With DI

```csharp
public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
}
```

Now the DI container creates and injects the dependency.

---

# Service Lifetimes

The most important DI interview topic is:

**How long should an object live?**

ASP.NET Core provides three lifetimes:

1. Transient
2. Scoped
3. Singleton

---

# 1. Transient

```csharp
services.AddTransient<IEmailService, EmailService>();
```

## Meaning

A new instance is created every time it is requested.

---

## Layman Example

Imagine a paper cup.

Whenever someone wants water:

- Take a new cup
- Use it
- Throw it away

Next person gets another new cup.

---

## Object Creation

```csharp
public HomeController(
    IServiceA service1,
    IServiceA service2)
{
}
```

Registration:

```csharp
services.AddTransient<IServiceA, ServiceA>();
```

Result:

```text
service1 != service2
```

Two different objects are created.

---

## Request Flow

Request 1

```text
Create ServiceA
Use ServiceA
Destroy ServiceA
```

Request 2

```text
Create ServiceA
Use ServiceA
Destroy ServiceA
```

Request 3

```text
Create ServiceA
Use ServiceA
Destroy ServiceA
```

Every request receives a brand-new instance.

---

## Best Use Cases

- EmailService
- PdfGenerator
- TaxCalculator
- ReportGenerator
- Validation Services
- Utility Services

These services do not maintain state.

---

## Advantages

- No shared state
- Thread-safe by nature
- Predictable behavior

---

## Disadvantages

- More memory allocations
- More object creation overhead
- Can impact performance if overused

---

# 2. Scoped

```csharp
services.AddScoped<IOrderService, OrderService>();
```

## Meaning

One object is created per HTTP request.

---

## Layman Example

Think about visiting a restaurant.

When you enter:

- You get one table
- One waiter
- One bill

Everything remains the same during your visit.

When you leave:

- The table is cleaned
- The bill is closed

The next customer gets a fresh setup.

---

## Request Lifecycle

User calls:

```text
GET /orders
```

ASP.NET Core creates:

```text
OrderService
Repository
DbContext
```

These instances are shared within the same request.

After the response:

```text
All instances destroyed
```

---

## Example

Registration:

```csharp
services.AddScoped<IOrderService, OrderService>();
```

Request 1:

```text
OrderService Instance = 100
```

Every component within Request 1 receives:

```text
100
100
100
```

Same instance.

Request 2:

```text
OrderService Instance = 200
```

New request = new object.

---

## Visual Representation

Request 1

```text
Controller
   |
Service
   |
Repository

Same Instance
```

Request Ends

```text
Destroyed
```

Request 2

```text
Controller
   |
Service
   |
Repository

New Instance
```

---

## Best Use Cases

- DbContext
- Repository
- Unit Of Work
- User Context
- Request-specific Services

---

## Advantages

- Better performance than Transient
- Consistent data throughout request
- Perfect for database operations

---

## Disadvantages

- Cannot be injected into Singleton
- Lifetime mismatch can cause runtime exceptions

---

# 3. Singleton

```csharp
services.AddSingleton<ICacheService, CacheService>();
```

## Meaning

Only one instance is created for the entire application lifetime.

---

## Layman Example

Think of a company CEO.

There is only:

```text
1 CEO
```

Every employee interacts with the same CEO.

No new CEO is created for each employee.

---

## Application Lifecycle

Application Starts

```text
Create CacheService
```

Request 1

```text
Use CacheService
```

Request 2

```text
Use CacheService
```

Request 3

```text
Use CacheService
```

Application Stops

```text
Destroy CacheService
```

---

## Example

Registration:

```csharp
services.AddSingleton<ICacheService, CacheService>();
```

Request 1

```text
CacheService = 500
```

Request 2

```text
CacheService = 500
```

Request 3

```text
CacheService = 500
```

Same object everywhere.

---

## Best Use Cases

- Memory Cache
- Configuration Service
- Logging Service
- Feature Flags
- Application Settings
- Expensive Initialization Services

---

## Advantages

- Fastest lifetime
- Minimal object creation
- Lower memory allocations

---

## Disadvantages

- Shared state across all users
- Requires thread safety
- Bugs can affect all requests

---

# Comparison Table

| Feature | Transient | Scoped | Singleton |
|----------|----------|----------|----------|
| Lifetime | Every Resolution | Per HTTP Request | Entire Application |
| Instance Creation | Every Time | Once Per Request | Once |
| Memory Usage | High | Medium | Low |
| Performance | Lowest | Medium | Highest |
| Shared Data | No | Per Request | Entire Application |
| Thread Safe Required | No | Usually No | Yes |

---

# Real Production Example

Suppose we have:

```csharp
public class CounterService
{
    public int Count;
}
```

Registered as:

```csharp
services.AddSingleton<CounterService>();
```

User A:

```text
Count = 1
```

User B:

```text
Count = 2
```

User C:

```text
Count = 3
```

All users share the same value.

This can lead to:

- Race conditions
- Incorrect data
- Threading issues

---

# Most Common Interview Question

## What lifetime should DbContext use?

Answer:

```text
Scoped
```

Reason:

- One database context per request
- Maintains transaction consistency
- Avoids concurrency issues

Example:

```csharp
services.AddDbContext<AppDbContext>();
```

Internally it is registered as Scoped.

---

# Senior-Level Interview Question

## Can a Singleton depend on a Scoped Service?

Example:

```csharp
public class CacheService
{
    private readonly AppDbContext _db;

    public CacheService(AppDbContext db)
    {
        _db = db;
    }
}
```

Registrations:

```csharp
services.AddSingleton<CacheService>();
services.AddScoped<AppDbContext>();
```

Result:

```text
Runtime Exception
```

Error:

```text
Cannot consume scoped service
from singleton
```

### Why?

Singleton Lifetime

```text
Application Start
|
|
|
Application End
```

Scoped Lifetime

```text
Request Start
|
Request End
```

The Singleton survives longer than the Scoped object.

After request completion:

```text
DbContext destroyed
```

But Singleton still references it.

This creates an invalid dependency.

---

# Easy Way to Remember

## Transient

```text
New paper cup every time
```

## Scoped

```text
One hotel room for one stay
```

## Singleton

```text
Own one house for life
```

---

# 30-Second Interview Answer

Dependency Injection in ASP.NET Core manages object creation and lifetime. Transient services create a new instance every time they are requested. Scoped services create one instance per HTTP request and are commonly used for DbContext. Singleton services create a single instance for the application's entire lifetime and are shared across all requests. Choosing the correct lifetime is important because incorrect lifetimes can cause performance issues, memory leaks, threading problems, and runtime exceptions.
