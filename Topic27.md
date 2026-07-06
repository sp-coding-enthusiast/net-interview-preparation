# 6. Exception Middleware in ASP.NET Core

One of the most important middleware in ASP.NET Core is **Exception Middleware**.

It is almost always the **first middleware** registered in the pipeline because its job is to catch any unhandled exceptions that occur later in the request pipeline.

---

# What is Exception Middleware?

## Definition

Exception Middleware catches **unhandled exceptions** that occur while processing an HTTP request and returns a user-friendly response instead of crashing the application.

Without exception middleware, users may see a generic server error or the application may terminate the request abruptly.

---

# Layman Example: Customer Service Desk

Imagine you're shopping in a mall.

```
Customer
    │
    ▼
Electronics
    │
    ▼
Billing Counter
    │
    ▼
Exit
```

Suppose the billing machine suddenly crashes.

Without customer service:

```
Customer

↓

Error

↓

Go Home
```

The customer has no idea what happened.

Now imagine there is a **Customer Service Desk** at the entrance.

Whenever any shop encounters a problem, the customer is redirected there.

```
Customer

↓

Shop

↓

Problem

↓

Customer Service

↓

"We're sorry. Please try again later."
```

The customer gets a friendly explanation instead of an unpleasant experience.

The Customer Service Desk is like **Exception Middleware**.

---

# Why Do We Need Exception Middleware?

Applications can fail due to:

- Divide by zero
- Null reference
- Database connection failure
- File not found
- Network timeout
- Invalid configuration
- Unexpected coding errors

Without exception handling, users receive a generic **500 Internal Server Error**.

Exception middleware:

- Prevents application crashes
- Logs the error
- Returns a meaningful response
- Protects sensitive information

---

# Request Pipeline

```
Browser
    │
    ▼
Exception Middleware
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
```

If an exception occurs anywhere below, it bubbles back up to the Exception Middleware.

```
Controller

❌ Exception

▲

Routing

▲

Authorization

▲

Authentication

▲

Exception Middleware

↓

Returns Friendly Error
```

---

# Without Exception Middleware

```csharp
app.MapGet("/", () =>
{
    int x = 10;
    int y = 0;

    return x / y;
});
```

Runtime

```
DivideByZeroException
```

User sees

```
500 Internal Server Error
```

No friendly message.

---

# With Exception Middleware

```csharp
app.UseExceptionHandler("/error");
```

Now

```
Controller

↓

Exception

↓

Exception Middleware

↓

/error endpoint

↓

Friendly Response
```

---

# Built-in Exception Middleware

ASP.NET Core provides built-in exception middleware.

```csharp
app.UseExceptionHandler("/error");
```

Whenever an unhandled exception occurs

Request automatically goes to

```
/error
```

---

# Example

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.UseExceptionHandler("/error");

app.Map("/error", () =>
{
    return Results.Problem("Something went wrong.");
});

app.MapGet("/", () =>
{
    throw new Exception("Database Failure");
});

app.Run();
```

---

# Output

Instead of

```
Database Failure
```

User receives

```json
{
    "title": "An error occurred.",
    "detail": "Something went wrong."
}
```

The internal exception is hidden from the client.

---

# Development vs Production

During development, detailed error pages help developers debug issues.

```csharp
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
}
```

---

# Development

```
Exception

↓

Developer Exception Page

↓

Stack Trace

↓

Line Number

↓

Source Code
```

Example:

```
DivideByZeroException

at Program.cs line 18
```

Useful only for developers.

---

# Production

```
Exception

↓

Exception Middleware

↓

Generic Error

↓

Browser
```

Example response

```json
{
    "message": "Something went wrong."
}
```

Sensitive details remain hidden.

---

# Create Custom Exception Middleware

Sometimes you want full control over error handling.

```csharp
public class ExceptionMiddleware
{
    private readonly RequestDelegate _next;

    public ExceptionMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            context.Response.StatusCode = 500;
            context.Response.ContentType = "application/json";

            await context.Response.WriteAsync(
                $"An error occurred: {ex.Message}");
        }
    }
}
```

---

# Register Custom Middleware

```csharp
app.UseMiddleware<ExceptionMiddleware>();
```

It should be registered **before** other middleware so it can catch exceptions from everything that follows.

---

# Execution Flow

```
Browser

↓

Exception Middleware

↓

Authentication

↓

Authorization

↓

Controller

↓

Exception

↓

Catch Block

↓

JSON Response

↓

Browser
```

---

# Better Production Example

Instead of exposing the exception message:

```csharp
catch (Exception ex)
{
    context.Response.StatusCode = 500;
    context.Response.ContentType = "application/json";

    await context.Response.WriteAsync(
        "An unexpected error occurred.");
}
```

Users should never see:

- SQL queries
- Passwords
- Connection strings
- File paths
- Stack traces

Those details should be written to logs, not returned to clients.

---

# Logging the Exception

A production middleware typically logs the error before returning a response.

```csharp
catch (Exception ex)
{
    logger.LogError(ex, "Unhandled exception");

    context.Response.StatusCode = 500;

    await context.Response.WriteAsync(
        "Unexpected server error.");
}
```

Flow

```
Exception

↓

Log File

↓

Return Friendly Error

↓

Browser
```

---

# Middleware Order Matters

Correct order:

```csharp
app.UseExceptionHandler("/error");

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

Incorrect order:

```csharp
app.UseAuthentication();

app.UseAuthorization();

app.UseExceptionHandler("/error");
```

In the incorrect order, exceptions thrown by authentication or authorization middleware may not be handled.

---

# Real-World Scenario

A banking application processes a transfer.

```
Transfer Money

↓

Database Update

↓

Network Failure

↓

Exception
```

Without exception middleware:

```
500 Internal Server Error
```

With exception middleware:

```json
{
    "message": "Transaction could not be completed. Please try again later."
}
```

Meanwhile:

- Error is logged
- Alert can be generated
- User receives a safe, consistent response

---

# Best Practices

- Register exception middleware **first** in the pipeline.
- Show detailed errors only in the Development environment.
- Return standardized error responses (e.g., JSON problem details for APIs).
- Log every unhandled exception.
- Never expose stack traces or sensitive information to end users.

---

# Common Interview Questions

### Why should exception middleware be registered first?

Because it needs to catch exceptions thrown by all middleware and endpoints that execute after it.

---

### Can exception middleware catch every exception?

It catches **unhandled exceptions** that occur after it in the middleware pipeline. Exceptions handled locally with `try/catch` blocks won't reach it.

---

### What is the difference between `UseDeveloperExceptionPage()` and `UseExceptionHandler()`?

| `UseDeveloperExceptionPage()` | `UseExceptionHandler()` |
|-------------------------------|-------------------------|
| Used in Development | Used in Production |
| Displays stack trace and source code | Returns a friendly error response |
| Helps debugging | Protects sensitive information |

---

# Interview Answer (2-Minute Version)

**What is Exception Middleware in ASP.NET Core?**

Exception Middleware is a middleware component that catches unhandled exceptions occurring during request processing. It prevents application crashes, logs errors, and returns a consistent, user-friendly response. In development, `UseDeveloperExceptionPage()` provides detailed debugging information. In production, `UseExceptionHandler()` or a custom exception middleware returns a safe error response without exposing internal implementation details. Exception middleware should be registered at the beginning of the middleware pipeline so it can handle exceptions from all subsequent middleware and endpoints.