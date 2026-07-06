# 35. Endpoint Middleware in ASP.NET Core

Endpoint Middleware is the middleware responsible for **executing the endpoint** that was selected by the Routing Middleware.

In simple words,

- **Routing Middleware** finds **where the request should go**
- **Endpoint Middleware** actually **executes it**

This is one of the most commonly asked ASP.NET Core interview questions.

---

# What is Endpoint Middleware?

## Definition

Endpoint Middleware executes the endpoint (Controller, Minimal API, Razor Page, or SignalR Hub) that was selected by the Routing Middleware.

It is the **last middleware** before your application code runs.

---

# Layman Example: Restaurant

Imagine you enter a restaurant.

### Reception

```
Customer

↓

Reception

↓

Table Number 12
```

Reception only tells you where to sit.

It doesn't serve food.

This is **Routing Middleware**.

---

Now,

the waiter comes to your table.

```
Table 12

↓

Waiter

↓

Takes Order

↓

Kitchen

↓

Food
```

The waiter actually serves you.

This is **Endpoint Middleware**.

---

# Another Example: Hospital

Reception

```
Patient

↓

Reception

↓

Room 205
```

Reception only identifies the doctor.

---

Doctor

```
Patient

↓

Doctor

↓

Treatment
```

Doctor performs the actual work.

That's Endpoint Middleware.

---

# Why Do We Need Endpoint Middleware?

Suppose you request

```
GET /products
```

Routing only finds

```
ProductsController

↓

GetProducts()
```

But someone still needs to call

```
GetProducts()
```

That job belongs to Endpoint Middleware.

---

# Request Pipeline

```
Browser
    │
    ▼
Exception Middleware
    │
    ▼
HTTPS
    │
    ▼
Static Files
    │
    ▼
Routing Middleware
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
Controller Action
```

---

# Routing vs Endpoint

This is a very common interview question.

## Routing Middleware

```
Find Endpoint
```

Example

```
/products

↓

ProductsController

↓

GetProducts()
```

No execution happens yet.

---

## Endpoint Middleware

```
Execute

↓

GetProducts()
```

Now the controller runs.

---

# Visual Flow

```
Browser

↓

Routing

↓

Controller Found

↓

Endpoint Middleware

↓

Controller Executes

↓

Response
```

---

# Configure Endpoint Middleware

In older ASP.NET Core versions (3.x/5),

you explicitly used:

```csharp
app.UseRouting();

app.UseAuthentication();

app.UseAuthorization();

app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
});
```

`UseEndpoints()` was the Endpoint Middleware.

---

# Modern ASP.NET Core (.NET 6+)

With the minimal hosting model,

you typically write:

```csharp
app.MapControllers();
```

or

```csharp
app.MapGet("/", () => "Hello");
```

Internally,

ASP.NET Core still creates and executes Endpoint Middleware for you.

---

# Example with Controller

Controller

```csharp
[ApiController]
[Route("products")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok("Products");
    }
}
```

Program

```csharp
app.MapControllers();
```

Request

```
GET /products
```

Pipeline

```
Browser

↓

Routing

↓

ProductsController

↓

Endpoint Middleware

↓

Get()

↓

Response
```

---

# Example with Minimal API

```csharp
app.MapGet("/hello", () =>
{
    return "Hello World";
});
```

Request

```
GET /hello
```

Pipeline

```
Browser

↓

Routing

↓

Minimal API

↓

Endpoint Middleware

↓

Lambda Executes

↓

Hello World
```

---

# What Happens Internally?

Step 1

Routing

```
GET /products

↓

Find Endpoint
```

---

Step 2

Authentication

```
User Verified
```

---

Step 3

Authorization

```
Permission Verified
```

---

Step 4

Endpoint Middleware

```
Execute Controller
```

---

# Execution Flow

```
Incoming Request

↓

Routing

↓

Endpoint Selected

↓

Authentication

↓

Authorization

↓

Endpoint Middleware

↓

Execute Endpoint

↓

Response
```

---

# Why Authentication Comes Before Endpoint Middleware

Suppose

```
Admin Dashboard
```

You don't want the controller to execute unless:

- User is authenticated
- User is authorized

Pipeline

```
Routing

↓

Authentication

↓

Authorization

↓

Endpoint Middleware

↓

Admin Controller
```

---

# Middleware Pipeline

```
Browser
    │
    ▼
Exception Middleware
    │
    ▼
HTTPS Redirection
    │
    ▼
Static Files
    │
    ▼
Routing Middleware
    │
    ▼
CORS Middleware
    │
    ▼
Authentication Middleware
    │
    ▼
Authorization Middleware
    │
    ▼
Endpoint Middleware
    │
    ▼
Controller / Razor Page / Minimal API / SignalR Hub
    │
    ▼
Response
```

---

# Real-World Example

E-commerce Website

Request

```
GET /orders
```

Pipeline

```
Routing

↓

OrdersController

↓

Authentication

↓

Authorization

↓

Endpoint Middleware

↓

GetOrders()

↓

JSON Response
```

---

# Endpoint Types

Endpoint Middleware can execute different endpoint types.

| Endpoint Type | Example |
|--------------|----------|
| Controller Action | `ProductsController.Get()` |
| Minimal API | `app.MapGet()` |
| Razor Pages | `/Index.cshtml` |
| SignalR Hub | Chat Hub |
| gRPC Service | gRPC Endpoint |

---

# Common Mistakes

### Assuming Routing Executes the Controller

No.

Routing **only selects** the endpoint.

Endpoint Middleware executes it.

---

### Placing Authentication After Endpoint Execution

Incorrect

```
Routing

↓

Endpoint

↓

Authentication
```

The endpoint would execute before security checks.

---

### Forgetting Endpoint Mapping

If you don't map controllers or endpoints,

no endpoint exists for the middleware to execute.

---

# Best Practices

- Always configure routing before authentication and authorization.
- Execute authentication and authorization before the endpoint.
- Use `app.MapControllers()` (or `MapGet`, `MapPost`, etc.) in modern ASP.NET Core applications.
- Keep endpoint logic focused on business functionality, leaving cross-cutting concerns to middleware.

---

# Frequently Asked Interview Questions

### What is Endpoint Middleware?

Endpoint Middleware executes the endpoint selected by the Routing Middleware, such as a controller action, Minimal API, Razor Page, or SignalR Hub.

---

### What is the difference between Routing Middleware and Endpoint Middleware?

| Routing Middleware | Endpoint Middleware |
|--------------------|---------------------|
| Selects the endpoint | Executes the selected endpoint |
| Matches URL patterns | Invokes application code |
| Does not execute business logic | Runs business logic |

---

### Is `UseEndpoints()` still used?

In ASP.NET Core 3.x and 5, `UseEndpoints()` was used explicitly.

In .NET 6+ with the minimal hosting model, methods like `MapControllers()` and `MapGet()` replace the explicit call, but Endpoint Middleware still exists internally.

---

### Can Endpoint Middleware execute Minimal APIs?

Yes.

It can execute:

- Controllers
- Minimal APIs
- Razor Pages
- SignalR Hubs
- gRPC services

---

# Complete Flow

```
Browser
      │
      ▼
Routing Middleware
      │
      ▼
Find Matching Endpoint
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
Execute Controller / Minimal API
      │
      ▼
Generate Response
      │
      ▼
Browser
```

---

# Interview Answer (2-Minute Version)

**What is Endpoint Middleware in ASP.NET Core?**

Endpoint Middleware is responsible for executing the endpoint selected by the Routing Middleware. Routing determines which endpoint matches the incoming request, while Endpoint Middleware invokes that endpoint, such as a controller action, Minimal API, Razor Page, SignalR Hub, or gRPC service. It executes after routing, authentication, and authorization have completed successfully. In modern ASP.NET Core applications (.NET 6+), Endpoint Middleware is configured implicitly through endpoint mapping methods such as `app.MapControllers()` and `app.MapGet()`, whereas older versions used `app.UseEndpoints()`.