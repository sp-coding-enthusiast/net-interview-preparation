# 54. Decorator Pattern in ASP.NET Core

The **Decorator Pattern** is a **Structural Design Pattern** that allows you to **add new behavior to an existing object without modifying its original code**.

Instead of changing the original service, you wrap it inside another class called a **Decorator**.

The decorator:

- Receives the original service
- Executes additional logic
- Calls the original service
- Can execute more logic after the original service

> **Interview Tip:** The Decorator Pattern is commonly used for **logging, caching, validation, authorization, metrics, auditing, retries, and performance monitoring**.

---

# What is the Decorator Pattern?

## Definition

A Decorator wraps an existing implementation of an interface and adds additional behavior while keeping the same interface.

```
Client

↓

Decorator

↓

Original Service
```

The client doesn't know whether it's calling the original service or the decorator.

---

# Layman Example: Gift Wrapping

Imagine you buy a gift.

Without decorator

```
Gift

↓

Customer
```

---

With decorator

```
Gift

↓

Gift Wrapper

↓

Customer
```

The gift is still the same,

but it now has extra decoration.

The gift wasn't changed.

It was wrapped.

---

# Another Example: Mobile Phone Cover

```
Phone

↓

Use
```

---

Add a cover

```
Phone

↓

Phone Cover

↓

Use
```

The phone still works.

The cover simply adds protection.

That's a decorator.

---

# Why Do We Need a Decorator?

Suppose we have:

```text
Email Service
```

Now the business asks for:

- Logging
- Performance measurement
- Audit trail

One approach is to modify the service.

```text
Email Service

↓

Logging

↓

Audit

↓

Timing

↓

Business Logic
```

The class becomes large and violates the **Single Responsibility Principle (SRP)**.

Instead,

create a decorator.

---

# Real-World Example

```
Controller

↓

Logging Decorator

↓

Email Service
```

The Email Service stays simple.

The Logging Decorator adds logging.

---

# Step 1 - Create Interface

```csharp
public interface INotificationService
{
    void Send(string message);
}
```

---

# Step 2 - Original Service

```csharp
public class EmailNotificationService : INotificationService
{
    public void Send(string message)
    {
        Console.WriteLine($"Sending Email: {message}");
    }
}
```

---

# Step 3 - Create Decorator

```csharp
public class LoggingNotificationDecorator : INotificationService
{
    private readonly INotificationService _inner;

    public LoggingNotificationDecorator(
        INotificationService inner)
    {
        _inner = inner;
    }

    public void Send(string message)
    {
        Console.WriteLine("Logging Started");

        _inner.Send(message);

        Console.WriteLine("Logging Finished");
    }
}
```

Notice:

The decorator implements the **same interface**.

---

# Flow

```
Controller

↓

Logging Decorator

↓

Email Service
```

Execution

```
Log Start

↓

Send Email

↓

Log End
```

---

# Another Example: Performance Monitoring

```csharp
public class PerformanceDecorator : INotificationService
{
    private readonly INotificationService _inner;

    public PerformanceDecorator(
        INotificationService inner)
    {
        _inner = inner;
    }

    public void Send(string message)
    {
        var start = DateTime.UtcNow;

        _inner.Send(message);

        Console.WriteLine(DateTime.UtcNow - start);
    }
}
```

The email service remains unchanged.

---

# Multiple Decorators

Decorators can be stacked.

```
Controller

↓

Logging

↓

Caching

↓

Performance

↓

Original Service
```

Execution

```
Logging

↓

Caching

↓

Performance

↓

Business Logic
```

Each decorator adds one responsibility.

---

# Real-World Scenarios

## Logging

```
Logging Decorator

↓

Order Service
```

---

## Caching

```
Caching Decorator

↓

Product Service
```

---

## Authorization

```
Authorization Decorator

↓

Customer Service
```

---

## Retry Logic

```
Retry Decorator

↓

Payment Service
```

---

## Metrics

```
Metrics Decorator

↓

API Client
```

---

# Decorator vs Inheritance

Inheritance

```
NotificationService

↓

LoggingNotificationService
```

↓

```
CachingLoggingNotificationService
```

↓

```
CachingLoggingAuditNotificationService
```

Classes keep growing.

---

Decorator

```
Logging

↓

Caching

↓

Audit

↓

NotificationService
```

Flexible and reusable.

---

# Decorator vs Middleware

Many people confuse these.

| Decorator | Middleware |
|-----------|------------|
| Wraps a service | Wraps an HTTP request pipeline |
| Used inside business logic | Used before the request reaches controllers |
| Implements the same interface as the wrapped service | Implements request pipeline behavior |

---

# Decorator vs Proxy

| Decorator | Proxy |
|-----------|-------|
| Adds behavior | Controls access to an object |
| Focuses on extending functionality | Focuses on access, security, or lazy loading |

---

# Registration in ASP.NET Core

The built-in DI container does **not** automatically decorate services.

A common approach is:

```csharp
builder.Services.AddScoped<EmailNotificationService>();
```

Then create the decorator by injecting the original implementation.

In larger applications, many teams use libraries such as **Scrutor**, which provides convenient support for service decoration.

---

# Internal Working

```
Request

↓

Controller

↓

Decorator

↓

Original Service

↓

Return
```

The controller never knows about the extra behavior.

---

# Advantages

- Follows the Open/Closed Principle.
- Adds behavior without modifying existing code.
- Keeps classes focused on a single responsibility.
- Easy to combine multiple decorators.
- Encourages composition over inheritance.

---

# Disadvantages

- Introduces additional classes.
- Can become difficult to trace if many decorators are chained together.
- Registration is slightly more involved than standard DI.

---

# Common Mistakes

## Modifying Original Service

Bad

```text
Email Service

↓

Logging

↓

Caching

↓

Validation

↓

Business Logic
```

One class doing everything.

---

## Creating Huge Decorators

Each decorator should have **one responsibility**.

Good

```
Logging Decorator
```

```
Caching Decorator
```

```
Retry Decorator
```

Instead of one giant decorator.

---

## Confusing Middleware with Decorators

Middleware operates on HTTP requests.

Decorators operate on services.

---

# Best Practices

- Keep decorators focused on one concern.
- Use decorators for cross-cutting concerns.
- Prefer composition over inheritance.
- Chain decorators only when each adds clear value.
- Keep business logic inside the original service.

---

# Visual Representation

Without Decorator

```
Controller

↓

Email Service
```

---

With Decorator

```
Controller

↓

Logging

↓

Email Service
```

---

Multiple Decorators

```
Controller

↓

Logging

↓

Caching

↓

Performance

↓

Email Service
```

---

# Easy Way to Remember

Think of **wearing a jacket**.

You are still the same person.

The jacket adds warmth.

If it's cold,

you add another layer.

```
You

↓

Sweater

↓

Jacket

↓

Raincoat
```

Each layer adds functionality.

That's exactly how decorators work.

---

# Frequently Asked Interview Questions

### What is the Decorator Pattern?

The Decorator Pattern is a structural design pattern that adds behavior to an object dynamically by wrapping it with another object that implements the same interface.

---

### Why use the Decorator Pattern?

It allows adding cross-cutting concerns such as logging, caching, validation, retries, and metrics without modifying the original implementation.

---

### Does the Decorator modify the original class?

No.

It wraps the original object and delegates calls to it while adding extra behavior.

---

### Can multiple decorators be combined?

Yes.

Multiple decorators can be chained together, each adding a single responsibility.

---

### Does ASP.NET Core DI support decorators automatically?

The built-in DI container does not provide automatic decoration out of the box. Developers can wire decorators manually or use helper libraries such as **Scrutor** to simplify registration.

---

# Decorator vs Middleware vs Factory

| Pattern | Purpose |
|----------|---------|
| Decorator | Add behavior to an existing service |
| Middleware | Process HTTP requests and responses |
| Factory | Create the appropriate object |

---

# Interview Answer (2-Minute Version)

**What is the Decorator Pattern in ASP.NET Core?**

The Decorator Pattern is a structural design pattern used to add additional behavior to an existing service without modifying its implementation. A decorator implements the same interface as the original service, holds a reference to that service, and delegates method calls while performing additional work before or after the original logic. It is commonly used for cross-cutting concerns such as logging, caching, auditing, retries, authorization, and performance monitoring. The pattern follows the Open/Closed Principle by extending functionality through composition rather than changing existing code, making applications easier to maintain and test.