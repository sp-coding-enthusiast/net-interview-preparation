# Middleware Pipeline - Custom Middleware Creation (ASP.NET Core)

Middleware is one of the most important concepts in ASP.NET Core because **every HTTP request and response passes through the middleware pipeline**.

For Lead-level interviews, you should understand:

- What middleware is
- How the middleware pipeline works
- How to create custom middleware
- How middleware interacts with Dependency Injection
- Common production use cases

---

# What is Middleware?

Middleware is a software component that sits between:

```text
Client Request
      ↓
Middleware Pipeline
      ↓
Controller/API
      ↓
Middleware Pipeline
      ↓
Client Response
```

Every request travels through middleware before reaching the controller.

Every response travels back through middleware before reaching the client.

---

# Real-Life Example

Imagine airport security.

Before boarding:

```text
Passenger
    ↓
Security Check
    ↓
Passport Verification
    ↓
Boarding Gate
    ↓
Flight
```

Each checkpoint performs a task.

Similarly:

```text
Request
    ↓
Authentication Middleware
    ↓
Authorization Middleware
    ↓
Logging Middleware
    ↓
Controller
```

Each middleware performs a specific task.

---

# ASP.NET Core Request Pipeline

```text
Browser
   ↓
Kestrel
   ↓
Middleware 1
   ↓
Middleware 2
   ↓
Middleware 3
   ↓
Controller
```

Response flows back:

```text
Controller
   ↑
Middleware 3
   ↑
Middleware 2
   ↑
Middleware 1
   ↑
Browser
```

---

# Built-in Middleware Examples

ASP.NET Core provides many built-in middleware components.

```csharp
app.UseAuthentication();

app.UseAuthorization();

app.UseCors();

app.UseHttpsRedirection();

app.UseStaticFiles();

app.MapControllers();
```

Each middleware has a specific responsibility.

---

# Middleware Execution Flow

Suppose:

```csharp
app.UseMiddleware<MiddlewareA>();
app.UseMiddleware<MiddlewareB>();
app.UseMiddleware<MiddlewareC>();
```

Request Flow:

```text
A → B → C → Controller
```

Response Flow:

```text
Controller → C → B → A
```

This is called the:

```text
Onion Model
```

because middleware wraps around each other.

---

# Visual Representation

```text
Request
  ↓

Middleware A (Before)

    ↓

Middleware B (Before)

      ↓

Middleware C (Before)

        ↓

      Controller

        ↑

Middleware C (After)

      ↑

Middleware B (After)

    ↑

Middleware A (After)

  ↑

Response
```

---

# Creating Custom Middleware

A middleware is simply a class.

---

## Step 1: Create Middleware Class

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

# Understanding the Components

## RequestDelegate

```csharp
private readonly RequestDelegate _next;
```

Represents the next middleware in the pipeline.

Think:

```text
Current Middleware
        ↓
Next Middleware
```

---

## Constructor

```csharp
public LoggingMiddleware(RequestDelegate next)
{
    _next = next;
}
```

ASP.NET Core injects the next middleware automatically.

---

## InvokeAsync()

```csharp
public async Task InvokeAsync(HttpContext context)
```

This method executes for every request.

---

# What Happens Here?

```csharp
Console.WriteLine("Request Started");

await _next(context);

Console.WriteLine("Request Completed");
```

Before:

```text
Request Started
```

After:

```text
Request Completed
```

This allows middleware to execute code both before and after the request.

---

# Registering Middleware

Program.cs

```csharp
app.UseMiddleware<LoggingMiddleware>();
```

Pipeline:

```csharp
app.UseMiddleware<LoggingMiddleware>();

app.MapControllers();
```

---

# Execution Example

Request:

```text
GET /api/orders
```

Output:

```text
Request Started

Controller Executed

Request Completed
```

---

# Middleware with Dependency Injection

Middleware fully supports DI.

Example:

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
        _logger.LogInformation("Request Started");

        await _next(context);

        _logger.LogInformation("Request Completed");
    }
}
```

ASP.NET Core automatically injects ILogger.

---

# Using Scoped Services in Middleware

Important interview topic.

Suppose:

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IUserService _userService;
}
```

If IUserService is Scoped:

```csharp
services.AddScoped<IUserService, UserService>();
```

Injecting through constructor causes problems because middleware is effectively singleton-like.

---

# Correct Way

Inject scoped services into InvokeAsync:

```csharp
public async Task InvokeAsync(
    HttpContext context,
    IUserService userService)
{
    await _next(context);
}
```

ASP.NET Core resolves scoped services per request.

---

# Common Custom Middleware Examples

---

## Logging Middleware

```text
Logs request and response
```

Example:

```text
GET /api/orders
Status: 200
Duration: 50 ms
```

---

## Exception Handling Middleware

Captures unhandled exceptions.

```csharp
try
{
    await _next(context);
}
catch(Exception ex)
{
}
```

Returns:

```text
500 Internal Server Error
```

instead of application crash.

---

## Request Timing Middleware

Measures request duration.

```csharp
var watch = Stopwatch.StartNew();

await _next(context);

watch.Stop();
```

Useful for performance monitoring.

---

## Correlation ID Middleware

Adds tracking ID.

```text
Request-ID: 12345
```

Used in distributed microservices.

---

## Security Header Middleware

Adds:

```text
X-Frame-Options
X-XSS-Protection
Content-Security-Policy
```

Improves security.

---

# Short-Circuiting Middleware

Middleware can stop pipeline execution.

Example:

```csharp
public async Task InvokeAsync(HttpContext context)
{
    context.Response.StatusCode = 403;

    await context.Response.WriteAsync("Forbidden");
}
```

Notice:

```csharp
await _next(context);
```

is NOT called.

Pipeline stops immediately.

---

# Use vs Run vs Map

---

## Use()

Calls next middleware.

```csharp
app.Use(async (context, next) =>
{
    await next();
});
```

Pipeline continues.

---

## Run()

Terminal middleware.

```csharp
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello");
});
```

Pipeline ends.

---

## Map()

Branches pipeline.

```csharp
app.Map("/admin", adminApp =>
{
    adminApp.Run(async context =>
    {
        await context.Response.WriteAsync("Admin Area");
    });
});
```

---

# Common Middleware Order

Correct order:

```csharp
app.UseExceptionHandler();

app.UseHttpsRedirection();

app.UseStaticFiles();

app.UseRouting();

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

Order matters.

---

# Common Interview Questions

## What happens if middleware does not call next()?

Answer:

```text
Pipeline stops
Request never reaches controller
```

---

## Can middleware modify response?

Answer:

```text
Yes
```

Before and after controller execution.

---

## Can middleware use DI?

Answer:

```text
Yes
```

ASP.NET Core injects dependencies automatically.

---

## How do you inject Scoped service in middleware?

Answer:

Inject through:

```csharp
InvokeAsync()
```

not constructor.

---

# Real Production Example

Request:

```text
POST /api/orders
```

Pipeline:

```text
Exception Middleware
        ↓
Logging Middleware
        ↓
Authentication Middleware
        ↓
Authorization Middleware
        ↓
Controller
```

Response:

```text
Controller
        ↑
Authorization
        ↑
Authentication
        ↑
Logging
        ↑
Exception Middleware
```

---

# Senior-Level Interview Answer

Middleware is a component in the ASP.NET Core request pipeline that can inspect, modify, short-circuit, or forward HTTP requests and responses. Custom middleware is created by implementing a class that accepts RequestDelegate and exposes an InvokeAsync method. Middleware is executed in registration order for requests and reverse order for responses. It is commonly used for logging, exception handling, security, authentication, correlation IDs, and performance monitoring. Scoped services should be injected into InvokeAsync rather than the constructor to ensure proper lifetime management.