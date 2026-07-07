# 66. Worker Service in ASP.NET Core

A **Worker Service** is a .NET application designed to run **background tasks continuously** without serving web pages or APIs.

Unlike an ASP.NET Core Web API, a Worker Service does **not** wait for HTTP requests. Instead, it starts when the application starts and keeps running until it is stopped.

Typical use cases include:

- Processing messages from a queue
- Sending emails
- Scheduled jobs
- Data synchronization
- File processing
- Cache cleanup
- Background monitoring

> **Interview Tip:** A Worker Service uses the **Generic Host**, just like an ASP.NET Core Web API. The main difference is that a Worker Service **does not host Kestrel** because it doesn't serve HTTP requests.

---

# What is a Worker Service?

## Definition

A Worker Service is a **long-running background application** built on the .NET Generic Host.

It continuously performs work in the background.

---

# Layman Example: Night Security Guard

Imagine a shopping mall.

During the day:

- Customers visit.
- Cashiers work.
- Shops are open.

At night:

The mall is closed,

but the **security guard keeps working**.

He:

- Patrols the building
- Checks doors
- Watches CCTV
- Reports issues

Nobody asks him to start working.

He works continuously.

A Worker Service behaves the same way.

---

# Another Example: Washing Machine

You start a washing machine.

After pressing **Start**,

it keeps running until the cycle finishes.

You don't need to press a button every minute.

A Worker Service also runs continuously after it starts.

---

# Why Do We Need Worker Services?

Suppose your application must:

Every minute

↓

Check new orders

↓

Send confirmation emails

Should users call an API every minute?

No.

Instead,

a Worker Service performs the task automatically.

---

# Architecture

```
Application Starts

↓

Generic Host

↓

Worker Service

↓

Background Task

↓

Repeat
```

---

# Creating a Worker Service

Create a project

```bash
dotnet new worker
```

This creates a Worker Service template.

---

# Project Structure

```
WorkerService

│

├── Program.cs

├── Worker.cs

├── appsettings.json

└── Properties
```

---

# Program.cs

```csharp
var builder = Host.CreateApplicationBuilder(args);

builder.Services.AddHostedService<Worker>();

var host = builder.Build();

host.Run();
```

Explanation

- Creates the Generic Host.
- Registers the background worker.
- Starts the application.

---

# Worker Class

```csharp
public class Worker : BackgroundService
{
    private readonly ILogger<Worker> _logger;

    public Worker(ILogger<Worker> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation(
                "Worker running at: {Time}",
                DateTime.Now);

            await Task.Delay(5000, stoppingToken);
        }
    }
}
```

This worker:

- Runs forever
- Logs the current time every 5 seconds
- Stops gracefully when the application shuts down

---

# What is BackgroundService?

`BackgroundService` is a base class provided by .NET.

Instead of implementing all hosting logic yourself,

you simply override:

```csharp
ExecuteAsync()
```

This method contains your background work.

---

# ExecuteAsync()

This is the heart of every Worker Service.

```csharp
protected override async Task ExecuteAsync(
    CancellationToken stoppingToken)
{
}
```

The method starts when the application starts.

It continues running until:

- Application stops
- Cancellation is requested

---

# CancellationToken

Notice

```csharp
while(!stoppingToken.IsCancellationRequested)
```

When the application shuts down,

the Generic Host signals cancellation.

Your worker should stop gracefully.

---

# Dependency Injection

Worker Services support full Dependency Injection.

Example

```csharp
public class Worker : BackgroundService
{
    private readonly IEmailService _emailService;

    public Worker(IEmailService emailService)
    {
        _emailService = emailService;
    }
}
```

Register services normally.

```csharp
builder.Services.AddScoped<IEmailService,
    EmailService>();
```

---

# Logging

Worker Services use the same logging system.

```csharp
_logger.LogInformation(
    "Email processed");
```

---

# Configuration

Read configuration normally.

```csharp
builder.Configuration["ApiKey"];
```

or

```csharp
IOptions<MySettings>
```

---

# Real-World Example

E-commerce System

Customer places order

↓

Order saved

↓

Message sent to Queue

↓

Worker Service reads Queue

↓

Sends Email

↓

Updates Inventory

↓

Generates Invoice

No HTTP request is needed.

---

# Worker Service vs Web API

| Worker Service | Web API |
|---------------|---------|
| Runs continuously | Runs on incoming HTTP requests |
| No Kestrel required | Uses Kestrel to receive requests |
| Executes background jobs | Serves clients |
| Usually no controllers | Uses controllers/endpoints |

---

# Worker Service vs Console Application

| Console App | Worker Service |
|-------------|----------------|
| Runs once and exits | Runs continuously |
| Manual lifecycle | Managed by Generic Host |
| Limited infrastructure | Built-in DI, Logging, Configuration |

---

# Worker Service Lifecycle

```
Application Starts

↓

Generic Host

↓

Create Worker

↓

ExecuteAsync()

↓

Loop

↓

Cancellation Requested

↓

Stop Gracefully
```

---

# Scheduled Tasks Example

Run every hour

```csharp
while (!stoppingToken.IsCancellationRequested)
{
    await ProcessOrders();

    await Task.Delay(
        TimeSpan.FromHours(1),
        stoppingToken);
}
```

---

# Processing Queue Messages

```
Queue

↓

Worker Service

↓

Read Message

↓

Process

↓

Delete Message

↓

Repeat
```

Common with:

- Azure Service Bus
- RabbitMQ
- Kafka
- Amazon SQS

---

# Advantages

- Ideal for long-running tasks.
- Supports Dependency Injection.
- Built-in logging.
- Built-in configuration.
- Graceful shutdown.
- Easy to deploy as Windows Services, Linux services, or containers.

---

# Disadvantages

- Not intended to serve HTTP requests.
- Poorly designed loops can consume excessive CPU or memory.
- Long-running tasks require careful error handling and monitoring.

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

## Blocking Threads

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

`Task.Delay` does not block the thread.

---

## Swallowing Exceptions

Unhandled exceptions can stop the worker.

Catch, log, and handle expected failures appropriately.

---

## Creating Infinite Tight Loops

Bad

```csharp
while(true)
{
}
```

This consumes CPU continuously.

Always include an appropriate delay or wait for work.

---

# Best Practices

- Inherit from `BackgroundService`.
- Respect the `CancellationToken`.
- Use asynchronous programming (`async`/`await`).
- Use Dependency Injection.
- Log important events and failures.
- Keep background tasks resilient and idempotent when possible.
- Use queues for scalable background processing.

---

# Visual Representation

```
Application Starts

↓

Generic Host

↓

Worker Service

↓

ExecuteAsync()

↓

Background Task

↓

Repeat Until Shutdown
```

---

# Easy Way to Remember

Imagine **a security guard**.

The office closes,

but the guard keeps working.

He doesn't wait for customers.

He continuously monitors the building.

A Worker Service continuously performs background work without waiting for HTTP requests.

---

# Frequently Asked Interview Questions

### What is a Worker Service?

A Worker Service is a .NET application that runs long-running background tasks using the Generic Host.

---

### Does a Worker Service use Kestrel?

No.

A Worker Service does not host a web server because it doesn't process HTTP requests.

---

### Which class is commonly inherited?

Most Worker Services inherit from `BackgroundService`.

---

### What is `ExecuteAsync()`?

`ExecuteAsync()` is the main method where the background work is implemented. It starts when the application starts and runs until cancellation is requested.

---

### Can Worker Services use Dependency Injection?

Yes.

They support the same Dependency Injection, Logging, and Configuration features as ASP.NET Core Web APIs.

---

# Interview Answer (2-Minute Version)

**What is a Worker Service in .NET?**

A Worker Service is a long-running background application built on the .NET Generic Host. Unlike an ASP.NET Core Web API, it does not host Kestrel or process HTTP requests. Instead, it continuously performs background work such as processing queue messages, sending emails, synchronizing data, or executing scheduled tasks. Most Worker Services inherit from `BackgroundService` and implement the `ExecuteAsync()` method, which runs until the application is stopped. Worker Services fully support Dependency Injection, Logging, Configuration, and graceful shutdown through the Generic Host, making them the preferred choice for background processing in modern .NET applications.