# 31. Response Caching Middleware in ASP.NET Core

Response Caching Middleware is a **performance optimization** feature in ASP.NET Core.

It stores the response of a request and reuses it for future requests instead of executing the controller every time.

This reduces:

- Server load
- Database calls
- CPU usage
- Response time

---

# What is Response Caching Middleware?

## Definition

Response Caching Middleware stores an HTTP response in memory (cache) and returns the cached response for identical requests, avoiding repeated execution of the controller.

Instead of generating the same response repeatedly, ASP.NET Core serves it directly from the cache.

---

# Layman Example: Restaurant

Imagine you order a popular dish.

Without caching

```
Customer

↓

Chef

↓

Cook Food

↓

Serve
```

Every customer waits for the chef.

---

With caching

The chef prepares a large batch in advance.

```
Customer

↓

Already Prepared

↓

Serve Immediately
```

No cooking required.

That's Response Caching.

---

# Another Example: Photocopy Shop

Without caching

```
Customer

↓

Create Document Again

↓

Print

↓

Deliver
```

Every customer gets a newly created document.

---

With caching

```
Customer

↓

Already Printed Copy

↓

Give Copy

↓

Done
```

Much faster.

---

# Why Do We Need Response Caching?

Suppose an API returns

```
GET /products
```

Every request

```
↓

Database

↓

Read 1000 Products

↓

Return JSON
```

If 10,000 users request the same data,

the database is queried 10,000 times.

---

With Response Caching

First request

```
Browser

↓

Controller

↓

Database

↓

Response

↓

Store in Cache
```

---

Second request

```
Browser

↓

Cache

↓

Return Cached Response
```

No database call.

---

# Request Flow

Without caching

```
Browser

↓

Controller

↓

Database

↓

Response

↓

Browser
```

---

With caching

First request

```
Browser

↓

Controller

↓

Database

↓

Cache

↓

Browser
```

Next request

```
Browser

↓

Cache

↓

Browser
```

---

# Enable Response Caching

## Step 1

Register services

```csharp
builder.Services.AddResponseCaching();
```

---

## Step 2

Enable middleware

```csharp
app.UseResponseCaching();
```

---

# Complete Example

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddResponseCaching();

var app = builder.Build();

app.UseResponseCaching();

app.MapControllers();

app.Run();
```

---

# Enable Caching for an Endpoint

Response Caching respects HTTP cache headers.

Example:

```csharp
[ResponseCache(Duration = 60)]
public IActionResult Products()
{
    return Ok(GetProducts());
}
```

Meaning

```
Cache this response

for

60 seconds
```

---

# First Request

```
Browser

↓

Controller

↓

Database

↓

Response

↓

Store in Cache
```

---

# Second Request

Within 60 seconds

```
Browser

↓

Cache

↓

Response
```

Controller is skipped.

---

# After 60 Seconds

```
Cache Expired

↓

Controller

↓

Database

↓

New Cache Created
```

---

# Example Timeline

```
10:00

↓

GET /products

↓

Database

↓

Cache Created
```

---

```
10:00:30

↓

GET /products

↓

Cache

↓

Response
```

---

```
10:01:05

↓

Cache Expired

↓

Database

↓

New Response

↓

Cache Again
```

---

# What Gets Cached?

Typically:

- JSON
- HTML
- Text
- API responses

Provided the response is considered cacheable based on HTTP caching rules.

---

# What Should NOT Be Cached?

Avoid caching:

- User profile pages
- Banking transactions
- Shopping cart contents
- Account balances
- Payment responses
- Personalized dashboards

These responses are user-specific and may expose incorrect or sensitive information if shared.

---

# Real-World Example

News website

```
GET /latest-news
```

News changes every

```
5 minutes
```

First request

```
↓

Database

↓

Latest News

↓

Cache
```

Next 1,000 users

```
↓

Cache

↓

Latest News
```

Database is not queried again until the cache expires.

---

# Middleware Pipeline

```
Browser
    │
    ▼
Exception Middleware
    │
    ▼
Response Caching Middleware
    │
    ├── Cached Response Exists?
    │         │
    │         ├── Yes → Return Cached Response
    │         │
    │         └── No
    ▼
Routing
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
Response Stored in Cache
    │
    ▼
Browser
```

---

# Cache Headers

A response may include headers such as:

```
Cache-Control: public,max-age=60
```

Meaning

```
Store response

for

60 seconds
```

These headers determine whether the middleware can cache the response.

---

# Response Caching vs Response Compression

| Response Caching | Response Compression |
|------------------|----------------------|
| Stores responses | Compresses responses |
| Improves repeated requests | Reduces response size |
| Reduces database and controller execution | Reduces bandwidth usage |
| Returns cached content | Returns compressed content |
| Saves server processing | Saves network transfer |

---

# Response Caching vs Output Caching

This is a common interview question.

| Response Caching | Output Caching |
|------------------|----------------|
| Follows standard HTTP caching headers | Configured entirely on the server |
| Depends on client request/response cache directives | Does not rely on client cache headers |
| Limited flexibility | More powerful and flexible |
| Suitable for HTTP-compliant caching | Recommended for most new ASP.NET Core applications (.NET 7+) |

> **Note:** In modern ASP.NET Core applications, **Output Caching** is generally preferred over **Response Caching** because it provides better control and isn't limited by HTTP cache semantics.

---

# Benefits

- Faster responses
- Lower database load
- Lower CPU usage
- Better scalability
- Improved user experience
- Reduced infrastructure costs

---

# Best Practices

- Cache only data that doesn't change frequently.
- Never cache personalized or sensitive responses.
- Set an appropriate cache duration based on how often data changes.
- Combine response caching with response compression for even better performance.
- For new ASP.NET Core applications targeting .NET 7 or later, consider **Output Caching** instead of Response Caching.

---

# Frequently Asked Interview Questions

### What does Response Caching Middleware do?

It stores cacheable HTTP responses and serves them directly for subsequent identical requests, reducing controller execution and backend processing.

---

### Does it cache every response?

No.

Only responses that satisfy HTTP caching rules (for example, appropriate `Cache-Control` headers) are cached.

---

### Should user-specific pages be cached?

No.

Pages containing personalized or sensitive information should not be cached because they may expose incorrect data to other users.

---

### What is the difference between Response Caching and Output Caching?

Response Caching is based on standard HTTP caching semantics and client cache headers.

Output Caching is server-controlled, more configurable, and is the recommended approach for most modern ASP.NET Core applications.

---

# Complete Flow

```
Browser
      │
      ▼
GET /products
      │
      ▼
Response Caching Middleware
      │
      ├── Cache Hit?
      │        │
      │        ├── Yes
      │        │      │
      │        │      ▼
      │        │  Return Cached Response
      │        │
      │        └── No
      ▼
Controller
      │
      ▼
Database
      │
      ▼
Generate Response
      │
      ▼
Store in Cache
      │
      ▼
Browser
```

---

# Interview Answer (2-Minute Version)

**What is Response Caching Middleware in ASP.NET Core?**

Response Caching Middleware stores cacheable HTTP responses and serves them directly for subsequent identical requests, reducing controller execution, database access, and overall server workload. It follows standard HTTP caching rules and relies on cache-related headers such as `Cache-Control`. It is enabled using `AddResponseCaching()` and `UseResponseCaching()`, and endpoints can specify caching behavior using the `[ResponseCache]` attribute. While it improves performance for public, infrequently changing content, it should not be used for personalized or sensitive data. For modern ASP.NET Core applications, **Output Caching** is often the preferred server-side caching solution because it offers greater flexibility and control.