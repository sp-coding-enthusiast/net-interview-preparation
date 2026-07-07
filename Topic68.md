# 67. BackgroundService in ASP.NET Core

`BackgroundService` is a **base class** provided by .NET that makes it easy to create **long-running background tasks**.

Instead of writing all the hosting logic yourself, you inherit from `BackgroundService` and implement **one method**:

```csharp
ExecuteAsync()
```

The .NET **Generic Host** automatically starts, monitors, and stops your background service.

> **Interview Tip:** `BackgroundService` is **not a separate application type**. It is a **base class** used inside a Worker Service or an ASP.NET Core application to implement background processing.

---

# What is BackgroundService?

## Definition

`BackgroundService` is an abstract class that implements the `IHostedService` interface and provides a simple way to run long-running asynchronous tasks.

Instead of implementing lifecycle methods yourself, you only implement:

```
ExecuteAsync()
```

---

# Layman Example: Coffee Machine

Imagine an office coffee machine.

Every morning,

it starts automatically.

Throughout the day,

it keeps making coffee whenever needed.

At closing time,

it shuts down safely.

Nobody manually starts or stops it every minute.

`BackgroundService` works the same way.

---

# Another Example: Night Watchman

An office closes at 6 PM.

The night watchman:

- Starts work automatically
- Patrols continuously
- Stops when the shift ends

He doesn't wait for customers.

He works in the background.

---

# Why Do We Need BackgroundService?

Suppose your application needs to:

Every minute

↓

Read new messages

↓

Send emails

↓

Update inventory

Instead of making users call an API,

a BackgroundService runs automatically.

---

# Architecture

```
Generic Host

↓

BackgroundService

↓

ExecuteAsync()

↓

Background Work

↓

Repeat
```

---

# Relationship with IHostedService

Internally,

`BackgroundService` implements:

```csharp
IHostedService
```

So instead of writing:

```csharp
StartAsync()

StopAsync()
```

you only implement:

```csharp
ExecuteAsync()
```

The framework handles the rest.

---

# Creating a BackgroundService

```csharp
public class Worker : BackgroundService
{
    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
    }
}
```

---

# ExecuteAsync()

This is the heart of every BackgroundService.

```csharp
protected override async Task ExecuteAsync(
    CancellationToken stoppingToken)
{
    while (!stoppingToken.IsCancellationRequested)
    {
        Console.WriteLine("Running...");

        await Task.Delay(
            5000,
            stoppingToken);
    }
}
```

The method:

- Starts automatically
- Runs continuously
- Stops when cancellation is requested

---

# What is CancellationToken?

Notice

```csharp
stoppingToken
```

When the application shuts down,

the Generic Host sends a cancellation request.

```
Application

↓

Cancellation Requested

↓

BackgroundService Stops
```

This enables graceful shutdown.

---

# Registering the Service

```csharp
builder.Services.AddHostedService<Worker>();
```

Although the class inherits from `BackgroundService`,

it is registered as a **Hosted Service**.

---

# Dependency Injection

BackgroundService supports Dependency Injection.

```csharp
public class Worker : BackgroundService
{
    private readonly ILogger<Worker> _logger;

    public Worker(
        ILogger<Worker> logger)
    {
        _logger = logger;
    }
}
```

Any registered service can be injected.

---

# Logging

```csharp
_logger.LogInformation(
    "Processing Orders");
```

Logging works exactly like in Web APIs.

---

# Configuration

```csharp
builder.Configuration["ApiKey"];
```

or

```csharp
IOptions<MySettings>
```

Configuration is available through the Generic Host.

---

# Real-World Example

Order Processing

```
Customer Places Order

↓

Database

↓

BackgroundService

↓

Generate Invoice

↓

Send Email

↓

Update Inventory
```

Everything happens in the background.

---

# Scheduled Job Example

```csharp
while (!stoppingToken.IsCancellationRequested)
{
    await GenerateReport();

    await Task.Delay(
        TimeSpan.FromHours(1),
        stoppingToken);
}
```

Runs once every hour.

---

# Queue Processing Example

```
Azure Service Bus

↓

BackgroundService

↓

Read Message

↓

Process

↓

Complete Message

↓

Repeat
```

---

# BackgroundService vs IHostedService

| BackgroundService | IHostedService |
|-------------------|----------------|
| Base class | Interface |
| Override only `ExecuteAsync()` | Implement `StartAsync()` and `StopAsync()` |
| Simpler | More control over lifecycle |
| Most common choice | Used for specialized scenarios |

---

# BackgroundService vs Worker Service

Many developers confuse these.

| Worker Service | BackgroundService |
|----------------|-------------------|
| Project template/application type | Base class |
| Uses the Generic Host | Inherited inside the project |
| Can contain one or more BackgroundServices | Implements the background logic |

Think of it this way:

```
Worker Service

↓

Contains

↓

BackgroundService
```

---

# Lifecycle

```
Application Starts

↓

Generic Host

↓

Create BackgroundService

↓

ExecuteAsync()

↓

Loop

↓

Cancellation Requested

↓

Stop
```

---

# Advantages

- Simple to implement.
- Supports asynchronous programming.
- Built-in graceful shutdown.
- Full Dependency Injection support.
- Integrated logging and configuration.
- Ideal for long-running tasks.

---

# Disadvantages

- Not intended for handling HTTP requests.
- Poorly designed loops can waste CPU or memory.
- Long-running operations should include error handling and retry strategies.

---

# Common Mistakes

## Ignoring CancellationToken

Bad

```csharp
while(true)
{
}
```

Good

```csharp
while(!stoppingToken.IsCancellationRequested)
{
}
```

---

## Using Thread.Sleep()

Bad

```csharp
Thread.Sleep(5000);
```

Good

```csharp
await Task.Delay(
    5000,
    stoppingToken);
```

---

## Blocking the Thread

Always use asynchronous operations when possible.

---

## Swallowing Exceptions

Catch, log, and handle exceptions appropriately.

Unexpected exceptions can terminate the background task.

---

# Best Practices

- Inherit from `BackgroundService`.
- Register it using `AddHostedService()`.
- Respect the `CancellationToken`.
- Use `async`/`await`.
- Log significant events and errors.
- Keep work idempotent where practical.
- Avoid infinite tight loops.

---

# Internal Working

```
Application Starts

↓

Generic Host

↓

StartAsync()

↓

ExecuteAsync()

↓

Background Loop

↓

Cancellation

↓

StopAsync()
```

Although you only implement `ExecuteAsync()`, the Generic Host internally calls the `StartAsync()` and `StopAsync()` methods provided by the `BackgroundService` base class.

---

# Visual Representation

```
Generic Host

↓

BackgroundService

↓

ExecuteAsync()

↓

Background Work

↓

Graceful Shutdown
```

---

# Easy Way to Remember

Imagine **an automatic cleaning robot**.

You press **Start**.

It keeps cleaning the house on its own.

When the battery is low,

it returns to the charging station and stops safely.

`BackgroundService` behaves the same way—it starts automatically, works continuously, and shuts down gracefully.

---

# Frequently Asked Interview Questions

### What is BackgroundService?

`BackgroundService` is an abstract base class that simplifies implementing long-running background tasks in .NET applications.

---

### Which method must be implemented?

You override the `ExecuteAsync(CancellationToken stoppingToken)` method.

---

### Does BackgroundService implement IHostedService?

Yes.

`BackgroundService` implements `IHostedService` internally and provides a simpler programming model.

---

### How is a BackgroundService registered?

Using Dependency Injection:

```csharp
builder.Services.AddHostedService<Worker>();
```

---

### What is the purpose of the CancellationToken?

It allows the Generic Host to request a graceful shutdown so background tasks can stop safely without losing work.

---

# Interview Answer (2-Minute Version)

**What is BackgroundService in ASP.NET Core?**

`BackgroundService` is an abstract base class provided by .NET for implementing long-running background tasks. It builds on the `IHostedService` interface and simplifies development by requiring developers to override only the `ExecuteAsync()` method, where the background logic is implemented. The Generic Host automatically starts the service when the application starts and signals it to stop using a `CancellationToken` during application shutdown. Background services support Dependency Injection, Logging, and Configuration, making them ideal for tasks such as queue processing, scheduled jobs, email sending, and data synchronization.