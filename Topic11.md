# Exception Handling Middleware (ASP.NET Core)

Exception handling middleware is a **global safety layer** in the ASP.NET Core pipeline that catches unhandled exceptions and converts them into controlled HTTP responses.

Instead of letting the application crash or leak stack traces, it ensures:

- consistent error responses
- proper logging
- secure output (no sensitive details exposed)

---

# 1. Why Exception Handling Middleware is Needed

Without it:

```text
Controller throws exception
→ App crashes OR returns raw stack trace
→ Client sees internal details
```

With it:

```text
Controller throws exception
→ Middleware catches it
→ Logs error
→ Returns clean response (500, message)
```

---

# 2. Where It Sits in Pipeline

Exception middleware must be the **first middleware** in the pipeline.

```text
Request
  ↓
Exception Handling Middleware  ← MUST BE FIRST
  ↓
Authentication
  ↓
Authorization
  ↓
Routing
  ↓
Controller
```

---

# 3. Basic Custom Exception Middleware

```csharp
public class ExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionMiddleware> _logger;

    public ExceptionMiddleware(
        RequestDelegate next,
        ILogger<ExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception occurred");

            context.Response.StatusCode = 500;
            context.Response.ContentType = "application/json";

            var response = new
            {
                error = "Something went wrong",
                traceId = context.TraceIdentifier
            };

            await context.Response.WriteAsJsonAsync(response);
        }
    }
}
```

---

# 4. How It Works Internally

## Normal flow

```text
Middleware A
   ↓
Middleware B
   ↓
Controller executes successfully
   ↓
Response flows back
```

---

## Exception flow

```text
Middleware A
   ↓
Middleware B
   ↓
Controller throws exception ❌
   ↓
Exception bubbles up
   ↓
Exception Middleware catches it
   ↓
Response returned safely
```

---

# 5. Built-in vs Custom Middleware

## Built-in Exception Handler

```csharp
app.UseExceptionHandler("/error");
```

Used in production scenarios.

Redirects to:

```text
/error endpoint
```

---

## Custom Middleware (More Control)

Used when you want:

- custom logging
- custom response format
- correlation ID support
- integration with monitoring tools

---

# 6. Common Production Features

A real-world exception middleware usually includes:

---

## 1. Logging

```csharp
_logger.LogError(ex, "Unhandled exception");
```

---

## 2. Correlation ID

```csharp
var traceId = context.TraceIdentifier;
```

Used for tracking request across microservices.

---

## 3. Safe Error Response

```json
{
  "error": "Internal Server Error",
  "traceId": "abc-123"
}
```

---

## 4. Environment-based behavior

### Development:

```text
Full stack trace
```

### Production:

```text
Generic message only
```

---

# 7. Example with Environment Handling

```csharp
public async Task InvokeAsync(HttpContext context, IWebHostEnvironment env)
{
    try
    {
        await _next(context);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Unhandled exception");

        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/json";

        if (env.IsDevelopment())
        {
            await context.Response.WriteAsJsonAsync(new
            {
                error = ex.Message,
                stackTrace = ex.StackTrace
            });
        }
        else
        {
            await context.Response.WriteAsJsonAsync(new
            {
                error = "Internal Server Error",
                traceId = context.TraceIdentifier
            });
        }
    }
}
```

---

# 8. Registration in Pipeline

```csharp
app.UseMiddleware<ExceptionMiddleware>();
```

⚠️ Must be FIRST middleware:

```csharp
app.UseMiddleware<ExceptionMiddleware>();
app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

---

# 9. Real-Life Analogy

Think of it like a **safety net in a circus**:

```text
Performers (Controllers)
     ↓
If they fall (exception)
     ↓
Safety Net (Middleware)
     ↓
Prevents injury (system crash)
```

---

# 10. Short-Circuit Behavior

Exception middleware can also **short-circuit the pipeline**:

```text
Exception occurs
→ catch block executes
→ response returned immediately
→ remaining middleware skipped
```

---

# 11. Common Pitfalls

---

## ❌ 1. Placing middleware after UseRouting

```csharp
app.UseRouting();
app.UseMiddleware<ExceptionMiddleware>(); // WRONG
```

Result:

- some exceptions may bypass middleware

---

## ❌ 2. Not calling next()

```csharp
await _next(context);
// missing try-catch → app crash
```

---

## ❌ 3. Exposing internal errors in production

```json
{
  "stackTrace": "..."
}
```

Security risk.

---

## ❌ 4. Logging without context

Always include:

- traceId
- request path
- user info (if available)

---

# 12. Difference: Exception Middleware vs try-catch

| Approach | Scope |
|----------|------|
| try-catch | Local (method level) |
| middleware | Global (entire app) |

Middleware is preferred for consistency.

---

# 13. Built-in Alternative (Recommended in many apps)

```csharp
app.UseExceptionHandler();
app.UseHsts();
```

Benefits:

- Microsoft supported
- production hardened
- integrates with logging systems

---

# 14. Senior-Level Interview Answer

Exception handling middleware in ASP.NET Core is a global pipeline component that captures unhandled exceptions thrown anywhere in the request pipeline. It ensures the application does not crash and returns a consistent, safe HTTP response. It is typically placed at the beginning of the pipeline so it can catch exceptions from all downstream middleware and controllers. It also enables centralized logging, correlation tracking, and environment-based error responses. In production systems, it is critical for reliability, security, and observability.

---

# 15. Easy Way to Remember

```text
Try-catch = local safety
Middleware = global safety net
```

Or:

```text
Controller crashes → Middleware saves app
```