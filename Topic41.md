# 40. How Do You Debug Middleware in ASP.NET Core?

Debugging middleware means identifying where a request is failing, stopping, slowing down, or behaving unexpectedly within the ASP.NET Core request pipeline.

Since every request passes through multiple middleware components, debugging middleware is a critical skill for ASP.NET Core developers, lead engineers, and solution architects.

---

# Why Debug Middleware?

Common problems include:

- Request never reaches the controller
- Authentication fails
- Authorization returns 403
- CORS errors
- Static files not loading
- Middleware order issues
- Performance bottlenecks
- Unhandled exceptions
- Infinite redirects

---

# Layman Example: Water Pipeline

Imagine water flowing through pipes.

```
Tank

↓

Pipe A

↓

Pipe B

↓

Pipe C

↓

House
```

Suddenly no water reaches the house.

You inspect:

```
Tank ✓

Pipe A ✓

Pipe B ✗

Pipe C ?
```

You found the issue.

Middleware debugging works exactly the same way.

---

# Another Example: Parcel Delivery

```
Warehouse

↓

Sorting Center

↓

Truck

↓

Customer
```

Package missing?

Check every step.

Middleware debugging means checking each stage of the request pipeline.

---

# Debugging Strategy

Always answer:

1. Did the request arrive?
2. Which middleware executed?
3. Which middleware stopped execution?
4. Did the controller execute?
5. Was a response generated?
6. Was an exception thrown?

---

# Technique 1: Add Logging

The easiest and most common approach.

---

## Example

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Middleware Start");

    await next();

    Console.WriteLine("Middleware End");
});
```

---

Request

```
GET /products
```

Output

```
Middleware Start

Middleware End
```

Middleware executed successfully.

---

# Debugging Multiple Middleware

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Middleware A");

    await next();
});

app.Use(async (context, next) =>
{
    Console.WriteLine("Middleware B");

    await next();
});

app.Run(async context =>
{
    Console.WriteLine("Controller");
});
```

Output

```
Middleware A

Middleware B

Controller
```

Now you know exactly how far the request traveled.

---

# Technique 2: Use ILogger

Preferred in production.

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<LoggingMiddleware> _logger;

    public LoggingMiddleware(
        RequestDelegate next,
        ILogger<LoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation(
            "Request: {Path}",
            context.Request.Path);

        await _next(context);

        _logger.LogInformation(
            "Response Status: {StatusCode}",
            context.Response.StatusCode);
    }
}
```

---

Logs

```
Request: /products

Response Status: 200
```

---

# Technique 3: Visual Studio Breakpoints

Place a breakpoint.

```csharp
public async Task InvokeAsync(HttpContext context)
{
    await _next(context);
}
```

---

Execution

```
Request

↓

Breakpoint Hit

↓

Inspect Variables

↓

Continue
```

Useful for:

- Request Headers
- Query Strings
- Claims
- User Identity
- Response Status Codes

---

# Technique 4: Check Middleware Order

One of the most common issues.

Bad

```csharp
app.UseAuthorization();

app.UseAuthentication();
```

Result

```
403 Forbidden
```

---

Correct

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

---

Debugging Tip

Verify middleware order first when behavior looks incorrect.

---

# Technique 5: Log Before and After next()

This helps identify where execution stops.

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Before");

    await next();

    Console.WriteLine("After");
});
```

---

Output

```
Before
```

but not

```
After
```

Possible causes:

- Exception thrown
- Short-circuit occurred
- Request never returned

---

# Technique 6: Detect Short-Circuiting

Example

```csharp
app.Use(async context =>
{
    await context.Response.WriteAsync("Blocked");
});
```

No `next()`.

Pipeline ends.

---

Symptoms

```
Controller Never Executes
```

---

Debugging

Check whether middleware calls

```csharp
await next();
```

when expected.

---

# Technique 7: Use Developer Exception Page

Development only.

```csharp
app.UseDeveloperExceptionPage();
```

Provides detailed error information in the browser.

---

Example

```
NullReferenceException

File: ProductController.cs

Line: 35
```

Much easier than a generic 500 error.

---

# Technique 8: Use Exception Middleware

```csharp
app.UseExceptionHandler("/error");
```

or custom middleware

```csharp
try
{
    await next();
}
catch(Exception ex)
{
    logger.LogError(ex.Message);
}
```

Captures unexpected failures.

---

# Technique 9: Inspect HttpContext

Useful properties:

```csharp
context.Request.Path
```

```csharp
context.Request.Method
```

```csharp
context.User.Identity.Name
```

```csharp
context.Response.StatusCode
```

---

Example

```csharp
_logger.LogInformation(
    "Path: {Path}",
    context.Request.Path);
```

---

# Technique 10: Use ASP.NET Core Request Logging

With tools such as structured logging frameworks.

Example

```csharp
app.UseHttpLogging();
```

---

Logs

```
Request:
GET /products

Status:
200
```

Useful for troubleshooting request flow.

---

# Technique 11: Browser Developer Tools

Open

```
F12

↓

Network Tab
```

Inspect:

- Status Codes
- Headers
- CORS Errors
- Redirects
- Response Time

---

Example

```
401 Unauthorized
```

Authentication issue.

---

Example

```
403 Forbidden
```

Authorization issue.

---

Example

```
CORS Error
```

CORS middleware configuration issue.

---

# Technique 12: Postman

Useful for API debugging.

Check:

- Headers
- Tokens
- Cookies
- Response Body
- Status Code

---

Example

```
Authorization:
Bearer Token
```

Verify the token is actually being sent.

---

# Technique 13: Enable Detailed Authentication Logs

Configuration

```json
{
  "Logging": {
    "LogLevel": {
      "Microsoft.AspNetCore.Authentication": "Debug"
    }
  }
}
```

---

Logs

```
Token Validation Failed
```

Helpful when debugging JWT authentication.

---

# Technique 14: Trace the Entire Pipeline

Example

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("1");

    await next();

    Console.WriteLine("1 End");
});

app.Use(async (context, next) =>
{
    Console.WriteLine("2");

    await next();

    Console.WriteLine("2 End");
});

app.Run(async context =>
{
    Console.WriteLine("3");
});
```

Output

```
1

2

3

2 End

1 End
```

Shows exact execution order.

---

# Common Middleware Issues

## Request Never Reaches Controller

Possible causes

- Short-circuiting
- Routing issue
- Authorization failure
- Authentication failure

---

## Controller Executes but No Response

Possible causes

- Exception
- Response overwritten
- Incorrect middleware order

---

## CORS Error

Possible causes

```csharp
app.UseCors();
```

placed incorrectly.

---

## Infinite Redirect Loop

Possible causes

```csharp
UseHttpsRedirection()
```

combined with incorrect proxy configuration.

---

# Real-World Debugging Example

Problem

```
GET /admin

↓

403 Forbidden
```

Investigation

```
Routing ✓

Authentication ✗
```

Cause

```csharp
app.UseAuthorization();

app.UseAuthentication();
```

Wrong order.

---

Fix

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

Problem solved.

---

# Best Practices

- Use structured logging (`ILogger`).
- Add logging before and after `next()`.
- Use breakpoints during development.
- Verify middleware order before deep investigation.
- Enable the Developer Exception Page in development.
- Monitor request and response status codes.
- Log exceptions centrally.
- Use browser tools and Postman to inspect requests.

---

# Debugging Flow

```
Request
   │
   ▼
Middleware 1
   │
   ▼
Middleware 2
   │
   ▼
Middleware 3
   │
   ▼
Controller
   │
   ▼
Response
```

Debug each step:

```
Reached Middleware 1?
        │
        ▼
Reached Middleware 2?
        │
        ▼
Reached Middleware 3?
        │
        ▼
Reached Controller?
        │
        ▼
Response Generated?
```

---

# Frequently Asked Interview Questions

### How do you debug middleware?

I typically use structured logging (`ILogger`), breakpoints, browser developer tools, Postman, and ASP.NET Core diagnostic middleware. I also verify middleware ordering and add logs before and after `next()` to determine where the request stops.

---

### How do you identify which middleware is stopping the pipeline?

Add logging before and after `await next()`.

If the log after `next()` never appears, the request may have been short-circuited or an exception may have occurred.

---

### What is the first thing to check when middleware behaves incorrectly?

Check the middleware registration order.

Many issues are caused by incorrect ordering.

---

### Which tools are commonly used?

- Visual Studio Debugger
- ILogger
- ASP.NET Core Http Logging
- Browser Developer Tools
- Postman
- Application Insights
- OpenTelemetry
- Serilog

---

# Easy Way to Remember

Use the acronym:

## T.R.A.C.E

**T** → Trace request flow  
**R** → Review middleware order  
**A** → Add logging (`ILogger`)  
**C** → Check `HttpContext` and status codes  
**E** → Examine exceptions

---

# Interview Answer (2-Minute Version)

**How do you debug middleware in ASP.NET Core?**

To debug middleware, I first verify the middleware order because many issues stem from incorrect registration. I use `ILogger` to log requests, responses, and execution flow, often placing logs before and after `await next()` to determine where the pipeline stops. During development, I use Visual Studio breakpoints and the Developer Exception Page to inspect `HttpContext`, headers, claims, and exceptions. I also use browser developer tools, Postman, and HTTP logging middleware to analyze requests and responses. For production environments, I rely on centralized logging and observability platforms such as Application Insights, OpenTelemetry, and Serilog to trace requests across the middleware pipeline.