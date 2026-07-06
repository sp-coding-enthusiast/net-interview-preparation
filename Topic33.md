# 32. CORS Middleware in ASP.NET Core

CORS (Cross-Origin Resource Sharing) Middleware is a security feature that controls **which websites are allowed to access your API or web application**.

Without CORS, modern browsers block web pages from making requests to a different domain, protocol, or port for security reasons.

---

# What is CORS?

## Definition

CORS (Cross-Origin Resource Sharing) is a browser security mechanism that allows or blocks requests from one origin to another based on rules configured on the server.

In simple words,

> **CORS decides who is allowed to call your API from a browser.**

---

# Layman Example: Apartment Security

Imagine you live in a gated apartment.

The security guard asks every visitor:

```
Which apartment are you visiting?
```

If the visitor is on the approved list,

```
Entry Allowed
```

Otherwise,

```
Access Denied
```

The security guard is exactly like **CORS Middleware**.

---

# Another Example: Office Building

Employees from your company can enter freely.

Visitors from another company need permission.

```
Employee

↓

Allowed
```

```
Visitor

↓

Permission Required

↓

Allowed or Denied
```

This is how CORS works.

---

# What is an Origin?

An **Origin** consists of:

```
Protocol + Domain + Port
```

Example

```
https://example.com:443
```

All three together define an origin.

---

# Examples

| URL | Origin |
|------|--------|
| https://example.com | Origin A |
| http://example.com | Different Origin (protocol changed) |
| https://api.example.com | Different Origin (subdomain changed) |
| https://example.com:5001 | Different Origin (port changed) |

Even a small change creates a different origin.

---

# Why Do We Need CORS?

Suppose your frontend runs at

```
https://myshop.com
```

Your API runs at

```
https://api.myshop.com
```

Browser request

```
Frontend

↓

API
```

Different origins.

Without CORS

```
Browser

↓

Blocked
```

---

# Same-Origin Policy

Browsers enforce the **Same-Origin Policy**.

It means

```
JavaScript

↓

Can access

↓

Same Origin Only
```

Different origin?

Blocked unless the server explicitly allows it using CORS.

---

# Request Flow Without CORS

```
Browser

↓

Frontend

↓

API

↓

Browser Blocks Response
```

Console error

```
Access to fetch has been blocked by CORS policy.
```

---

# Request Flow With CORS

```
Browser

↓

Frontend

↓

API

↓

CORS Middleware

↓

Allowed?

↓

Yes

↓

Browser Receives Response
```

---

# Enable CORS

## Step 1

Register CORS services

```csharp
builder.Services.AddCors();
```

---

## Step 2

Configure a policy

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAngular",
        policy =>
        {
            policy.WithOrigins("https://localhost:4200")
                  .AllowAnyHeader()
                  .AllowAnyMethod();
        });
});
```

---

## Step 3

Enable middleware

```csharp
app.UseCors("AllowAngular");
```

---

# Complete Example

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors(options =>
{
    options.AddPolicy("MyPolicy", policy =>
    {
        policy.WithOrigins("https://localhost:4200")
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();

app.UseCors("MyPolicy");

app.MapControllers();

app.Run();
```

---

# Allow Multiple Origins

```csharp
policy.WithOrigins(
    "https://abc.com",
    "https://xyz.com");
```

Only these websites are allowed.

---

# Allow Any Origin

```csharp
policy.AllowAnyOrigin();
```

This means

```
Any Website

↓

Can Access API
```

Useful for public APIs, but generally **not recommended** for secured applications.

---

# Allow Specific Methods

```csharp
policy.WithMethods(
    "GET",
    "POST");
```

Only

```
GET

POST
```

are allowed.

---

# Allow Any Method

```csharp
policy.AllowAnyMethod();
```

Allows

- GET
- POST
- PUT
- DELETE
- PATCH
- OPTIONS

---

# Allow Specific Headers

```csharp
policy.WithHeaders(
    "Content-Type",
    "Authorization");
```

---

# Allow Any Header

```csharp
policy.AllowAnyHeader();
```

---

# Allow Credentials

Suppose authentication uses cookies.

```csharp
policy.AllowCredentials();
```

This allows browsers to include credentials such as cookies or authentication information in cross-origin requests.

> **Important:** `AllowCredentials()` **cannot** be combined with `AllowAnyOrigin()`. You must specify explicit origins.

---

# Middleware Pipeline

```
Browser
    │
    ▼
Exception Middleware
    │
    ▼
Routing
    │
    ▼
CORS Middleware
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

CORS should execute after routing and before authentication/authorization.

---

# Real-World Example

Frontend

```
https://amazon.com
```

API

```
https://api.amazon.com
```

Browser

```
Amazon Website

↓

Call API

↓

CORS Middleware

↓

Allowed?

↓

Yes

↓

Products Returned
```

---

# Common CORS Headers

Server response

```
Access-Control-Allow-Origin

https://localhost:4200
```

Meaning

```
This origin

↓

May access my API
```

Other common headers include:

```
Access-Control-Allow-Methods

Access-Control-Allow-Headers

Access-Control-Allow-Credentials
```

---

# Preflight Request (OPTIONS)

For certain cross-origin requests (such as `PUT`, `DELETE`, or requests with custom headers), the browser first sends an **OPTIONS** request.

```
Browser

↓

OPTIONS

↓

Server

↓

Allowed?

↓

Yes

↓

Actual Request
```

This is called a **Preflight Request**.

---

# Benefits

- Protects APIs from unauthorized browser access.
- Allows trusted frontend applications to communicate with APIs.
- Supports modern SPA frameworks such as Angular, React, and Vue.
- Provides fine-grained control over allowed origins, methods, and headers.

---

# Common Mistakes

### Using `AllowAnyOrigin()` in production

```csharp
policy.AllowAnyOrigin();
```

This may expose your API to any website. Prefer specifying trusted origins.

---

### Forgetting `UseCors()`

Registering services alone is not enough.

You must also enable the middleware.

---

### Wrong Middleware Order

Incorrect

```csharp
app.UseAuthorization();

app.UseCors();
```

Correct

```csharp
app.UseRouting();

app.UseCors();

app.UseAuthentication();

app.UseAuthorization();
```

---

### Combining `AllowAnyOrigin()` and `AllowCredentials()`

This configuration is invalid and ASP.NET Core prevents it because it is insecure.

---

# Best Practices

- Allow only trusted origins using `WithOrigins()`.
- Restrict HTTP methods and headers whenever possible.
- Enable credentials only when necessary.
- Place `UseCors()` before authentication and authorization middleware.
- Review CORS settings carefully in production environments.

---

# Frequently Asked Interview Questions

### What is CORS?

CORS (Cross-Origin Resource Sharing) is a browser security mechanism that allows a server to specify which origins are permitted to access its resources.

---

### Why is CORS needed?

Browsers enforce the Same-Origin Policy. CORS allows safe cross-origin communication between trusted web applications and APIs.

---

### Does CORS protect APIs from tools like Postman?

No.

CORS is enforced by **web browsers**.

Tools such as Postman, curl, or server-to-server requests are not restricted by CORS.

---

### What is a Preflight Request?

A preflight request is an HTTP `OPTIONS` request that browsers send before certain cross-origin requests to verify that the server allows the actual request.

---

# Complete Flow

```
Browser
      │
      ▼
Frontend (https://app.example.com)
      │
      ▼
API Request
      │
      ▼
CORS Middleware
      │
      ├── Origin Allowed?
      │         │
      │         ├── Yes
      │         │      │
      │         │      ▼
      │         │  Continue to Controller
      │         │
      │         └── No
      │
      ▼
Browser Blocks Response
```

---

# Interview Answer (2-Minute Version)

**What is CORS Middleware in ASP.NET Core?**

CORS (Cross-Origin Resource Sharing) Middleware enables controlled cross-origin communication between browsers and servers. It allows the server to specify which origins, HTTP methods, headers, and credentials are permitted for cross-origin requests. This is necessary because browsers enforce the Same-Origin Policy by default. In ASP.NET Core, CORS is configured using `AddCors()` and enabled using `UseCors()`. It should be placed in the middleware pipeline after routing and before authentication and authorization. Proper CORS configuration improves security by allowing access only from trusted client applications.