# 4. `Use()` vs `Run()` vs `Map()` in ASP.NET Core Middleware

One of the most frequently asked ASP.NET Core interview questions is:

> **What is the difference between `Use()`, `Run()`, and `Map()`?**

The easiest way to understand them is by imagining a **highway**.

---

# Layman Example: Highway Checkpoints

Imagine you're driving to a shopping mall.

On the way, you encounter several checkpoints.

```
Home
 │
 ▼
Traffic Police
 │
 ▼
Toll Booth
 │
 ▼
Security Check
 │
 ▼
Shopping Mall
```

Each checkpoint decides one of three things:

- Let the car continue → `Use()`
- Stop the car here → `Run()`
- Send the car to another road → `Map()`

That's exactly how middleware works.

---

# What is `Use()`?

## Definition

`Use()` adds middleware that can:

- Process the request
- Call the next middleware
- Process the response while returning

It is the **most commonly used middleware**.

---

# Think of `Use()` as

A security guard saying:

> "I've completed my check. You may continue."

```
Visitor
   │
   ▼
Security Guard
   │
   ▼
Reception
```

---

# Syntax

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Before");

    await next();

    Console.WriteLine("After");
});
```

Notice

```
await next();
```

This passes the request to the next middleware.

---

# Execution

```
Request

↓

Use Middleware

↓

next()

↓

Next Middleware

↓

Controller

↓

Response

↓

Use Middleware Again

↓

Browser
```

The same middleware executes **twice**:

- Before the next middleware
- After the response comes back

---

# Real Example

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Logging Request");

    await next();

    Console.WriteLine("Logging Response");
});
```

This middleware logs both the incoming request and outgoing response.

---

# What is `Run()`?

## Definition

`Run()` is a **terminal middleware**.

It handles the request and **does not call the next middleware**.

Once `Run()` executes, the pipeline ends.

---

# Layman Example

Imagine you're entering a building.

```
Security Guard

↓

Reception

↓

Office
```

The security guard says

> "Sorry, visitors are not allowed."

You return home immediately.

You never reach reception.

That is exactly how `Run()` works.

---

# Syntax

```csharp
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello World");
});
```

Notice

There is **no** `next()`.

---

# Execution

```
Browser

↓

Run Middleware

↓

Response

↓

Browser
```

No other middleware executes after `Run()`.

---

# Example

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Middleware 1");

    await next();
});

app.Run(async context =>
{
    Console.WriteLine("Run Middleware");

    await context.Response.WriteAsync("Hello");
});

app.Use(async (context, next) =>
{
    Console.WriteLine("Middleware 2");

    await next();
});
```

---

# Output

```
Middleware 1

Run Middleware
```

`Middleware 2` never executes because `Run()` stops the pipeline.

---

# What is `Map()`?

## Definition

`Map()` creates a **branch** in the middleware pipeline.

It routes requests to a different pipeline based on the URL path.

---

# Layman Example

Imagine you're driving.

```
Main Road
      │
      ├──────── Airport
      │
      ├──────── Hospital
      │
      └──────── Shopping Mall
```

Depending on the destination, you take a different road.

`Map()` works exactly like that.

---

# Example

```csharp
app.Map("/admin", adminApp =>
{
    adminApp.Run(async context =>
    {
        await context.Response.WriteAsync("Admin Area");
    });
});

app.Run(async context =>
{
    await context.Response.WriteAsync("Home Page");
});
```

---

# Request 1

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

Output

```
Admin Area
```

---

# Request 2

```
GET /
```

Pipeline

```
Browser

↓

Skip Admin Pipeline

↓

Default Run()

↓

Home Page
```

Output

```
Home Page
```

---

# Real-World Example

Suppose your application has:

```
/api

/admin

/customer

/health
```

You can create separate pipelines for each.

```csharp
app.Map("/api", api =>
{
    api.UseAuthentication();
    api.Run(async context =>
    {
        await context.Response.WriteAsync("API");
    });
});

app.Map("/admin", admin =>
{
    admin.UseAuthorization();
    admin.Run(async context =>
    {
        await context.Response.WriteAsync("Admin");
    });
});
```

Each branch has its own middleware pipeline.

---

# Visual Diagram

```
                     Incoming Request
                            │
                            ▼
                     Is URL "/admin"?
                     /             \
                   Yes              No
                    │                │
                    ▼                ▼
             Admin Pipeline     Main Pipeline
                    │                │
                    ▼                ▼
               Admin Response   Home Response
```

---

# Difference Between `Use()`, `Run()`, and `Map()`

| Feature | `Use()` | `Run()` | `Map()` |
|----------|----------|----------|----------|
| Continues pipeline | ✅ Yes | ❌ No | ✅ Yes (to a branch) |
| Calls `next()` | ✅ Yes | ❌ No | Branch-specific |
| Terminal middleware | ❌ No | ✅ Yes | Usually ends in `Run()` |
| Creates new pipeline | ❌ No | ❌ No | ✅ Yes |
| Used for | Logging, Authentication, Routing, CORS | Final response | Route-specific middleware |

---

# Easy Memory Trick

## `Use()`

> **Use → Continue**

```
Use()

↓

Next Middleware
```

Think:

> "Do my work and continue."

---

## `Run()`

> **Run → Finish**

```
Run()

↓

Response

↓

End
```

Think:

> "Handle it here and stop."

---

## `Map()`

> **Map → Branch**

```
Request

        │
        ├── /admin
        │
        ├── /api
        │
        └── /
```

Think:

> "If the URL matches, take another road."

---

# When to Use Each?

### Use `Use()`

- Logging
- Authentication
- Authorization
- CORS
- Exception handling
- Request/response modification

---

### Use `Run()`

- Return a final response
- End the pipeline
- Health check endpoints
- Simple demo applications

---

### Use `Map()`

- `/admin`
- `/api`
- `/health`
- `/customer`
- `/tenant1`
- `/tenant2`

Whenever different URL paths require separate middleware pipelines.

---

# Interview Answer (2-Minute Version)

**What is the difference between `Use()`, `Run()`, and `Map()`?**

- **`Use()`** registers middleware that can execute logic before and after the next middleware by calling `await next()`. It is used for cross-cutting concerns such as logging, authentication, exception handling, and CORS.
- **`Run()`** registers terminal middleware. It generates the response and ends the pipeline because it does not call `next()`.
- **`Map()`** creates a separate middleware branch based on the request path. Requests matching the specified path are processed by that branch, while all other requests continue through the main pipeline.

### Simple Way to Remember

- **Use = Continue**
- **Run = Stop**
- **Map = Branch**