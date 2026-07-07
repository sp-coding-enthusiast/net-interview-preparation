# 62. Logging Configuration in ASP.NET Core

Logging is one of the most important features of every production application.

It helps developers answer questions like:

- What happened?
- When did it happen?
- Why did it fail?
- Which user made the request?
- How long did the request take?

ASP.NET Core provides a **built-in logging framework** that can be configured using `appsettings.json` or code.

> **Interview Tip:** Logging is not just about writing messages. A good logging strategy helps with **debugging, monitoring, troubleshooting, auditing, and production support**.

---

# What is Logging?

## Definition

Logging is the process of **recording important information** about an application's execution.

Examples include:

- Application startup
- User login
- API requests
- Database errors
- Payment failures
- Performance metrics

---

# Layman Example: CCTV Camera

Imagine a shopping mall.

There is a CCTV camera recording everything.

If something goes wrong,

security can review the footage.

Logging works the same way.

```
Application

↓

Logger

↓

Log Records

↓

Developer
```

---

# Another Example: Flight Black Box

Every airplane has a **black box**.

It records:

- Speed
- Altitude
- Engine status
- Pilot actions

If an accident happens,

investigators analyze the black box.

Application logs are the software equivalent of a black box.

---

# Why Do We Need Logging?

Suppose a customer says:

```
Payment Failed
```

Without logs,

you have no idea why.

With logs,

you might see:

```
10:30 AM

↓

Payment API Timeout
```

Problem solved much faster.

---

# Logging Architecture

```
Application

↓

ILogger

↓

Logging Provider

↓

Log Destination
```

The destination could be:

- Console
- Debug Window
- File (via third-party provider)
- Azure Monitor
- Application Insights
- Elasticsearch
- Splunk

---

# Built-in Logging

ASP.NET Core provides `ILogger<T>`.

Example

```csharp
public class ProductService
{
    private readonly ILogger<ProductService> _logger;

    public ProductService(
        ILogger<ProductService> logger)
    {
        _logger = logger;
    }

    public void Save()
    {
        _logger.LogInformation("Product saved.");
    }
}
```

The framework injects the logger automatically.

---

# Log Levels

ASP.NET Core supports several log levels.

| Level | Purpose |
|--------|---------|
| Trace | Very detailed diagnostic information |
| Debug | Information useful during development |
| Information | Normal application events |
| Warning | Unexpected situations that do not stop execution |
| Error | Operation failed |
| Critical | Application or system failure |
| None | Disables logging |

---

# Log Level Hierarchy

```
Trace

↓

Debug

↓

Information

↓

Warning

↓

Error

↓

Critical
```

If the minimum log level is **Warning**,

only these are written:

```
Warning

Error

Critical
```

---

# Configure Logging in appsettings.json

Example

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

Explanation

- Application logs are written from **Information** and above.
- ASP.NET Core framework logs are limited to **Warning** and above to reduce noise.

---

# Logging Example

```csharp
_logger.LogTrace("Trace message");

_logger.LogDebug("Debug message");

_logger.LogInformation("Product created.");

_logger.LogWarning("Low inventory.");

_logger.LogError("Database connection failed.");

_logger.LogCritical("Application shutting down.");
```

---

# Output Example

```
Information

Product created.
```

```
Warning

Low inventory.
```

```
Error

Database connection failed.
```

---

# Structured Logging

Instead of

```csharp
_logger.LogInformation(
    "Product Id = " + product.Id);
```

Use

```csharp
_logger.LogInformation(
    "Product {ProductId} created",
    product.Id);
```

Benefits

- Searchable
- Filterable
- Better analytics
- Easier integration with log management tools

---

# Exception Logging

Always include the exception object.

Good

```csharp
try
{
}
catch(Exception ex)
{
    _logger.LogError(
        ex,
        "Payment failed.");
}
```

This records:

- Message
- Stack trace
- Exception details

---

# Logging Providers

ASP.NET Core supports multiple logging providers.

Examples

- Console
- Debug
- EventSource
- EventLog (Windows)
- Azure Application Insights
- Third-party providers (Serilog, NLog, etc.)

Multiple providers can be active simultaneously.

```
Application

↓

ILogger

↓

Console

↓

Application Insights

↓

File

↓

Monitoring Dashboard
```

---

# Real-World Example

Customer places an order.

Logs

```
Information

Order Created
```

↓

```
Information

Payment Started
```

↓

```
Information

Payment Completed
```

↓

```
Information

Email Sent
```

If something fails,

the logs show exactly where.

---

# Logging Configuration by Environment

Development

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  }
}
```

Production

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  }
}
```

Development gets detailed logs.

Production reduces unnecessary log volume.

---

# Logging Flow

```
Request

↓

Controller

↓

ILogger

↓

Logging Provider

↓

Console / Monitoring Tool / Storage
```

---

# Advantages

- Simplifies debugging.
- Helps troubleshoot production issues.
- Supports monitoring and alerting.
- Improves observability.
- Useful for auditing and diagnostics.

---

# Common Mistakes

## Logging Sensitive Information

Bad

```csharp
_logger.LogInformation(
    "Password = {Password}",
    password);
```

Never log:

- Passwords
- API Keys
- JWT Tokens
- Credit card numbers
- Personal sensitive data

---

## Using Only Information Logs

Use appropriate log levels.

Errors should be logged as **Error**, not **Information**.

---

## Ignoring Exceptions

Bad

```csharp
catch(Exception)
{
}
```

Always log unexpected exceptions with context.

---

## Building Log Messages with String Concatenation

Prefer structured logging instead of concatenation.

---

# Best Practices

- Use `ILogger<T>`.
- Configure log levels by environment.
- Use structured logging.
- Log exceptions with the exception object.
- Avoid logging sensitive information.
- Choose appropriate log levels.
- Integrate with centralized monitoring tools in production.

---

# Visual Representation

```
Application

↓

ILogger

↓

Console

↓

Application Insights

↓

Monitoring
```

---

# Easy Way to Remember

Imagine **a security camera**.

If nothing is recorded,

you cannot investigate incidents.

If everything is recorded,

you can understand exactly what happened.

Logging is your application's security camera.

---

# Frequently Asked Interview Questions

### What is logging in ASP.NET Core?

Logging is the process of recording application events, errors, warnings, and diagnostic information to help monitor and troubleshoot the application.

---

### What is `ILogger<T>`?

`ILogger<T>` is the built-in logging abstraction in ASP.NET Core. It is registered with Dependency Injection and allows applications to write logs without depending on a specific logging provider.

---

### Where is logging configured?

Logging can be configured in:

- `appsettings.json`
- Environment-specific configuration files
- Code during application startup

---

### What are the available log levels?

- Trace
- Debug
- Information
- Warning
- Error
- Critical
- None

---

### Why use structured logging?

Structured logging stores named properties instead of plain text, making logs easier to search, filter, analyze, and visualize in centralized logging systems.

---

# Interview Answer (2-Minute Version)

**Explain Logging Configuration in ASP.NET Core.**

ASP.NET Core provides a built-in logging framework through `ILogger<T>`, which integrates with the Dependency Injection system. Logging behavior is typically configured in `appsettings.json`, where developers can specify minimum log levels globally or for specific categories such as `Microsoft.AspNetCore`. The framework supports multiple log levels including Trace, Debug, Information, Warning, Error, and Critical, as well as multiple logging providers such as the Console, Debug output, EventSource, Application Insights, and third-party providers like Serilog. In production, applications typically use structured logging, avoid logging sensitive information, configure different log levels for different environments, and send logs to centralized monitoring platforms for troubleshooting and observability.