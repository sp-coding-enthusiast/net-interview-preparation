# 53. Open Generic Registrations in ASP.NET Core Dependency Injection (DI)

Open Generic Registration is an advanced feature of ASP.NET Core Dependency Injection that allows you to register **one generic service** and let the DI container automatically create the correct implementation for **any type**.

Instead of registering:

- `IRepository<Product>`
- `IRepository<Order>`
- `IRepository<Customer>`

individually,

you register **one generic mapping**:

```csharp
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
```

Now ASP.NET Core can automatically create repositories for **any entity type**.

> **Interview Tip:** Open Generic Registration eliminates repetitive registrations and is heavily used in repositories, caching, logging, validation, MediatR, and many enterprise applications.

---

# What is an Open Generic?

## Definition

An **Open Generic** is a generic type where the type parameter has **not been specified yet**.

Example

```csharp
IRepository<T>
```

`T` is still unknown.

This is an **Open Generic**.

---

A **Closed Generic** has a specific type.

```csharp
IRepository<Product>
```

Here,

`T = Product`

The generic is now closed.

---

# Layman Example: Universal Phone Charger

Imagine a charger.

Old way

```
iPhone Charger

Android Charger

Tablet Charger

Laptop Charger
```

You buy one charger for every device.

---

Open Generic

```
Universal Charger

↓

Works With

↓

Any Device
```

You buy one charger,

it adapts automatically.

That's exactly what Open Generic Registration does.

---

# Another Example: Printer

Without Open Generic

```
Photo Printer

↓

Photos Only
```

```
Document Printer

↓

Documents Only
```

```
Label Printer

↓

Labels Only
```

---

With Open Generic

```
Universal Printer

↓

Photo

↓

Document

↓

Label
```

One printer works for all supported types.

---

# Why Do We Need Open Generic Registration?

Suppose you have:

```csharp
Product
```

```csharp
Order
```

```csharp
Customer
```

Each needs a repository.

Without Open Generic

```csharp
builder.Services.AddScoped<IRepository<Product>, ProductRepository>();

builder.Services.AddScoped<IRepository<Order>, OrderRepository>();

builder.Services.AddScoped<IRepository<Customer>, CustomerRepository>();
```

Imagine 100 entities.

You would need 100 registrations.

---

Instead,

register once.

```csharp
builder.Services.AddScoped(
    typeof(IRepository<>),
    typeof(Repository<>));
```

Done.

---

# Step 1 - Generic Interface

```csharp
public interface IRepository<T>
{
    IEnumerable<T> GetAll();
}
```

---

# Step 2 - Generic Implementation

```csharp
public class Repository<T> : IRepository<T>
{
    public IEnumerable<T> GetAll()
    {
        return Enumerable.Empty<T>();
    }
}
```

---

# Step 3 - Register Open Generic

```csharp
builder.Services.AddScoped(
    typeof(IRepository<>),
    typeof(Repository<>));
```

Notice

```
<>
```

No type specified.

---

# Step 4 - Inject

```csharp
public class ProductService
{
    private readonly IRepository<Product> _repository;

    public ProductService(
        IRepository<Product> repository)
    {
        _repository = repository;
    }
}
```

ASP.NET Core automatically creates:

```
Repository<Product>
```

You never registered it explicitly.

---

# Another Example

```csharp
public class OrderService
{
    public OrderService(
        IRepository<Order> repository)
    {
    }
}
```

DI automatically creates

```
Repository<Order>
```

Again,

no extra registration needed.

---

# How It Works Internally

Registration

```
IRepository<>

↓

Repository<>
```

---

Request

```
Need IRepository<Product>
```

DI Container

↓

Creates

```
Repository<Product>
```

---

Another request

```
Need IRepository<Customer>
```

Creates

```
Repository<Customer>
```

Automatically.

---

# Internal Flow

```
Application Starts

↓

Register Open Generic

↓

Request IRepository<Product>

↓

DI Creates Repository<Product>

↓

Return Instance
```

---

# Real-World Example

Suppose an e-commerce application has:

```
Product
```

```
Order
```

```
Customer
```

```
Category
```

```
Invoice
```

Without Open Generic

Need five registrations.

---

With Open Generic

One registration supports them all.

---

# Repository Pattern Example

Entity

```csharp
public class Product
{
}
```

---

Service

```csharp
public class ProductService
{
    private readonly IRepository<Product> _repository;

    public ProductService(
        IRepository<Product> repository)
    {
        _repository = repository;
    }
}
```

DI resolves

```
Repository<Product>
```

Automatically.

---

# Another Common Example

Caching

```csharp
ICacheService<T>
```

↓

```csharp
CacheService<T>
```

Registration

```csharp
builder.Services.AddSingleton(
    typeof(ICacheService<>),
    typeof(CacheService<>));
```

---

Logging

```
ILogger<T>
```

ASP.NET Core automatically provides:

```
ILogger<ProductController>
```

```
ILogger<OrderService>
```

This is another example of open generic concepts in action.

---

# Popular Frameworks Using Open Generics

Many popular libraries rely heavily on open generic registrations.

Examples include:

- Generic Repository implementations
- Validation frameworks
- Mapping libraries
- Request/Response pipelines
- Mediator-based architectures

---

# Advantages

- Eliminates repetitive registrations.
- Easy to maintain.
- Supports any type automatically.
- Encourages reusable generic components.
- Keeps `Program.cs` clean.

---

# Disadvantages

- Requires understanding of generics.
- Debugging generic type resolution can be more challenging.
- Not appropriate if each entity truly requires a completely different implementation.

---

# Open Generic vs Closed Generic

## Open Generic

```csharp
IRepository<T>
```

Type not decided.

---

## Closed Generic

```csharp
IRepository<Product>
```

Type fixed.

---

# Comparison

| Open Generic | Closed Generic |
|--------------|----------------|
| `IRepository<>` | `IRepository<Product>` |
| Reusable | Specific |
| Registered once | Registered per type |
| DI creates concrete type automatically | Already concrete |

---

# Common Mistakes

## Forgetting `typeof`

Bad

```csharp
builder.Services.AddScoped<IRepository<>, Repository<>>();
```

This does not compile.

Correct

```csharp
builder.Services.AddScoped(
    typeof(IRepository<>),
    typeof(Repository<>));
```

---

## Registering Every Entity Separately

Bad

```csharp
AddScoped<IRepository<Product>, Repository<Product>>();
```

```
AddScoped<IRepository<Order>, Repository<Order>>();
```

```
AddScoped<IRepository<Customer>, Repository<Customer>>();
```

When the implementation is identical,

prefer one open generic registration.

---

## Using Open Generic for Completely Different Logic

If every entity has significantly different behavior,

separate implementations may be more appropriate.

---

# Best Practices

- Use Open Generic Registration for reusable generic services.
- Keep generic implementations simple and reusable.
- Use specific implementations only when business logic genuinely differs.
- Prefer open generics for repositories, caching, and similar cross-cutting services.

---

# Visual Representation

Without Open Generic

```
Product Repository

↓

Registration
```

```
Order Repository

↓

Registration
```

```
Customer Repository

↓

Registration
```

Many registrations.

---

With Open Generic

```
Repository<>

↓

Works For

↓

Product

↓

Order

↓

Customer

↓

Invoice
```

One registration.

---

# Easy Way to Remember

Think of **a universal remote control**.

Instead of owning:

```
TV Remote
```

```
AC Remote
```

```
Speaker Remote
```

You own:

```
Universal Remote

↓

Works With Everything
```

Open Generic Registration is like that universal remote.

---

# Frequently Asked Interview Questions

### What is an Open Generic Registration?

It is a Dependency Injection registration where the generic type parameter is left unspecified, allowing the DI container to create concrete implementations for any requested type.

---

### How do you register an Open Generic?

```csharp
builder.Services.AddScoped(
    typeof(IRepository<>),
    typeof(Repository<>));
```

---

### What is the difference between Open Generic and Closed Generic?

An Open Generic has unspecified type parameters, such as `IRepository<T>`, while a Closed Generic specifies the type, such as `IRepository<Product>`.

---

### Why use Open Generic Registrations?

They reduce repetitive registrations, improve maintainability, and allow generic services to work with many different types automatically.

---

### Where are Open Generic Registrations commonly used?

Common scenarios include generic repositories, caching services, validation, logging, and mediator pipelines.

---

# Interview Answer (2-Minute Version)

**What are Open Generic Registrations in ASP.NET Core?**

Open Generic Registrations allow the Dependency Injection container to register a generic interface and its generic implementation only once, without specifying a concrete type. For example, registering `typeof(IRepository<>)` with `typeof(Repository<>)` enables ASP.NET Core to automatically resolve `IRepository<Product>`, `IRepository<Order>`, or any other closed generic type at runtime. This significantly reduces repetitive registrations, keeps configuration clean, and is widely used in repository patterns, caching, validation, and other reusable generic components in enterprise applications.