Scoped Services in Middleware (Senior-Level ASP.NET Core DI Concept)

Short Answer

A middleware class is effectively created once for the application lifetime, so it behaves like a singleton.

Because of that:

❌ Do NOT inject a Scoped service through the middleware constructor.

public class LoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMyScopedService _service; // BAD
    public LoggingMiddleware(
        RequestDelegate next,
        IMyScopedService service)
    {
        _next = next;
        _service = service;
    }
}

ASP.NET Core throws:

Cannot consume scoped service from singleton.

⸻

Then How Does InvokeAsync Get Scoped Services?

ASP.NET Core provides a special mechanism.

Services passed as parameters to Invoke() or InvokeAsync() are resolved from the current HTTP request scope.

Example:

public class LoggingMiddleware
{
    private readonly RequestDelegate _next;
    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    public async Task InvokeAsync(
        HttpContext context,
        IMyScopedService scopedService)
    {
        scopedService.Log();
        await _next(context);
    }
}

This works perfectly.

⸻

Why Does It Work?

Let’s understand what happens internally.

Application Startup:

app.UseMiddleware<LoggingMiddleware>();

ASP.NET Core creates the middleware instance:

Application Starts
       |
       v
Create LoggingMiddleware
       |
       v
Store Instance

Only one middleware object exists.

⸻

Request #1

ASP.NET Core creates:

Request Scope #1

Inside that scope:

IMyScopedService Instance #1

Then:

middleware.InvokeAsync(
    context,
    scopedService1);

⸻

Request #2

ASP.NET Core creates:

Request Scope #2

Inside that scope:

IMyScopedService Instance #2

Then:

middleware.InvokeAsync(
    context,
    scopedService2);

⸻

Visual Representation

                 Middleware Instance
                 (Singleton-like)
                         |
       -------------------------------------
       |                  |               |
 Request 1           Request 2       Request 3
   Scope               Scope           Scope
     |                   |               |
 ScopedService1    ScopedService2   ScopedService3

The middleware instance remains the same.

The scoped service changes for every request.

⸻

Internal Working

Every request has:

HttpContext.RequestServices

This property points to the current request’s DI scope.

Conceptually, ASP.NET Core performs:

var scope = httpContext.RequestServices;
var scopedService =
    scope.GetRequiredService<IMyScopedService>();

It then injects that service into InvokeAsync() automatically.

You don’t need to write this code yourself.

⸻

Real Logging Middleware Example

Service Registration:

builder.Services.AddScoped<
    IRequestAuditService,
    RequestAuditService>();

Service Interface:

public interface IRequestAuditService
{
    Task SaveLog();
}

Middleware:

public class LoggingMiddleware
{
    private readonly RequestDelegate _next;
    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    public async Task InvokeAsync(
        HttpContext context,
        IRequestAuditService auditService)
    {
        await auditService.SaveLog();
        await _next(context);
    }
}

This is the recommended approach.

⸻

Why Constructor Injection Fails

Consider:

public LoggingMiddleware(
    RequestDelegate next,
    IRequestAuditService auditService)
{
}

Middleware is created once.

Scoped services are created per request.

The framework cannot determine:

Which request's auditService
should be stored in middleware?

Request #1?

Request #100?

Request #10,000?

There is no correct answer.

Therefore ASP.NET Core prevents this registration.

⸻

Advanced Architecture View

Request Processing Flow:

Kestrel
   |
   v
Create HttpContext
   |
   v
Create IServiceScope
   |
   v
HttpContext.RequestServices
   |
   v
Middleware Pipeline

When middleware executes:

InvokeAsync(
    HttpContext,
    ScopedService,
    AnotherScopedService
)

ASP.NET Core:

1. Examines the InvokeAsync parameters.
2. Identifies services that need resolution.
3. Resolves them from the current request scope.
4. Calls InvokeAsync with those instances.

Conceptually:

var service =
    context.RequestServices
           .GetRequiredService<IMyScopedService>();
await middleware.InvokeAsync(
    context,
    service);

⸻

What Happens Internally During Every Request?

Request Arrives
      |
      v
Create IServiceScope
      |
      v
Resolve Scoped Services
      |
      v
Execute Middleware
      |
      v
Dispose Scoped Services
      |
      v
Send Response

This ensures:

* Proper isolation between requests
* No shared mutable state
* Thread safety
* Automatic disposal of scoped dependencies

⸻

Interview Questions & Answers

Q1. Is Middleware a Singleton?

Answer

Middleware instances are generally created once and reused throughout the application’s lifetime, so they behave like singleton objects.

⸻

Q2. Can Middleware Constructor Inject Scoped Services?

Answer

No.

Constructor injection of scoped services into conventional middleware is not allowed because middleware has application-level lifetime while scoped services have request-level lifetime.

⸻

Q3. How Can We Use Scoped Services Inside Middleware?

Answer

Inject them into the Invoke() or InvokeAsync() method.

public async Task InvokeAsync(
    HttpContext context,
    IMyScopedService service)
{
}

ASP.NET Core resolves them from the current request scope.

⸻

Q4. Where Are Scoped Services Resolved From?

Answer

From:

HttpContext.RequestServices

which represents the current request’s dependency injection scope.

⸻

Q5. What Lifetime Does a Scoped Service Have?

Answer

One instance per HTTP request.

Request #1 -> Service Instance #1
Request #2 -> Service Instance #2
Request #3 -> Service Instance #3

The instance is disposed automatically when the request completes.

⸻

Q6. Why Is This Design Important?

Answer

Because scoped services often contain request-specific data:

* DbContext
* Unit of Work
* User Context
* Correlation IDs
* Audit Information

Sharing them across requests would cause data corruption and thread-safety issues.

⸻

Key Takeaway (Interview Summary)

Middleware Instance
      =
Singleton-like
InvokeAsync Parameters
      =
Resolved Per Request
Scoped Services
      =
Safe inside InvokeAsync
Scoped Services
      =
NOT Safe in Middleware Constructor

This distinction between constructor injection (application lifetime) and InvokeAsync injection (request lifetime) is one of the most frequently asked ASP.NET Core Dependency Injection and Middleware interview topics for Senior, Lead, and Principal Engineer roles.