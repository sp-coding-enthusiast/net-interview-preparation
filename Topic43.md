# 42. Service Lifetimes in ASP.NET Core Dependency Injection (DI)

One of the most important concepts in ASP.NET Core Dependency Injection (DI) is **Service Lifetime**.

A service lifetime determines:

> **"How long should the DI container keep an object alive?"**

ASP.NET Core provides **three built-in service lifetimes**:

- **Transient**
- **Scoped**
- **Singleton**

Choosing the correct lifetime is very important for:

- Performance
- Memory usage
- Thread safety
- Scalability
- Application correctness

> **Interview Tip:** Service lifetimes are among the most frequently asked ASP.NET Core interview questions.

---

# What is a Service Lifetime?

## Definition

A **Service Lifetime** defines how long an instance of a service exists before ASP.NET Core creates a new one.

Think of it as:

```
Object Created

↓

How Long Should It Live?

↓

Destroy Object
```

---

# Layman Example: Taxi

Imagine you book a taxi.

### Option 1

Every trip gets a new taxi.

```
Home → Office

Taxi 1

↓

Office → Mall

Taxi 2

↓

Mall → Home

Taxi 3
```

This is **Transient**.

---

### Option 2

One taxi for your entire day.

```
Morning

↓

Taxi

↓

Office

↓

Mall

↓

Home
```

This is **Scoped**.

---

### Option 3

One permanent chauffeur.

```
Monday

↓

Tuesday

↓

Wednesday

↓

Same Driver
```

This is **Singleton**.

---

# Three Service Lifetimes

```
Transient

↓

New Object Every Time
```

```
Scoped

↓

One Object Per Request
```

```
Singleton

↓

One Object For Entire Application
```

---

# 1. Transient Lifetime

## Definition

A **new object** is created **every time** the service is requested.

Registration

```csharp
builder.Services.AddTransient<IEmailService, EmailService>();
```

---

Request

```
Controller

↓

Service

↓

New Object
```

Another request

```
Controller

↓

Service

↓

Another New Object
```

Even within the same HTTP request, multiple injections receive different instances.

---

# Example

```csharp
builder.Services.AddTransient<ProductService>();
```

Controller

```csharp
public ProductController(ProductService service)
{
}
```

Each resolution gets a brand-new `ProductService`.

---

# Real-World Examples

Good candidates:

- Email service
- PDF generator
- Report generator
- Data formatter
- Validation service

These services usually don't maintain state.

---

# Advantages

- No shared state.
- Safe for concurrent use.
- Simple lifecycle.

---

# Disadvantages

- More object creation.
- Slightly higher memory allocation and garbage collection.

---

# 2. Scoped Lifetime

## Definition

One instance is created **per HTTP request**.

Registration

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
```

---

Request 1

```
Controller

↓

ProductService

↓

Instance A
```

Every component resolved during **that request** shares **Instance A**.

---

Request 2

```
Controller

↓

ProductService

↓

Instance B
```

A different request gets a different instance.

---

# Visual Example

```
Request 1

Controller

↓

ProductService

↓

Same Instance
```

```
Request 2

Controller

↓

ProductService

↓

New Instance
```

---

# Real-World Examples

Good candidates:

- Business services
- Repository classes
- Entity Framework `DbContext`
- Unit of Work

These services often represent request-specific state.

---

# Advantages

- Shared within a request.
- Efficient.
- Ideal for web applications.

---

# Disadvantages

- Cannot be safely injected into a Singleton directly.

---

# 3. Singleton Lifetime

## Definition

Only **one instance** is created for the **entire lifetime of the application**.

Registration

```csharp
builder.Services.AddSingleton<ICacheService, CacheService>();
```

---

Application

```
Starts

↓

Create Object

↓

Reuse Same Object

↓

Application Stops
```

---

All requests share the same instance.

```
Request 1

↓

Singleton
```

```
Request 2

↓

Same Singleton
```

---

# Real-World Examples

Good candidates:

- Configuration service
- Memory cache
- Logging providers
- Feature flag service

These services are typically shared across the application.

---

# Advantages

- Very fast.
- Only one object allocation.
- Low memory allocation overhead.

---

# Disadvantages

- Shared state must be thread-safe.
- Incorrect use can lead to stale data or concurrency issues.

---

# Visual Comparison

```
Transient

Request

↓

New Object

↓

Destroyed
```

---

```
Scoped

Request Starts

↓

One Object

↓

Shared

↓

Request Ends
```

---

```
Singleton

Application Starts

↓

One Object

↓

Shared By Everyone

↓

Application Stops
```

---

# Complete Example

```csharp
builder.Services.AddTransient<IEmailService, EmailService>();

builder.Services.AddScoped<IProductService, ProductService>();

builder.Services.AddSingleton<ICacheService, CacheService>();
```

---

# Lifetime Comparison

| Lifetime | Instance Created | Shared | Typical Use |
|-----------|------------------|--------|-------------|
| **Transient** | Every resolution | No | Lightweight, stateless services |
| **Scoped** | Once per HTTP request | Within the same request | Business services, repositories, `DbContext` |
| **Singleton** | Once per application | Across all requests | Configuration, caching, logging |

---

# How ASP.NET Core Handles Requests

```
Application Starts
       │
       ▼
Singleton Created
       │
       ▼
Request 1
       │
       ├── Scoped Created
       │
       ├── Transient A
       │
       ├── Transient B
       │
       ▼
Request Ends
       │
       ▼
Scoped Destroyed
       │
       ▼
Request 2
       │
       ├── New Scoped
       │
       ├── New Transient
       │
       ▼
Application Stops
       │
       ▼
Singleton Destroyed
```

---

# Real-World E-Commerce Example

```
Configuration Service

↓

Singleton
```

```
ProductService

↓

Scoped
```

```
EmailService

↓

Transient
```

This combination is common in production applications.

---

# Common Mistakes

## Injecting Scoped into Singleton

Example

```csharp
builder.Services.AddSingleton<OrderService>();
builder.Services.AddScoped<ProductRepository>();
```

Then

```csharp
public OrderService(ProductRepository repo)
```

❌ This is incorrect because the Singleton outlives the Scoped service.

ASP.NET Core typically throws an exception because a shorter-lived service cannot be captured by a longer-lived service.

---

## Using Singleton for Mutable Data

Bad

```csharp
Customer Name
```

stored inside a Singleton.

Every user shares the same object, leading to incorrect behavior.

---

## Making Everything Singleton

Not every service should be Singleton.

Choose the lifetime based on the service's responsibilities and state.

---

# Best Practices

- Use **Transient** for lightweight, stateless services.
- Use **Scoped** for request-based business logic and database access.
- Use **Singleton** only for shared, thread-safe services.
- Prefer interface-based registrations.
- Avoid injecting Scoped services directly into Singletons.
- Keep Singleton services stateless whenever possible.

---

# Easy Way to Remember

Think of a **Hotel**.

### Transient

```
New Guest

↓

New Room
```

Every guest gets a different room.

---

### Scoped

```
Guest Checks In

↓

Same Room

↓

Checks Out
```

One room for the duration of the stay.

---

### Singleton

```
Hotel Reception

↓

One Reception Desk

↓

Everyone Uses It
```

---

# Interview Scenario

Suppose an e-commerce application has:

```
EmailService

↓

Transient
```

because each email operation is independent.

```
ProductService

↓

Scoped
```

because it works with the current request and database context.

```
MemoryCache

↓

Singleton
```

because all users share the same cache.

---

# Frequently Asked Interview Questions

### What are the three service lifetimes in ASP.NET Core?

- Transient
- Scoped
- Singleton

---

### Which lifetime is used for Entity Framework `DbContext`?

**Scoped**, because one `DbContext` instance is typically used per HTTP request.

---

### Which lifetime is the fastest?

**Singleton**, because the object is created only once.

However, it must be thread-safe.

---

### Can a Singleton depend on a Scoped service?

No.

A Singleton should not directly depend on a Scoped service because the Scoped service has a shorter lifetime.

---

### Which lifetime is most commonly used?

**Scoped** is the most common choice for business services in web applications.

---

# Interview Answer (2-Minute Version)

**What are service lifetimes in ASP.NET Core?**

Service lifetimes define how long the Dependency Injection container keeps a service instance alive. ASP.NET Core provides three built-in lifetimes. **Transient** creates a new instance every time the service is requested and is suitable for lightweight, stateless services. **Scoped** creates one instance per HTTP request and is commonly used for business services, repositories, and Entity Framework `DbContext`. **Singleton** creates a single instance for the lifetime of the application and is appropriate for shared, thread-safe services such as configuration, caching, and logging. Choosing the correct lifetime is important for application performance, memory usage, scalability, and correctness.