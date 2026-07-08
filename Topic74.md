# 76. Filters in ASP.NET Core

---

# What are Filters?

Filters are components in ASP.NET Core that allow you to execute code **before or after different stages of the request pipeline**.

Think of filters as **security guards or checkpoints**.

Whenever a request enters your application, it passes through several checkpoints before reaching the controller.

Each checkpoint can:

- Check something
- Modify something
- Stop execution
- Log information
- Handle exceptions

---

# Real-Life Analogy

Imagine boarding a flight.

You don't directly enter the airplane.

Instead, you pass through several checkpoints.

```
Airport Entry

↓

Security Check

↓

Passport Check

↓

Boarding Gate

↓

Flight
```

Each checkpoint performs a specific task.

ASP.NET Core Filters work similarly.

```
HTTP Request

↓

Authorization Filter

↓

Resource Filter

↓

Action Filter

↓

Controller Action

↓

Result Filter

↓

Response
```

---

# Why Do We Need Filters?

Without filters, every controller action would contain repetitive code.

Example:

```csharp
public IActionResult GetUsers()
{
    Log();

    Validate();

    CheckPermission();

    ExecuteBusinessLogic();

    Log();

    return Ok();
}
```

Every API would repeat this.

Filters solve this problem.

---

# Types of Filters

ASP.NET Core provides several filters.

| Filter | Purpose |
|----------|----------|
| Authorization Filter | Authentication & Authorization |
| Resource Filter | Before and after model binding |
| Action Filter | Before and after controller action |
| Exception Filter | Handle exceptions |
| Result Filter | Before and after response |
| Endpoint Filter (.NET 7+) | Filters for Minimal APIs |

---

# Request Execution Order

```
HTTP Request

↓

Authorization Filter

↓

Resource Filter

↓

Model Binding

↓

Action Filter

↓

Controller Action

↓

Result Filter

↓

HTTP Response
```

If an exception occurs

↓

Exception Filter

---

# Filter Scopes

Filters can be applied

- Globally
- Controller level
- Action level

Example

```csharp
[MyFilter]
public class UsersController : ControllerBase
{
}
```

or

```csharp
[MyFilter]
public IActionResult Get()
{
}
```

---

# Built-in Examples

```
AuthorizeAttribute

ResponseCacheAttribute

AutoValidateAntiforgeryToken

TypeFilterAttribute

ServiceFilterAttribute
```

---

# Advantages

- Reusable
- Cleaner code
- Centralized logic
- Easy maintenance
- Cross-cutting concerns

---

# Interview Questions

## What are filters?

Filters are components that execute before or after different stages of the ASP.NET Core request pipeline.

---

## Why use filters?

To avoid duplicated code and implement cross-cutting concerns like logging, authorization, caching, and exception handling.

---

# Quick Revision

- Filters execute around request processing.
- Reduce duplicated code.
- Can run before or after controller execution.
- Applied globally, at controller level, or action level.

---

# 77. Action Filters

---

# What is an Action Filter?

Action Filters execute

- Before a controller action
- After a controller action

They are mainly used for

- Logging
- Validation
- Measuring execution time
- Modifying requests
- Modifying responses

---

# Real-Life Analogy

Imagine a teacher taking an exam.

Before exam

```
Distribute Question Paper
```

Students write exam.

After exam

```
Collect Answer Sheets
```

The teacher performs work before and after.

Exactly what Action Filters do.

---

# Execution Flow

```
HTTP Request

↓

Action Filter (Before)

↓

Controller Action

↓

Action Filter (After)

↓

HTTP Response
```

---

# Example

```csharp
using Microsoft.AspNetCore.Mvc.Filters;

public class LoggingFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        Console.WriteLine("Before Action");
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        Console.WriteLine("After Action");
    }
}
```

---

Apply it

```csharp
[LoggingFilter]
public IActionResult GetUsers()
{
    return Ok();
}
```

Output

```
Before Action

Controller Executes

After Action
```

---

# Common Uses

- Logging
- Performance measurement
- Input modification
- Output modification
- Auditing

---

# Interview Questions

## When do Action Filters execute?

Immediately before and immediately after the controller action executes.

---

## Can Action Filters modify the response?

Yes.

They can inspect or modify the response before it is sent.

---

# Quick Revision

- Execute before and after controller actions.
- Useful for logging, auditing, timing, and validation.
- Implement by inheriting from `ActionFilterAttribute`.

---

# 78. Exception Filters

---

# What is an Exception Filter?

Exception Filters catch **unhandled exceptions** thrown by controller actions.

Instead of crashing the application, they allow you to return a meaningful response.

---

# Real-Life Analogy

Imagine a customer slips inside a shopping mall.

Instead of everyone panicking,

the manager handles the situation.

The customer doesn't see the internal chaos.

Exception Filters work the same way.

---

# Without Exception Filter

```
Controller

↓

Exception

↓

Application Error

↓

500 Internal Server Error
```

---

# With Exception Filter

```
Controller

↓

Exception

↓

Exception Filter

↓

Friendly Error Response
```

---

# Example

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;

public class ApiExceptionFilter : ExceptionFilterAttribute
{
    public override void OnException(ExceptionContext context)
    {
        context.Result = new ObjectResult("Something went wrong.")
        {
            StatusCode = 500
        };

        context.ExceptionHandled = true;
    }
}
```

---

Apply it

```csharp
[ApiExceptionFilter]
public IActionResult Get()
{
    throw new Exception();
}
```

Response

```
500

Something went wrong.
```

---

# Common Uses

- Global error handling
- Logging exceptions
- Returning consistent error responses
- Hiding internal implementation details

---

# Exception Filter vs Middleware

| Exception Filter | Exception Middleware |
|------------------|----------------------|
| Works only for MVC actions | Works for the entire application |
| Cannot catch middleware exceptions | Can catch almost all exceptions |
| Executes later in the pipeline | Executes earlier in the pipeline |

In modern ASP.NET Core, **Exception Handling Middleware** is generally preferred for global exception handling.

---

# Interview Questions

## What does an Exception Filter do?

It catches unhandled exceptions from MVC actions and converts them into meaningful HTTP responses.

---

## Can it catch exceptions thrown by middleware?

No.

Middleware exceptions should be handled by exception handling middleware.

---

# Quick Revision

- Handles controller exceptions.
- Prevents raw error pages.
- Returns friendly error responses.
- Middleware is preferred for application-wide exception handling.

---

# 79. Resource Filters

---

# What is a Resource Filter?

Resource Filters execute

- Before model binding
- After the action result

They wrap almost the entire MVC execution process.

---

# Why Are They Special?

Unlike Action Filters,

Resource Filters run **before model binding**.

This means they can:

- Skip model binding
- Skip controller execution
- Implement caching
- Allocate or release resources

---

# Real-Life Analogy

Imagine entering a movie theater.

Before you even reach your seat,

security checks your ticket.

If the ticket is invalid,

you never enter the theater.

Resource Filters work similarly.

---

# Execution

```
Request

↓

Resource Filter

↓

Model Binding

↓

Action Filter

↓

Controller

↓

Result

↓

Resource Filter
```

---

# Example

```csharp
using Microsoft.AspNetCore.Mvc.Filters;

public class ResourceLoggingFilter : Attribute, IResourceFilter
{
    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        Console.WriteLine("Before Resource");
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        Console.WriteLine("After Resource");
    }
}
```

---

Apply

```csharp
[ResourceLoggingFilter]
public IActionResult Get()
{
    return Ok();
}
```

---

# Common Uses

- Response caching
- Resource allocation
- Performance optimization
- Skipping expensive processing

---

# Interview Questions

## When does a Resource Filter execute?

Before model binding starts and after the action result finishes.

---

## Why are Resource Filters useful?

They can short-circuit requests before expensive operations such as model binding occur.

---

# Quick Revision

- Execute before model binding.
- Wrap almost the entire MVC pipeline.
- Commonly used for caching and optimization.

---

# 80. Endpoint Filters (.NET 7+)

---

# What are Endpoint Filters?

Endpoint Filters are filters designed specifically for **Minimal APIs** introduced in .NET 7.

They provide functionality similar to Action Filters but work with Minimal API endpoints instead of MVC controllers.

---

# Real-Life Analogy

Imagine a security guard standing outside a meeting room.

Before anyone enters,

the guard checks:

- Identity
- Meeting invitation
- Security clearance

After the meeting,

the guard records who attended.

Endpoint Filters work similarly for Minimal APIs.

---

# Execution Flow

```
Request

↓

Endpoint Filter

↓

Minimal API Endpoint

↓

Endpoint Filter

↓

Response
```

---

# Example

```csharp
public class LoggingFilter : IEndpointFilter
{
    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context,
        EndpointFilterDelegate next)
    {
        Console.WriteLine("Before");

        var result = await next(context);

        Console.WriteLine("After");

        return result;
    }
}
```

---

Register

```csharp
app.MapGet("/users", () =>
{
    return Results.Ok("Users");
})
.AddEndpointFilter<LoggingFilter>();
```

Execution

```
Before

Endpoint Executes

After
```

---

# Common Uses

- Logging
- Validation
- Authentication
- Performance measurement
- Request modification
- Response modification

---

# Endpoint Filter vs Action Filter

| Endpoint Filter | Action Filter |
|-----------------|---------------|
| Used with Minimal APIs | Used with MVC Controllers |
| Introduced in .NET 7 | Available since ASP.NET MVC |
| Implements `IEndpointFilter` | Inherits `ActionFilterAttribute` or implements `IActionFilter` |

---

# Complete Request Pipeline

```
HTTP Request

↓

Middleware

↓

Authorization Filter

↓

Resource Filter

↓

Model Binding

↓

Action Filter

↓

Controller

↓

Result Filter

↓

HTTP Response

        ↑

Exception Filter (if needed)
```

For Minimal APIs

```
HTTP Request

↓

Middleware

↓

Endpoint Filter

↓

Minimal API

↓

Response
```

---

# Interview Questions

## What is an Endpoint Filter?

An Endpoint Filter is a filter for Minimal APIs that executes code before and after an endpoint handler.

---

## Why were Endpoint Filters introduced?

To bring reusable cross-cutting functionality such as logging, validation, and authorization to Minimal APIs without requiring MVC controllers.

---

## Difference between Action Filter and Endpoint Filter?

Action Filters work with MVC controllers, while Endpoint Filters work with Minimal APIs.

---

# Quick Revision

- Endpoint Filters are for Minimal APIs.
- Introduced in .NET 7.
- Execute before and after endpoint handlers.
- Implement `IEndpointFilter`.
- Ideal for logging, validation, authentication, and reusable request processing.