# 48. Factory Pattern in ASP.NET Core

The **Factory Pattern** is a **Creational Design Pattern** that creates objects **without exposing the object creation logic to the client**.

Instead of using the `new` keyword everywhere, you ask a **Factory** to create the object for you.

The Factory decides:

- Which object to create
- How to create it
- When to create it

This makes the application:

- Loosely coupled
- Easier to maintain
- Easier to test
- Easier to extend

> **Interview Tip:** The Factory Pattern is commonly used in ASP.NET Core for creating services whose implementation depends on runtime conditions, configuration, or user input.

---

# What is the Factory Pattern?

## Definition

The Factory Pattern encapsulates object creation inside a dedicated class called a **Factory**.

Instead of:

```csharp
var payment = new CreditCardPayment();
```

You write:

```csharp
var payment = factory.Create();
```

The client doesn't know how the object is created.

---

# Layman Example: Coffee Shop

Imagine you enter a coffee shop.

Without Factory

```
Customer

↓

Buy Coffee Beans

↓

Buy Milk

↓

Make Coffee

↓

Drink
```

You do everything.

---

With Factory

```
Customer

↓

Coffee Shop

↓

Order Latte

↓

Coffee Shop Makes It

↓

Serve Coffee
```

The coffee shop is the **Factory**.

---

# Another Example: Car Factory

Without Factory

```
Customer

↓

Build Engine

↓

Install Wheels

↓

Paint Car

↓

Drive
```

---

With Factory

```
Customer

↓

Car Factory

↓

Choose SUV

↓

Factory Builds SUV

↓

Drive
```

The customer only requests the type of car.

---

# Why Do We Need a Factory?

Suppose we have multiple payment methods:

- Credit Card
- UPI
- PayPal
- Net Banking

Without Factory

```csharp
if(type == "CreditCard")
    return new CreditCardPayment();

if(type == "UPI")
    return new UpiPayment();

if(type == "PayPal")
    return new PaypalPayment();
```

Everywhere in the application, this logic gets repeated.

---

With Factory

```csharp
var payment = factory.Create(type);
```

The factory contains all the creation logic in one place.

---

# Real-World ASP.NET Core Example

Imagine an e-commerce application.

```
Checkout

↓

Payment Factory

↓

Credit Card
```

or

```
Checkout

↓

Payment Factory

↓

UPI
```

or

```
Checkout

↓

Payment Factory

↓

PayPal
```

The controller doesn't need to know which implementation is being created.

---

# Step 1 - Create Interface

```csharp
public interface IPaymentService
{
    void Pay();
}
```

---

# Step 2 - Implementations

Credit Card

```csharp
public class CreditCardPayment : IPaymentService
{
    public void Pay()
    {
        Console.WriteLine("Paid using Credit Card");
    }
}
```

UPI

```csharp
public class UpiPayment : IPaymentService
{
    public void Pay()
    {
        Console.WriteLine("Paid using UPI");
    }
}
```

PayPal

```csharp
public class PaypalPayment : IPaymentService
{
    public void Pay()
    {
        Console.WriteLine("Paid using PayPal");
    }
}
```

---

# Step 3 - Factory

```csharp
public interface IPaymentFactory
{
    IPaymentService Create(string paymentType);
}
```

---

Implementation

```csharp
public class PaymentFactory : IPaymentFactory
{
    public IPaymentService Create(string paymentType)
    {
        return paymentType switch
        {
            "CreditCard" => new CreditCardPayment(),
            "UPI" => new UpiPayment(),
            "PayPal" => new PaypalPayment(),
            _ => throw new ArgumentException("Invalid Payment Type")
        };
    }
}
```

---

# Step 4 - Register Factory

```csharp
builder.Services.AddSingleton<IPaymentFactory, PaymentFactory>();
```

---

# Step 5 - Use Factory

```csharp
public class CheckoutController : ControllerBase
{
    private readonly IPaymentFactory _factory;

    public CheckoutController(IPaymentFactory factory)
    {
        _factory = factory;
    }

    [HttpPost]
    public IActionResult Pay(string type)
    {
        var payment = _factory.Create(type);

        payment.Pay();

        return Ok();
    }
}
```

---

# How It Works

```
HTTP Request
       │
       ▼
Controller
       │
       ▼
Factory
       │
       ▼
Creates Correct Payment Service
       │
       ▼
Returns Service
       │
       ▼
Execute Payment
```

---

# Factory with Dependency Injection

Instead of using `new`, let the DI container create the implementations.

Registration

```csharp
builder.Services.AddTransient<CreditCardPayment>();
builder.Services.AddTransient<UpiPayment>();
builder.Services.AddTransient<PaypalPayment>();

builder.Services.AddSingleton<IPaymentFactory, PaymentFactory>();
```

Factory

```csharp
public class PaymentFactory : IPaymentFactory
{
    private readonly IServiceProvider _serviceProvider;

    public PaymentFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public IPaymentService Create(string paymentType)
    {
        return paymentType switch
        {
            "CreditCard" => _serviceProvider.GetRequiredService<CreditCardPayment>(),
            "UPI" => _serviceProvider.GetRequiredService<UpiPayment>(),
            "PayPal" => _serviceProvider.GetRequiredService<PaypalPayment>(),
            _ => throw new ArgumentException("Invalid Payment Type")
        };
    }
}
```

This allows the DI container to resolve any dependencies required by the payment services.

---

# Real-World Use Cases

Factory Pattern is commonly used for:

- Payment gateways
- Notification providers (Email/SMS/Push)
- Report generators (PDF/Excel/CSV)
- Cloud storage providers (Azure Blob/AWS S3/Google Cloud Storage)
- Database provider selection
- Authentication provider selection

---

# Advantages

- Centralizes object creation.
- Reduces tight coupling.
- Simplifies maintenance.
- Makes adding new implementations easier.
- Works well with Dependency Injection.

---

# Disadvantages

- Adds extra classes and interfaces.
- Can become complex if overused.
- A large `switch` statement may need refactoring as implementations grow.

---

# Factory vs Dependency Injection

Many developers confuse these.

| Dependency Injection | Factory Pattern |
|----------------------|-----------------|
| Provides dependencies automatically | Chooses which object to create |
| Object is usually known at registration time | Object is selected at runtime |
| Lifetime managed by DI container | Factory contains selection logic |

---

# Factory vs Simple Object Creation

Without Factory

```
Controller

↓

new CreditCardPayment()
```

Controller knows the implementation.

---

With Factory

```
Controller

↓

PaymentFactory

↓

Correct Implementation
```

Controller only knows the interface.

---

# Common Mistakes

## Using `new` Inside Controllers

Bad

```csharp
var payment = new CreditCardPayment();
```

This tightly couples the controller to the implementation.

---

## Putting Business Logic in Factory

Factories should create objects.

They should **not** execute business workflows.

---

## Creating Huge Factories

If one factory creates dozens of unrelated objects,

consider splitting it into multiple factories.

---

## Ignoring Dependency Injection

Instead of manually creating services with `new`, prefer resolving them through the DI container when they have dependencies.

---

# Best Practices

- Depend on interfaces.
- Keep factories focused only on object creation.
- Combine Factory Pattern with Dependency Injection.
- Use factories when the implementation is selected at runtime.
- Throw meaningful exceptions for unsupported types.

---

# Factory Workflow

```
Client
   │
   ▼
Factory
   │
   ▼
Choose Implementation
   │
   ▼
Create Object
   │
   ▼
Return Interface
```

---

# Easy Way to Remember

Think of **a Pizza Shop**.

```
Customer

↓

Orders Veg Pizza

↓

Pizza Shop

↓

Makes Pizza

↓

Serves Pizza
```

You don't prepare the pizza yourself.

The shop (Factory) decides how to make it.

---

# Frequently Asked Interview Questions

### What is the Factory Pattern?

The Factory Pattern is a creational design pattern that encapsulates object creation and returns the appropriate implementation without exposing creation logic to the client.

---

### Why use the Factory Pattern?

To centralize object creation, reduce coupling, and select implementations dynamically at runtime.

---

### Can the Factory Pattern be used with Dependency Injection?

Yes.

In ASP.NET Core, factories often use the built-in DI container to resolve dependencies for the objects they create.

---

### When should you use a Factory instead of normal Dependency Injection?

Use a Factory when the concrete implementation depends on runtime information, such as user input, configuration, or business rules.

---

# Interview Answer (2-Minute Version)

**What is the Factory Pattern in ASP.NET Core?**

The Factory Pattern is a creational design pattern that centralizes object creation in a dedicated factory class. Instead of directly instantiating objects with the `new` keyword, clients request an object from the factory, which decides which implementation to create based on runtime conditions. In ASP.NET Core, the Factory Pattern is commonly combined with Dependency Injection so that the factory resolves services from the DI container. This approach improves maintainability, reduces coupling, and makes it easier to extend applications by adding new implementations without changing client code.