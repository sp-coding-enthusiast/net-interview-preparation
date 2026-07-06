# 37. Branching Middleware in ASP.NET Core

Branching Middleware allows the ASP.NET Core request pipeline to **split into different execution paths** based on the incoming request.

Instead of sending every request through the same middleware pipeline, you can create **different pipelines for different requests**.

This is useful when different URLs require different processing.

---

# What is Branching Middleware?

## Definition

Branching Middleware creates multiple request pipelines based on conditions such as:

- URL Path
- Host Name
- Request Header
- Query String
- HTTP Method
- Custom Logic

It allows different requests to execute different middleware.

---

# Layman Example: Highway

Imagine you're driving on a highway.

```
Main Highway

↓

Exit 1 → Airport

↓

Exit 2 → Railway Station

↓

Exit 3 → Shopping Mall
```

All vehicles start on the same road.

Depending on the destination,

they take different exits.

Branching Middleware works exactly the same way.

---

# Another Example: Hospital

```
Reception

↓

Children

↓

Pediatric Department
```

```
Adults

↓

General Medicine
```

```
Emergency

↓

Emergency Room
```

One entrance.

Multiple departments.

---

# Why Do We Need Branching?

Suppose our application has

```
/api

/admin

/images

/health
```

Should every request execute the same middleware?

Probably not.

Example

```
/images
```

doesn't need authentication.

```
/admin
```

must require authentication.

```
/health
```

may need only a lightweight response.

Branching lets us create specialized pipelines.

---

# Normal Pipeline

Without branching

```
Request

↓

Middleware A

↓

Middleware B

↓

Middleware C

↓

Controller
```

Every request follows the same path.

---

# Branching Pipeline

```
Request

↓

Middleware A

↓

Is URL /admin?

↓

Yes

↓

Admin Pipeline

↓

Admin Controller
```

```
No

↓

Normal Pipeline

↓

Controller
```

---

# ASP.NET Core Branching Methods

There are three commonly used branching methods:

- `Map()`
- `MapWhen()`
- `UseWhen()`

---

# 1. Map()

`Map()` creates a completely separate pipeline based on the URL path.

Example

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

## Request

```
GET /admin
```

Pipeline

```
Browser

↓

Map("/admin")

↓

Admin Pipeline

↓

Response
```

---

Request

```
GET /products
```

Pipeline

```
Normal Pipeline
```

---

# How Map() Works

```
Request

↓

Path Starts With /admin ?

↓

Yes

↓

Admin Branch

↓

Response
```

Otherwise

```
Continue Main Pipeline
```

---

# 2. MapWhen()

`MapWhen()` creates a new branch using **any condition**, not just the URL path.

Example

```csharp
app.MapWhen(
    context => context.Request.Query.ContainsKey("debug"),
    branch =>
    {
        branch.Run(async context =>
        {
            await context.Response.WriteAsync("Debug Mode");
        });
    });
```

---

Request

```
/products?debug=true
```

Branch executes.

---

Request

```
/products
```

Normal pipeline executes.

---

# Visual Flow

```
Request

↓

Query Contains debug?

↓

Yes

↓

Debug Branch

↓

Response
```

---

# 3. UseWhen()

`UseWhen()` also branches based on a condition.

Unlike `MapWhen()`, the branch can return to the main pipeline after executing its middleware.

Example

```csharp
app.UseWhen(
    context => context.Request.Path.StartsWithSegments("/api"),
    appBuilder =>
    {
        appBuilder.Use(async (context, next) =>
        {
            Console.WriteLine("API Request");

            await next();
        });
    });
```

---

Request

```
/api/products
```

Pipeline

```
Main Pipeline

↓

API Branch Middleware

↓

Back To Main Pipeline

↓

Controller
```

---

# Difference Between Map() and UseWhen()

This is a favorite interview question.

## Map()

```
Request

↓

Branch

↓

Ends Here
```

The request enters a separate pipeline.

It does **not** return to the original pipeline.

---

## UseWhen()

```
Request

↓

Branch

↓

Back To Main Pipeline

↓

Controller
```

The request returns after branch middleware finishes.

---

# Difference Between MapWhen() and UseWhen()

| MapWhen() | UseWhen() |
|------------|-----------|
| Creates a separate branch | Adds conditional middleware |
| Branch usually handles the request independently | Returns to the main pipeline after branch middleware |
| Often ends with `Run()` | Usually calls `next()` |

---

# Middleware Pipeline

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
Routing
      │
      ▼
Branch?
      │
      ├── /admin
      │      │
      │      ▼
      │  Admin Middleware
      │      │
      │      ▼
      │  Admin Controller
      │
      └── Normal Pipeline
             │
             ▼
      Authentication
             │
             ▼
      Authorization
             │
             ▼
      Controller
```

---

# Real-World Example

Suppose an application has

```
/api

↓

JWT Authentication
```

```
/admin

↓

Extra Logging

↓

Role Check
```

```
/health

↓

Return OK
```

Each branch has its own middleware.

---

# Example: Health Check

```csharp
app.Map("/health", healthApp =>
{
    healthApp.Run(async context =>
    {
        await context.Response.WriteAsync("Healthy");
    });
});
```

Request

```
GET /health
```

Immediately returns

```
Healthy
```

No controllers involved.

---

# Example: API Logging

```csharp
app.UseWhen(
    context => context.Request.Path.StartsWithSegments("/api"),
    api =>
    {
        api.Use(async (context, next) =>
        {
            Console.WriteLine("API Request");

            await next();
        });
    });
```

Only API requests are logged.

---

# Common Mistakes

### Using `Map()` When You Need to Return

`Map()` creates an independent branch.

If you need to continue to the main pipeline,

use `UseWhen()`.

---

### Forgetting `next()` Inside `UseWhen()`

```csharp
await next();
```

Without it,

the request stops inside the branch.

---

### Using `Map()` for Complex Conditions

`Map()` only matches URL paths.

For headers, query strings, or custom logic,

use `MapWhen()` or `UseWhen()`.

---

# Best Practices

- Use `Map()` for path-based branching.
- Use `MapWhen()` for custom conditional branching.
- Use `UseWhen()` when you want conditional middleware but still need to continue through the main pipeline.
- Keep branch-specific middleware focused on that branch's responsibilities.
- Avoid creating too many deeply nested branches, as they make the pipeline harder to understand.

---

# Frequently Asked Interview Questions

### What is Branching Middleware?

Branching Middleware allows ASP.NET Core to create different request pipelines based on request characteristics such as the URL path or custom conditions.

---

### What is the difference between `Map()` and `MapWhen()`?

- `Map()` branches based on a URL path.
- `MapWhen()` branches based on any custom condition.

---

### What is the difference between `Map()` and `UseWhen()`?

| Map() | UseWhen() |
|--------|-----------|
| Separate pipeline | Conditional middleware |
| Does not return to the main pipeline | Returns to the main pipeline |
| Typically ends the request | Usually continues processing |

---

### When should `UseWhen()` be used?

Use `UseWhen()` when only certain requests require additional middleware (such as logging or validation), but should still continue through the normal request pipeline.

---

# Complete Flow

```
Incoming Request
        │
        ▼
Main Pipeline
        │
        ▼
Condition?
        │
        ├── /admin
        │       │
        │       ▼
        │   Admin Branch
        │       │
        │       ▼
        │   Response
        │
        └── Normal Request
                │
                ▼
        Authentication
                │
                ▼
        Authorization
                │
                ▼
        Controller
                │
                ▼
        Response
```

---

# Easy Way to Remember

```
Map()

↓

Different Road
```

```
MapWhen()

↓

Different Road Based On Condition
```

```
UseWhen()

↓

Take a Small Detour

↓

Come Back
```

---

# Interview Answer (2-Minute Version)

**What is Branching Middleware in ASP.NET Core?**

Branching Middleware allows an ASP.NET Core application to split the request pipeline into different execution paths based on URL paths or custom conditions. It is useful when different types of requests require different middleware. ASP.NET Core provides three common branching methods: `Map()`, which creates a separate pipeline for a specific URL path; `MapWhen()`, which creates a separate pipeline based on any custom condition; and `UseWhen()`, which conditionally executes middleware and then returns to the main pipeline. Choosing the correct branching method helps keep the request pipeline efficient, modular, and easier to maintain.