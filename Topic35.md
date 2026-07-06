# 34. HTTPS Redirection Middleware in ASP.NET Core

HTTPS Redirection Middleware is a security feature that automatically redirects users from **HTTP** to **HTTPS**.

It ensures that all communication between the browser and your application is encrypted.

---

# What is HTTPS Redirection?

## Definition

HTTPS Redirection Middleware intercepts incoming **HTTP** requests and redirects them to the equivalent **HTTPS** URL.

For example:

```
http://example.com/products
```

becomes

```
https://example.com/products
```

before the request is processed by your application.

---

# Layman Example: Bank Entrance

Imagine a bank has two entrances.

```
Door 1

HTTP

Unsafe
```

```
Door 2

HTTPS

Secure
```

Whenever someone enters through the unsafe door,

the security guard says:

> **"Please use the secure entrance."**

The visitor is redirected.

That's exactly how HTTPS Redirection works.

---

# Another Example: Hospital

Suppose a hospital has:

```
Old Entrance

↓

Closed
```

Everyone arriving there is redirected to

```
New Secure Entrance
```

Nobody is allowed to use the old entrance.

---

# Why Do We Need HTTPS?

Suppose you log in to a website.

Without HTTPS

```
Username

Password

↓

Internet

↓

Anyone Can Read
```

An attacker on the same network may intercept the data.

---

With HTTPS

```
Username

Password

↓

Encrypted

↓

Internet

↓

Website
```

Only the browser and server can read the data.

---

# What Happens Without HTTPS Redirection?

User types

```
http://mybank.com
```

Request

```
Browser

↓

HTTP

↓

Website
```

The connection is not encrypted.

---

# What Happens With HTTPS Redirection?

User types

```
http://mybank.com
```

Pipeline

```
Browser

↓

HTTP

↓

HTTPS Redirection Middleware

↓

Redirect

↓

HTTPS

↓

Controller
```

The request is processed securely.

---

# Enable HTTPS Redirection

```csharp
app.UseHttpsRedirection();
```

That's all.

---

# Complete Example

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.UseHttpsRedirection();

app.MapControllers();

app.Run();
```

---

# Request Flow

```
Browser

↓

HTTP Request

↓

HTTPS Redirection Middleware

↓

Redirect

↓

HTTPS Request

↓

Controller

↓

Response
```

---

# Browser Example

User enters

```
http://localhost:5000/products
```

Server responds

```
307 Temporary Redirect
```

Browser automatically requests

```
https://localhost:5001/products
```

The user usually doesn't notice the redirect.

---

# Middleware Pipeline

```
Browser
    │
    ▼
HTTPS Redirection Middleware
    │
    ▼
HSTS Middleware
    │
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
```

A common production order is:

```csharp
app.UseHsts();

app.UseHttpsRedirection();

app.UseRouting();
```

> **Note:** HSTS is typically enabled only in production.

---

# What HTTP Status Code Is Used?

HTTPS Redirection commonly returns:

```
307 Temporary Redirect
```

or

```
308 Permanent Redirect
```

depending on configuration and framework behavior.

---

# What Is the Difference Between 307 and 308?

| Status Code | Meaning |
|--------------|----------|
| **307 Temporary Redirect** | Redirect is temporary. The browser preserves the HTTP method (GET, POST, etc.). |
| **308 Permanent Redirect** | Redirect is permanent. The browser also preserves the HTTP method. |

Unlike older redirects (301/302), both 307 and 308 preserve the original HTTP method and request body.

---

# HTTPS Redirection vs HSTS

This is one of the most frequently asked interview questions.

| HTTPS Redirection | HSTS |
|-------------------|------|
| Redirects an HTTP request to HTTPS | Tells the browser to always use HTTPS in the future |
| Happens on the server | Enforced by the browser after receiving the HSTS header |
| Requires the first HTTP request | Avoids future HTTP requests once the policy is stored |
| Uses redirect status codes (307/308) | Uses the `Strict-Transport-Security` header |

---

# Request Timeline

## First Visit

```
Browser

↓

HTTP

↓

Redirect

↓

HTTPS
```

---

## Later Visits (with HSTS)

```
Browser

↓

HTTPS Directly
```

No HTTP request is sent.

---

# Real-World Example

Online Banking

User enters

```
http://bank.com
```

Without HTTPS Redirection

```
HTTP

↓

Login Page

↓

Unsafe
```

---

With HTTPS Redirection

```
HTTP

↓

Redirect

↓

HTTPS

↓

Secure Login
```

---

# Benefits

- Encrypts communication between clients and servers.
- Protects usernames, passwords, and sensitive data.
- Helps prevent eavesdropping on unsecured networks.
- Improves user trust by using secure connections.
- Supports compliance with modern web security standards.

---

# Common Mistakes

### Forgetting `UseHttpsRedirection()`

Without it, users can continue accessing the application over HTTP.

---

### Assuming HTTPS Redirection Is the Same as HSTS

They work together but have different purposes.

- HTTPS Redirection redirects requests.
- HSTS tells browsers to stop using HTTP in future requests.

---

### Using HTTP Links Internally

Example

```html
<a href="http://example.com">
```

Use

```html
<a href="https://example.com">
```

or relative URLs whenever possible.

---

# Best Practices

- Always enable HTTPS Redirection in production.
- Use HTTPS for every endpoint, not just login pages.
- Combine HTTPS Redirection with HSTS for stronger security.
- Ensure a valid TLS/SSL certificate is installed.
- Update bookmarks, documentation, and integrations to use HTTPS URLs.

---

# Frequently Asked Interview Questions

### What does `UseHttpsRedirection()` do?

It automatically redirects incoming HTTP requests to the corresponding HTTPS endpoint before the request reaches your application logic.

---

### Why is HTTPS important?

HTTPS encrypts communication between the client and server, protecting sensitive data from interception or tampering.

---

### Does HTTPS Redirection replace HSTS?

No.

HTTPS Redirection handles the current request.

HSTS influences how the browser makes future requests.

---

### What status code is typically returned?

ASP.NET Core commonly uses:

- **307 Temporary Redirect**
- **308 Permanent Redirect**

Both preserve the original HTTP method and request body.

---

# Complete Flow

```
User Types

http://example.com
        │
        ▼
HTTPS Redirection Middleware
        │
        ▼
307 / 308 Redirect
        │
        ▼
https://example.com
        │
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
Response
```

---

# Interview Answer (2-Minute Version)

**What is HTTPS Redirection Middleware in ASP.NET Core?**

HTTPS Redirection Middleware automatically redirects incoming HTTP requests to HTTPS, ensuring that communication between the client and server is encrypted. It is enabled using `app.UseHttpsRedirection()` and typically returns a **307 Temporary Redirect** or **308 Permanent Redirect** response, preserving the original HTTP method. Unlike HSTS, which instructs browsers to always use HTTPS for future requests, HTTPS Redirection secures the current request by redirecting it to the HTTPS endpoint. In production applications, it is commonly used together with HSTS to provide comprehensive transport security.