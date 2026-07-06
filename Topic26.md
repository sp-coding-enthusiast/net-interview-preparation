# 5. How to Write Custom Middleware in ASP.NET Core

One of the most common ASP.NET Core interview questions is:

> **How do you create custom middleware?**

Let's understand it in the simplest possible way.

---

# What is Custom Middleware?

## Definition

Custom middleware is **middleware that you create yourself** to perform a specific task before or after a request is processed.

ASP.NET Core provides built-in middleware like:

- Authentication
- Authorization
- Routing
- Static Files
- CORS

But sometimes your application needs functionality that isn't built in.

That's when you write **custom middleware**.

---

# Layman Example: Shopping Mall Security

Imagine entering a shopping mall.

The mall has standard checks:

- Security check
- Bag scan
- Ticket verification

Now imagine the mall introduces a new rule:

> "Every visitor must have their temperature checked."

Since no existing checkpoint does this, the mall hires a new security guard.

That new guard is your **custom middleware**.

```
Customer
    │
    ▼
Security Check
    │
    ▼
Temperature Check (Custom Middleware)
    │
    ▼
Bag Scan
    │
    ▼
Shopping Mall
```

---

# Why Do We Need Custom Middleware?

Real-world applications often require custom logic such as:

- Request logging
- Response logging
- API key validation
- Custom authentication
- Rate limiting
- Request timing
- Correlation ID generation
- Maintenance mode
- Auditing
- Custom headers

Instead of writing the same code in every controller, you write it once as middleware.

---

# Ways to Create Custom Middleware

There are two common approaches:

1. **Inline Middleware (using `app.Use()`)**
2. **Middleware Class (recommended for production)**

---

# Method 1: Inline Middleware

This is the simplest way.

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.Use(async (context, next) =>
{
    Console.WriteLine("Before Request");

    await next();

    Console.WriteLine("After Response");
});

app.Run(async context =>
{
    await context.Response.WriteAsync("Hello World");
});

app.Run();
```

---

# Execution Flow

```
Browser

↓

Before Request

↓

Hello World

↓

After Response

↓

Browser
```

---

# Output

```
Before Request

After Response
```

---

# Method 2: Create a Middleware Class (Recommended)

This is the approach used in production applications.

---

# Step 1: Create a Middleware Class

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine("Request Started");

        await _next(context);

        Console.WriteLine("Request Completed");
    }
}
```

---

# Understanding the Code

## `RequestDelegate`

```csharp
private readonly RequestDelegate _next;
```

This represents the **next middleware** in the pipeline.

Think of it as:

> "After I finish my work, who should I send the request to?"

---

## Constructor

```csharp
public LoggingMiddleware(RequestDelegate next)
{
    _next = next;
}
```

ASP.NET Core automatically passes the next middleware into your custom middleware.

---

## `InvokeAsync()`

Every custom middleware must expose one of these methods:

- `Invoke()`
- `InvokeAsync()`

`InvokeAsync()` is preferred because most middleware performs asynchronous work.

---

# Request Flow

```
Browser

↓

Logging Middleware

↓

_next()

↓

Controller

↓

Response

↓

Logging Middleware

↓

Browser
```

---

# Step 2: Register the Middleware

In `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.UseMiddleware<LoggingMiddleware>();

app.Run(async context =>
{
    await context.Response.WriteAsync("Hello World");
});

app.Run();
```

---

# Output

```
Request Started

Request Completed
```

---

# Example: Measure Request Time

Suppose you want to know how long every request takes.

```csharp
using System.Diagnostics;

public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;

    public RequestTimingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();

        await _next(context);

        stopwatch.Stop();

        Console.WriteLine($"Time Taken: {stopwatch.ElapsedMilliseconds} ms");
    }
}
```

---

# Example: Add a Custom Response Header

```csharp
public class HeaderMiddleware
{
    private readonly RequestDelegate _next;

    public HeaderMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        context.Response.Headers.Add("Application", "My ASP.NET Core App");

        await _next(context);
    }
}
```

Every response now includes:

```
Application: My ASP.NET Core App
```

---

# Example: Block Unauthorized Requests

```csharp
public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;

    public ApiKeyMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        if (!context.Request.Headers.ContainsKey("ApiKey"))
        {
            context.Response.StatusCode = 401;

            await context.Response.WriteAsync("API Key Missing");

            return;
        }

        await _next(context);
    }
}
```

---

# Execution

Request

↓

API Key Present?

```
        Yes
         │
         ▼
   Next Middleware
```

```
        No
         │
         ▼
401 Unauthorized
```

The pipeline stops if the API key is missing.

---

# Create an Extension Method (Best Practice)

Instead of repeatedly calling `UseMiddleware<T>()`, create an extension.

```csharp
public static class LoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseLoggingMiddleware(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<LoggingMiddleware>();
    }
}
```

Now register it like this:

```csharp
app.UseLoggingMiddleware();
```

This improves readability and keeps `Program.cs` clean.

---

# Complete Request Lifecycle

```
Browser
    │
    ▼
Exception Middleware
    │
    ▼
Logging Middleware
    │
    ▼
Authentication
    │
    ▼
Authorization
    │
    ▼
Routing
    │
    ▼
Controller
    ▲
    │
Routing
    ▲
Authorization
    ▲
Authentication
    ▲
Logging Middleware
    ▲
Exception Middleware
    ▲
Browser
```

---

# Best Practices

- Keep middleware focused on **one responsibility**.
- Always call `await _next(context)` unless you intentionally want to stop the pipeline.
- Prefer asynchronous operations (`InvokeAsync`).
- Avoid putting business logic in middleware.
- Register middleware in the correct order (e.g., authentication before authorization).

---

# Common Interview Follow-Up Questions

### Why do we use `RequestDelegate`?

It represents the next middleware in the request pipeline. Calling it passes control to the next component.

---

### Why use `InvokeAsync()` instead of `Invoke()`?

`InvokeAsync()` supports asynchronous operations such as database access, logging, or network calls without blocking threads.

---

### Can middleware stop the pipeline?

Yes. If it does **not** call `await _next(context)` (or returns early), the request ends there and no later middleware executes.

---

### Can middleware modify both request and response?

Yes.

- Before calling `_next(context)`, it can inspect or modify the **request**.
- After `_next(context)` returns, it can inspect or modify the **response**.

---

# Interview Answer (2-Minute Version)

**How do you write custom middleware in ASP.NET Core?**

Custom middleware is created by implementing a class that accepts a `RequestDelegate` in its constructor and exposes an `InvokeAsync(HttpContext context)` method. Inside `InvokeAsync`, you perform your custom logic, call `await _next(context)` to continue the pipeline, and optionally execute additional logic after the response returns. The middleware is registered in `Program.cs` using `app.UseMiddleware<YourMiddleware>()` or a custom extension method.

**Typical use cases** include request/response logging, API key validation, adding custom headers, measuring request execution time, rate limiting, auditing, and maintenance mode.