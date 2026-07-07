# 51. Circular Dependencies in ASP.NET Core Dependency Injection (DI)

A **Circular Dependency** occurs when two or more services depend on each other, creating an endless loop.

ASP.NET Core's built-in Dependency Injection container **detects circular dependencies** and throws an exception instead of entering an infinite loop.

> **Interview Tip:** Circular dependency questions are common in Senior, Lead, and Architect interviews because they test your understanding of software design principles, not just Dependency Injection.

---

# What is a Circular Dependency?

## Definition

A Circular Dependency happens when:

- Service A depends on Service B
- Service B depends on Service A

Neither service can be created first.

This creates a cycle.

---

# Layman Example: Two Friends

Imagine two friends.

**Alice says:**

> "I'll leave only after Bob leaves."

**Bob says:**

> "I'll leave only after Alice leaves."

```
Alice

↓

Waiting for Bob
```

```
Bob

↓

Waiting for Alice
```

Result

```
Nobody Leaves
```

This is a circular dependency.

---

# Another Example: Two Keys

Imagine two locked boxes.

```
Box A

↓

Key Inside Box B
```

```
Box B

↓

Key Inside Box A
```

You cannot open either box first.

---

# Simple Circular Dependency

```
Service A

↓

Needs Service B

↓

Needs Service A
```

Graphically:

```
Service A
     │
     ▼
Service B
     ▲
     │
─────┘
```

The dependency graph forms a circle.

---

# Example

## Step 1 - Service A

```csharp
public class OrderService
{
    private readonly PaymentService _payment;

    public OrderService(PaymentService payment)
    {
        _payment = payment;
    }
}
```

---

## Step 2 - Service B

```csharp
public class PaymentService
{
    private readonly OrderService _order;

    public PaymentService(OrderService order)
    {
        _order = order;
    }
}
```

---

## Registration

```csharp
builder.Services.AddScoped<OrderService>();

builder.Services.AddScoped<PaymentService>();
```

---

# What Happens Internally?

ASP.NET Core tries to create `OrderService`.

```
Create OrderService
        │
        ▼
Needs PaymentService
```

Now it tries to create `PaymentService`.

```
Create PaymentService
        │
        ▼
Needs OrderService
```

But `OrderService` is still being created.

The container cannot continue.

```
OrderService

↓

PaymentService

↓

OrderService

↓

PaymentService

↓

OrderService

...

Forever
```

Instead of looping forever,

ASP.NET Core throws an exception.

---

# Runtime Exception

You'll typically see an exception similar to:

```
InvalidOperationException:

A circular dependency was detected
for the service of type
'OrderService'.
```

The exception usually shows the dependency chain to help identify the cycle.

---

# Real-World Example

Imagine an e-commerce application.

```
OrderService

↓

PaymentService

↓

InvoiceService

↓

OrderService
```

A cycle has formed.

```
Order

↓

Payment

↓

Invoice

↓

Order
```

The DI container cannot construct any of them.

---

# Why Are Circular Dependencies Bad?

They cause:

- Tight coupling
- Difficult testing
- Poor maintainability
- Complex object graphs
- Runtime failures

They also violate the **Single Responsibility Principle (SRP)** because services become overly dependent on each other.

---

# How to Fix Circular Dependencies

## Solution 1 - Refactor Responsibilities (Recommended)

Instead of both services depending on each other,

move the shared logic into a separate service.

Before

```
OrderService

↓

PaymentService

↓

OrderService
```

---

After

```
OrderService

↓

OrderProcessor

↑

PaymentService
```

Now there is no cycle.

---

# Example

```csharp
public class OrderProcessor
{
    public void Process()
    {
    }
}
```

Both services depend on `OrderProcessor`.

```csharp
public class OrderService
{
    private readonly OrderProcessor _processor;

    public OrderService(OrderProcessor processor)
    {
        _processor = processor;
    }
}
```

```csharp
public class PaymentService
{
    private readonly OrderProcessor _processor;

    public PaymentService(OrderProcessor processor)
    {
        _processor = processor;
    }
}
```

---

# Solution 2 - Depend on Abstractions

Sometimes the cycle exists because services know too much about each other.

Instead of concrete classes,

introduce interfaces and redesign responsibilities.

```
OrderService

↓

IOrderNotifier

↓

EmailService
```

Not every service should directly reference another service.

> **Note:** Simply replacing classes with interfaces does **not** remove a circular dependency if the dependency graph is still circular. The design itself must change.

---

# Solution 3 - Event-Driven Design

Instead of directly calling another service,

publish an event.

Example

```
Order Created

↓

Publish Event

↓

Payment Service Listens

↓

Process Payment
```

The services no longer depend on each other directly.

This approach is common in large applications and microservices.

---

# Solution 4 - Lazy Resolution (Use Sparingly)

Sometimes you can delay object creation.

Example

```csharp
public class PaymentService
{
    private readonly Lazy<OrderService> _order;

    public PaymentService(Lazy<OrderService> order)
    {
        _order = order;
    }
}
```

The object is created only when first accessed.

However, this often hides a design problem rather than solving it.

---

# Solution 5 - IServiceProvider (Last Resort)

You can resolve a dependency manually.

```csharp
public class PaymentService
{
    private readonly IServiceProvider _provider;

    public PaymentService(IServiceProvider provider)
    {
        _provider = provider;
    }

    public void Process()
    {
        var order =
            _provider.GetRequiredService<OrderService>();
    }
}
```

This breaks constructor cycles,

but it should be used sparingly because it introduces the **Service Locator** pattern, which hides dependencies and makes code harder to test.

---

# Internal Resolution Flow

```
Request

↓

DI Container

↓

Resolve Controller

↓

Resolve Service A

↓

Resolve Service B

↓

Resolve Service A

↓

Cycle Detected

↓

Throw Exception
```

---

# Real-World Scenario

Suppose:

```
UserService

↓

NotificationService
```

Notification service sends emails.

Later,

Notification service starts requesting `UserService`.

```
UserService

↓

NotificationService

↓

UserService
```

Circular dependency created.

Better design:

```
UserService

↓

Event

↓

NotificationService
```

No direct dependency.

---

# Common Mistakes

## Services Doing Too Much

Large services often accumulate unrelated responsibilities and begin depending on each other.

Refactor into smaller, focused services.

---

## Trying to Fix by Changing Lifetime

Changing

```
Scoped

↓

Singleton
```

does **not** remove a circular dependency.

The cycle still exists.

---

## Injecting IServiceProvider Everywhere

Using `IServiceProvider` to resolve every dependency manually hides the real problem and makes the code harder to understand.

---

# Best Practices

- Keep services focused on a single responsibility.
- Design dependencies as one-way relationships.
- Prefer constructor injection with clear dependencies.
- Extract shared logic into dedicated services.
- Consider events or messaging for cross-service communication.
- Avoid the Service Locator pattern unless there is a compelling reason.

---

# Dependency Graph

Good

```
Controller
      │
      ▼
OrderService
      │
      ▼
Repository
      │
      ▼
DbContext
```

No cycles.

---

Bad

```
OrderService
      │
      ▼
PaymentService
      ▲
      │
──────┘
```

Circular dependency.

---

# Easy Way to Remember

Think of **two people trying to enter a room**.

Person A says:

> "You go first."

Person B says:

> "No, you go first."

```
A

↓

B

↓

A

↓

B
```

Nobody moves.

That's exactly what happens in a circular dependency.

---

# Frequently Asked Interview Questions

### What is a circular dependency?

A circular dependency occurs when two or more services depend on each other, forming a cycle that prevents the DI container from creating the objects.

---

### What happens if ASP.NET Core detects one?

The built-in DI container throws an `InvalidOperationException` indicating that a circular dependency was detected.

---

### How do you fix circular dependencies?

The preferred approach is to refactor the design by extracting shared responsibilities into another service or by using event-driven communication. Lazy resolution or `IServiceProvider` can break the cycle but should be used cautiously.

---

### Does using interfaces automatically solve circular dependencies?

No.

If the dependency graph remains circular, changing classes to interfaces does not solve the underlying design problem.

---

# Interview Answer (2-Minute Version)

**What is a circular dependency in ASP.NET Core, and how do you resolve it?**

A circular dependency occurs when two or more services depend on each other directly or indirectly, creating a dependency cycle. For example, if `OrderService` depends on `PaymentService` and `PaymentService` depends on `OrderService`, the ASP.NET Core Dependency Injection container cannot determine which service to create first and throws an `InvalidOperationException`. The best solution is to redesign the application by separating shared responsibilities into another service or using event-driven communication so that dependencies flow in one direction. Techniques such as `Lazy<T>` or `IServiceProvider` can break constructor cycles but should generally be considered workarounds rather than primary design solutions.