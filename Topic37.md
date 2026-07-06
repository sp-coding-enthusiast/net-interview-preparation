# 36. Ordering of Middleware in ASP.NET Core

One of the **most important concepts** in ASP.NET Core is the **order of middleware**.

Middleware executes **in the exact order** in which it is added to the pipeline.

A wrong order can lead to:

- Authentication failures
- Authorization failures
- CORS issues
- Static files not loading
- Controllers not executing
- Security vulnerabilities

> **Interview Tip:** The order of middleware is one of the most frequently asked ASP.NET Core interview topics.

---

# What is Middleware Ordering?

## Definition

Middleware ordering refers to the sequence in which middleware components are registered in the request pipeline. Since each middleware can process the request before and after the next middleware, the registration order determines the application's behavior.

---

# Layman Example: Airport Security

Imagine you're boarding a flight.

Correct order:

```
Enter Airport

↓

Security Check

↓

Passport Verification

↓

Board Flight
```

Makes sense.

---

Wrong order:

```
Board Flight

↓

Passport Verification

↓

Security Check
```

Impossible.

Middleware works exactly the same way.

---

# Another Example: Restaurant

Correct order

```
Customer

↓

Reception

↓

Table Assigned

↓

Order Taken

↓

Chef Cooks

↓

Serve Food

↓

Bill

↓

Exit
```

---

Wrong order

```
Chef Cooks

↓

Reception

↓

Customer Arrives
```

Nothing works.

---

# Middleware Executes Like a Pipeline

Imagine the middleware pipeline as a series of checkpoints.

```
Request

↓

Middleware 1

↓

Middleware 2

↓

Middleware 3

↓

Controller

↓

Response

↑

Middleware 3

↑

Middleware 2

↑

Middleware 1
```

Notice:

- **Request** travels from top to bottom.
- **Response** travels from bottom to top.

This is often called the **"onion model"** because each middleware wraps the next one.

---

# Visual Representation

```
Request

↓

Exception Middleware

↓

HTTPS Redirection

↓

Static Files

↓

Routing

↓

CORS

↓

Authentication

↓

Authorization

↓

Endpoint

↓

Controller

↓

Response

↑

Endpoint

↑

Authorization

↑

Authentication

↑

CORS

↑

Routing

↑

Static Files

↑

HTTPS

↑

Exception Middleware
```

---

# Recommended Middleware Order

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddAuthentication();

builder.Services.AddAuthorization();

builder.Services.AddCors();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}

app.UseHttpsRedirection();

app.UseStaticFiles();

app.UseRouting();

app.UseCors("MyPolicy");

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();

app.Run();
```

---

# Why This Order?

## 1. Exception Middleware

```
Request

↓

Exception Middleware
```

Should be first so it can catch exceptions thrown by any later middleware.

---

## 2. HSTS

```
Browser

↓

HSTS Policy
```

Adds the `Strict-Transport-Security` header (typically only in production).

---

## 3. HTTPS Redirection

```
HTTP

↓

HTTPS
```

Redirects insecure requests before other middleware runs.

---

## 4. Static Files

```
logo.png

↓

Return Image
```

Static files don't require authentication or controllers, so serve them early.

---

## 5. Routing

```
GET /products

↓

ProductsController
```

Routing identifies the endpoint but does not execute it.

---

## 6. CORS

```
Origin Allowed?

↓

Yes

↓

Continue
```

CORS needs routing information to evaluate endpoint metadata and should run before authentication/authorization.

---

## 7. Authentication

```
JWT

↓

Identity Verified
```

Creates `HttpContext.User`.

---

## 8. Authorization

```
Admin?

↓

Yes

↓

Continue
```

Uses the authenticated identity to verify permissions.

---

## 9. Endpoint

```
Execute Controller
```

Runs the selected controller action, Minimal API, Razor Page, etc.

---

# Complete Flow

```
Browser
      │
      ▼
Exception Middleware
      │
      ▼
HSTS
      │
      ▼
HTTPS Redirection
      │
      ▼
Static Files
      │
      ▼
Routing
      │
      ▼
CORS
      │
      ▼
Authentication
      │
      ▼
Authorization
      │
      ▼
Endpoint
      │
      ▼
Controller
      │
      ▼
Response
```

---

# Wrong Order Example

```csharp
app.UseAuthorization();

app.UseAuthentication();
```

Execution

```
Authorization

↓

No User

↓

403
```

Authentication never had a chance to establish the user's identity.

---

# Correct Order

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

Execution

```
Authentication

↓

User Created

↓

Authorization

↓

Controller
```

---

# Another Wrong Order

```csharp
app.UseRouting();

app.MapControllers();

app.UseAuthentication();
```

Execution

```
Endpoint Executes

↓

Authentication Never Runs
```

Security is bypassed for that endpoint.

---

# Correct Version

```csharp
app.UseRouting();

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

---

# Middleware Execution Example

Suppose we have

```csharp
app.Use(async (ctx, next) =>
{
    Console.WriteLine("Middleware A - Before");

    await next();

    Console.WriteLine("Middleware A - After");
});

app.Use(async (ctx, next) =>
{
    Console.WriteLine("Middleware B - Before");

    await next();

    Console.WriteLine("Middleware B - After");
});

app.Run(async ctx =>
{
    Console.WriteLine("Controller");
});
```

Output

```
Middleware A - Before

Middleware B - Before

Controller

Middleware B - After

Middleware A - After
```

This demonstrates that:

- Requests flow **top to bottom**.
- Responses flow **bottom to top**.

---

# Onion Model

```
Exception

    Authentication

        Authorization

            Controller

        Authorization

    Authentication

Exception
```

Every middleware wraps the next one.

---

# Common Mistakes

### Authentication After Authorization

Incorrect

```csharp
app.UseAuthorization();

app.UseAuthentication();
```

---

### Routing After Authentication

Incorrect

```csharp
app.UseAuthentication();

app.UseRouting();
```

Authentication may not have endpoint metadata available when needed.

---

### CORS After Endpoint

Incorrect

```csharp
app.MapControllers();

app.UseCors();
```

CORS won't be applied to those endpoints.

---

### Static Files at the End

Incorrect

```csharp
app.MapControllers();

app.UseStaticFiles();
```

Static files should be served before endpoint execution.

---

# Best Practices

- Place exception handling middleware first.
- Enable HSTS (production) before HTTPS redirection.
- Redirect HTTP to HTTPS early.
- Serve static files before routing.
- Call `UseRouting()` before CORS, authentication, and authorization.
- Call `UseAuthentication()` before `UseAuthorization()`.
- Map endpoints (`MapControllers()`, `MapGet()`, etc.) after all request-processing middleware.

---

# Frequently Asked Interview Questions

### Does middleware execute in the order it is registered?

Yes.

Requests move from the first registered middleware to the last.

Responses travel back in reverse order.

---

### Why must Authentication come before Authorization?

Authorization depends on the authenticated user stored in `HttpContext.User`.

Without authentication, authorization has no identity to evaluate.

---

### Why should Static Files be before Routing?

If a requested static file exists, it can be returned immediately without invoking routing or controllers.

---

### What is the Onion Model?

The Onion Model describes how each middleware wraps the next middleware. Requests travel inward toward the endpoint, and responses travel outward in reverse order.

---

# Complete Pipeline Diagram

```
Incoming Request
        │
        ▼
Exception Middleware
        │
        ▼
HSTS
        │
        ▼
HTTPS Redirection
        │
        ▼
Static Files
        │
        ▼
Routing
        │
        ▼
CORS
        │
        ▼
Authentication
        │
        ▼
Authorization
        │
        ▼
Endpoint Middleware
        │
        ▼
Controller / Minimal API
        │
        ▼
Outgoing Response
```

---

# Easy Interview Mnemonic

Remember this order:

```
Exceptions

↓

HSTS

↓

HTTPS

↓

Static Files

↓

Routing

↓

CORS

↓

Authentication

↓

Authorization

↓

Endpoints
```

Or simply:

> **"Secure the request, find the route, verify the user, check permissions, then execute the endpoint."**

---

# Interview Answer (2-Minute Version)

**What is middleware ordering in ASP.NET Core?**

Middleware ordering determines how requests and responses flow through the ASP.NET Core pipeline. Each middleware executes in the order it is registered for incoming requests and in reverse order for outgoing responses. A typical production pipeline starts with exception handling, followed by HSTS (production), HTTPS redirection, static files, routing, CORS, authentication, authorization, and finally endpoint execution (`MapControllers()` or other mapped endpoints). Correct ordering is critical because many middleware components depend on earlier ones—for example, authorization requires authentication to have already established the user's identity.