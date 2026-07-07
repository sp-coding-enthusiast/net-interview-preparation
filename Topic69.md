# 68. Hosted Services in ASP.NET Core

A **Hosted Service** is a service that runs **in the background** along with your application.

It starts automatically when the application starts and stops automatically when the application shuts down.

Hosted Services are commonly used for:

- Background jobs
- Scheduled tasks
- Queue processing
- Email sending
- Cache refresh
- Data synchronization
- Monitoring

> **Interview Tip:** Every `BackgroundService` is a Hosted Service, but not every Hosted Service needs to inherit from `BackgroundService`.

---

# What is a Hosted Service?

## Definition

A Hosted Service is a service managed by the **Generic Host** that performs background work during the application's lifetime.

---

# Layman Example: Hotel Housekeeping

Imagine a hotel.

Guests interact with:

- Reception
- Restaurant
- Room Service

Meanwhile,

Housekeeping works in the background.

Guests don't request it every second.

It runs automatically.

Hosted Services behave exactly like housekeeping.

---

# Architecture

```
Application Starts

↓

Generic Host

↓

Hosted Service Starts

↓

Runs in Background

↓

Application Stops

↓

Hosted Service Stops
```

---

# Types of Hosted Services

## 1. IHostedService

Provides complete lifecycle control.

You implement:

```csharp
StartAsync()
```

and

```csharp
StopAsync()
```

---

## Example

```csharp
public class MyHostedService : IHostedService
{
    public Task StartAsync(CancellationToken token)
    {
        Console.WriteLine("Started");

        return Task.CompletedTask;
    }

    public Task StopAsync(CancellationToken token)
    {
        Console.WriteLine("Stopped");

        return Task.CompletedTask;
    }
}
```

---

## Register

```csharp
builder.Services.AddHostedService<MyHostedService>();
```

---

# 2. BackgroundService

Simpler approach.

```csharp
public class Worker : BackgroundService
{
    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
    }
}
```

Only implement:

```
ExecuteAsync()
```

---

# Lifecycle

```
Application Starts

↓

StartAsync()

↓

ExecuteAsync()

↓

Running

↓

StopAsync()

↓

Application Ends
```

---

# Real-World Example

```
Azure Service Bus

↓

Hosted Service

↓

Read Message

↓

Process

↓

Delete Message
```

---

# Advantages

- Automatic startup.
- Automatic shutdown.
- Full DI support.
- Logging.
- Configuration.
- Great for background processing.

---

# Hosted Service vs BackgroundService

| Hosted Service | BackgroundService |
|----------------|-------------------|
| Concept | Base class |
| Can implement `IHostedService` | Implements `IHostedService` internally |
| More control | Simpler implementation |

---

# Interview Answer

A Hosted Service is a background service managed by the Generic Host. It starts when the application starts and stops gracefully during shutdown. Hosted Services are commonly used for scheduled jobs, queue processing, email sending, and other background tasks. They can be implemented either by using the `IHostedService` interface or, more commonly, by inheriting from `BackgroundService`.

---

# 69. Graceful Shutdown

Graceful Shutdown means allowing an application to **finish ongoing work safely before stopping**.

Instead of terminating immediately,

the application:

- Stops accepting new work.
- Finishes current work.
- Releases resources.
- Exits cleanly.

---

# Layman Example: Restaurant Closing

Imagine a restaurant closes at 10 PM.

The manager doesn't:

- Throw customers out.
- Turn off lights immediately.

Instead,

- No new customers are admitted.
- Existing customers finish eating.
- Bills are paid.
- Staff clean up.
- Restaurant closes.

ASP.NET Core behaves the same way.

---

# Why is Graceful Shutdown Important?

Suppose a payment is processing.

Suddenly,

the application crashes.

Possible problems:

- Partial payment
- Corrupted data
- Lost messages
- Database inconsistencies

Graceful shutdown prevents these issues.

---

# Shutdown Flow

```
Shutdown Signal

↓

Stop Accepting Requests

↓

Cancel Background Services

↓

Finish Current Work

↓

Dispose Resources

↓

Application Stops
```

---

# CancellationToken

Hosted Services receive

```csharp
CancellationToken stoppingToken
```

Example

```csharp
while(!stoppingToken.IsCancellationRequested)
{
    await Task.Delay(
        1000,
        stoppingToken);
}
```

When shutdown begins,

the token is cancelled.

---

# What Happens Internally?

```
Ctrl + C

↓

SIGTERM

↓

Generic Host

↓

Cancellation Requested

↓

Stop Hosted Services

↓

Dispose Services

↓

Exit
```

---

# Real-World Example

Order Processing

```
Receive Order

↓

Payment Running

↓

Shutdown Requested

↓

Complete Payment

↓

Save Order

↓

Shutdown
```

---

# Best Practices

- Always respect `CancellationToken`.
- Dispose unmanaged resources.
- Complete current transactions.
- Avoid abrupt termination.
- Log shutdown events.

---

# Common Mistakes

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

# Interview Answer

Graceful Shutdown is the process of stopping an application safely. When ASP.NET Core receives a shutdown signal, it stops accepting new work, signals hosted services using a `CancellationToken`, allows ongoing operations to finish, releases resources, and then exits cleanly. This helps prevent data loss and incomplete transactions.

---

# 70. Health Checks in ASP.NET Core

Health Checks allow applications to expose an endpoint that reports whether the application is **healthy**.

Monitoring systems use this endpoint to determine whether the application is functioning correctly.

Typical URL

```
/health
```

---

# Layman Example: Hospital Checkup

Imagine a doctor checking:

- Heart
- Blood Pressure
- Temperature

If everything is normal,

the patient is healthy.

Health Checks do the same for applications.

---

# Why Do We Need Health Checks?

Suppose your API depends on:

- SQL Server
- Redis
- Azure Service Bus

If SQL Server is down,

your application may still be running,

but it isn't healthy.

Health Checks detect this.

---

# Architecture

```
Monitoring Tool

↓

/health

↓

ASP.NET Core

↓

Check Database

↓

Check Cache

↓

Check Queue

↓

Healthy / Unhealthy
```

---

# Enable Health Checks

Register

```csharp
builder.Services.AddHealthChecks();
```

Map endpoint

```csharp
app.MapHealthChecks("/health");
```

---

# Access

```
GET

/health
```

Example response

```
Healthy
```

or

```
Unhealthy
```

---

# Custom Health Check

```csharp
public class DatabaseHealthCheck
    : IHealthCheck
{
    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken token)
    {
        return Task.FromResult(
            HealthCheckResult.Healthy());
    }
}
```

Register

```csharp
builder.Services
    .AddHealthChecks()
    .AddCheck<DatabaseHealthCheck>("Database");
```

---

# Types of Health Checks

## Liveness Probe

Checks:

"Is the application running?"

---

## Readiness Probe

Checks:

"Is the application ready to serve requests?"

Useful for Kubernetes deployments.

---

## Dependency Health

Checks:

- SQL Server
- Redis
- Azure Storage
- Service Bus
- External APIs

---

# Real-World Example

```
Application

↓

Health Endpoint

↓

SQL Server ✓

↓

Redis ✓

↓

Azure Service Bus ✓

↓

Healthy
```

---

# Kubernetes Example

```
Kubernetes

↓

/health

↓

Healthy?

↓

Yes

↓

Keep Running

↓

No

↓

Restart Container
```

---

# Advantages

- Early problem detection.
- Supports cloud monitoring.
- Automatic container restarts.
- Easy integration with load balancers.
- Improves reliability.

---

# Common Mistakes

- Checking only whether the application is running.
- Ignoring critical dependencies.
- Performing slow health checks.
- Returning sensitive information.

---

# Best Practices

- Keep health checks fast.
- Check important dependencies.
- Separate liveness and readiness checks.
- Don't expose internal details publicly.
- Monitor health continuously.

---

# Frequently Asked Interview Questions

### What is a Hosted Service?

A Hosted Service is a background service managed by the Generic Host that starts automatically when the application starts and stops gracefully when the application shuts down.

---

### What is Graceful Shutdown?

Graceful Shutdown allows an application to stop safely by completing ongoing work, releasing resources, and respecting cancellation requests before exiting.

---

### What are Health Checks?

Health Checks expose endpoints (such as `/health`) that monitoring systems use to determine whether an application and its critical dependencies are healthy.

---

### What is the difference between Liveness and Readiness probes?

| Liveness | Readiness |
|----------|-----------|
| Checks if the application is alive | Checks if the application is ready to serve traffic |
| Failure usually causes a restart | Failure usually removes the instance from receiving traffic until it recovers |

---

# Interview Answer (2-Minute Version)

**Hosted Services:** Hosted Services are background services managed by the Generic Host. They start automatically when the application starts and stop gracefully during shutdown. They are commonly implemented using `BackgroundService` and are used for scheduled jobs, queue processing, and background tasks.

**Graceful Shutdown:** Graceful Shutdown ensures that when an application stops, it first stops accepting new work, signals running services through a `CancellationToken`, allows current operations to complete, releases resources, and then exits cleanly, preventing data loss and incomplete operations.

**Health Checks:** Health Checks provide endpoints, typically `/health`, that report the health of an application and its dependencies. They are widely used by monitoring systems, load balancers, and orchestration platforms like Kubernetes to determine whether an application should continue receiving traffic or be restarted.