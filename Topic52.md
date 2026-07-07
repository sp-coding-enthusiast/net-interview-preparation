# 52. Multiple Implementations in ASP.NET Core Dependency Injection (DI)

One of the most common interview questions is:

> **"What if multiple classes implement the same interface?"**

ASP.NET Core's Dependency Injection (DI) container allows **multiple implementations** of the same interface.

This is useful when your application supports different ways of performing the same task.

For example:

- Multiple payment gateways
- Multiple notification providers
- Multiple storage providers
- Multiple authentication providers
- Multiple report generators

> **Interview Tip:** When multiple implementations are registered, injecting a **single interface (`IService`) returns the last registered implementation**, while injecting **`IEnumerable<IService>` returns all registered implementations**.

---

# What are Multiple Implementations?

## Definition

Multiple implementations mean that **one interface has more than one concrete implementation**.

Example:

```
IPaymentService

↓

CreditCardPayment
```

```
IPaymentService

↓

UPIPayment
```

```
IPaymentService

↓

PayPalPayment
```

All implement the same interface.

---

# Layman Example: Transportation

Imagine you want to travel.

There are many options.

```
Travel

↓

Car
```

```
Travel

↓

Bus
```

```
Travel

↓

Train
```

You only know:

```
ITravel
```

The actual implementation depends on your choice.

---

# Another Example: Notification

A company can notify users using:

```
Notification

↓

Email
```

```
Notification

↓

SMS
```

```
Notification

↓

Push Notification
```

Each performs the same job differently.

---

# Step 1 - Create Interface

```csharp
public interface INotificationService
{
    void Send(string message);
}
```

---

# Step 2 - Multiple Implementations

## Email

```csharp
public class EmailService : INotificationService
{
    public void Send(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}
```

---

## SMS

```csharp
public class SmsService : INotificationService
{
    public void Send(string message)
    {
        Console.WriteLine($"SMS: {message}");
    }
}
```

---

## Push Notification

```csharp
public class PushService : INotificationService
{
    public void Send(string message)
    {
        Console.WriteLine($"Push: {message}");
    }
}
```

---

# Step 3 - Register Services

```csharp
builder.Services.AddTransient<INotificationService, EmailService>();

builder.Services.AddTransient<INotificationService, SmsService>();

builder.Services.AddTransient<INotificationService, PushService>();
```

Notice that all three registrations use the same interface.

---

# What Happens Internally?

The DI container stores all implementations.

```
INotificationService

↓

EmailService
```

```
INotificationService

↓

SmsService
```

```
INotificationService

↓

PushService
```

---

# Injecting a Single Service

```csharp
public class NotificationController
{
    public NotificationController(
        INotificationService service)
    {
    }
}
```

What will ASP.NET Core inject?

The **last registered implementation**.

```
Email

↓

SMS

↓

Push
```

Result

```
PushService
```

because it was registered last.

---

# Why?

Registrations are processed in order.

When requesting a **single service**,

ASP.NET Core resolves the last matching registration.

---

# Injecting All Implementations

Instead of:

```csharp
INotificationService
```

inject

```csharp
IEnumerable<INotificationService>
```

Example

```csharp
public class NotificationManager
{
    private readonly IEnumerable<INotificationService> _services;

    public NotificationManager(
        IEnumerable<INotificationService> services)
    {
        _services = services;
    }

    public void Notify(string message)
    {
        foreach (var service in _services)
        {
            service.Send(message);
        }
    }
}
```

---

Result

```
Email

↓

SMS

↓

Push
```

All three services execute.

---

# Internal Flow

```
Controller

↓

IEnumerable<INotificationService>

↓

EmailService
```

```
↓

SmsService
```

```
↓

PushService
```

---

# Real-World Example

Payment System

```
Checkout

↓

IPaymentService

↓

Credit Card
```

```
↓

UPI
```

```
↓

PayPal
```

You may:

- Execute one provider based on user choice.
- Execute all providers for health checks or diagnostics.

---

# Selecting One Implementation at Runtime

Suppose the user chooses a payment method.

You can combine **multiple implementations** with a **Factory Pattern**.

```csharp
public interface IPaymentFactory
{
    IPaymentService Get(string paymentType);
}
```

The factory returns the appropriate implementation based on runtime input.

---

# Using a Dictionary (One Approach)

```csharp
public class PaymentFactory
{
    private readonly IEnumerable<IPaymentService> _services;

    public PaymentFactory(
        IEnumerable<IPaymentService> services)
    {
        _services = services;
    }

    // Selection logic goes here
}
```

The factory can inspect the implementations and return the appropriate one.

---

# Real Production Scenarios

## Storage Providers

```
IStorageService

↓

Azure Blob Storage
```

```
↓

Amazon S3
```

```
↓

Local File Storage
```

---

## Payment Providers

```
IPaymentService

↓

Stripe
```

```
↓

Razorpay
```

```
↓

PayPal
```

---

## Authentication

```
IAuthProvider

↓

Google
```

```
↓

Microsoft
```

```
↓

GitHub
```

---

## Reports

```
IReportGenerator

↓

PDF
```

```
↓

Excel
```

```
↓

CSV
```

---

# Advantages

- Easy to extend.
- Supports the Open/Closed Principle (add new implementations without changing existing consumers).
- Reduces coupling.
- Enables runtime selection.
- Works naturally with Dependency Injection.

---

# Disadvantages

- Selecting the correct implementation requires additional logic (Factory, strategy, keyed services, etc.).
- A large number of implementations can become harder to manage.
- Relying on "last registration wins" can be confusing if done unintentionally.

---

# Common Mistakes

## Assuming the First Registration Is Used

Bad assumption.

```csharp
builder.Services.AddTransient<IService, A>();

builder.Services.AddTransient<IService, B>();
```

Injecting `IService` returns **B**, not A.

---

## Not Using IEnumerable

If your requirement is to execute all implementations,

inject:

```csharp
IEnumerable<IService>
```

instead of a single service.

---

## Using `if-else` Everywhere

Bad

```csharp
if(type == "Email")
```

```
if(type == "SMS")
```

```
if(type == "Push")
```

Repeated throughout the application.

A Factory or Strategy pattern keeps the selection logic centralized.

---

# Best Practices

- Depend on interfaces.
- Inject `IEnumerable<T>` when multiple implementations should be processed.
- Use a Factory or Strategy pattern for runtime selection.
- Keep each implementation focused on one responsibility.
- Register implementations intentionally and be aware that the last registration is returned for single-service resolution.

---

# Visual Comparison

## Single Injection

```
INotificationService

↓

PushService
```

(last registration)

---

## Multiple Injection

```
IEnumerable<INotificationService>

↓

Email
```

```
↓

SMS
```

```
↓

Push
```

---

# Easy Way to Remember

Imagine ordering food.

Restaurant offers:

```
Pizza
```

```
Burger
```

```
Pasta
```

If you ask for **one dish**,

you choose a single option.

If you ask for a **buffet**,

you get everything.

`INotificationService`

↓

One implementation

---

`IEnumerable<INotificationService>`

↓

All implementations

---

# Frequently Asked Interview Questions

### Can ASP.NET Core register multiple implementations for the same interface?

Yes.

The DI container supports registering multiple implementations of the same interface.

---

### What happens if I inject a single interface?

ASP.NET Core returns the **last registered implementation**.

---

### How do I get all implementations?

Inject:

```csharp
IEnumerable<IMyService>
```

The DI container returns all registered implementations in registration order.

---

### How do I select one implementation based on runtime conditions?

Use a Factory Pattern, Strategy Pattern, or another selection mechanism to choose the appropriate implementation.

---

# Interview Answer (2-Minute Version)

**How does ASP.NET Core handle multiple implementations of the same interface?**

ASP.NET Core allows multiple implementations of a single interface to be registered with the Dependency Injection container. If a class requests a single interface, such as `INotificationService`, the container resolves the **last registered implementation**. If a class requests `IEnumerable<INotificationService>`, the container provides all registered implementations in registration order. This feature is commonly used for scenarios such as payment gateways, notification providers, storage providers, and report generators. When the implementation must be selected based on runtime conditions, it is considered a best practice to use a Factory or Strategy pattern to encapsulate the selection logic.