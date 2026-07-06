# 33. HSTS (HTTP Strict Transport Security) Middleware in ASP.NET Core

HSTS Middleware is a **security feature** that forces browsers to always communicate with your website using **HTTPS** instead of **HTTP**.

It helps protect users from:

- Man-in-the-Middle (MITM) attacks
- Protocol downgrade attacks
- SSL stripping attacks

---

# What is HSTS?

## Definition

HSTS (HTTP Strict Transport Security) is a web security policy that instructs browsers:

> **"From now on, always use HTTPS when connecting to this website."**

Once a browser receives this instruction, it will automatically convert future HTTP requests into HTTPS requests.

---

# Layman Example: Bank Entrance

Imagine a bank with two entrances.

```
Front Door

(HTTPS)

Secure
```

```
Back Door

(HTTP)

Unsafe
```

The bank tells every customer:

> **"Never use the back door. Always enter through the secure front door."**

Even if a customer accidentally walks toward the back door, they immediately switch to the front door.

That's exactly what HSTS does.

---

# Another Example: School Bus

A school tells children:

```
Never walk home.

Always take the school bus.
```

After hearing this rule once,

they always use the school bus.

Similarly,

after receiving the HSTS policy,

the browser always uses HTTPS.

---

# Why Do We Need HSTS?

Suppose a user types

```
http://mybank.com
```

Without HSTS

```
Browser

↓

HTTP

↓

Attacker intercepts

↓

Reads data

↓

Forwards request
```

Sensitive information could be exposed.

---

With HSTS

```
Browser

↓

Automatically changes

↓

https://mybank.com

↓

Encrypted Communication
```

The attacker cannot force the browser back to HTTP.

---

# Without HSTS

```
Browser

↓

HTTP

↓

Server Redirects

↓

HTTPS
```

The **first HTTP request** can still be intercepted.

---

# With HSTS

Browser remembers

```
Always HTTPS
```

Future requests become

```
Browser

↓

HTTPS Directly

↓

Server
```

No insecure first request after the policy is stored.

---

# Request Flow

Without HSTS

```
Browser

↓

HTTP

↓

Redirect

↓

HTTPS

↓

Website
```

---

With HSTS

```
Browser

↓

HTTPS Directly

↓

Website
```

---

# Enable HSTS

```csharp
app.UseHsts();
```

That's all.

---

# Typical Production Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}

app.UseHttpsRedirection();

app.MapControllers();

app.Run();
```

---

# Why Not in Development?

During development,

you may not always use trusted HTTPS certificates.

Enabling HSTS can make local testing inconvenient because the browser remembers the HTTPS requirement.

Therefore,

HSTS is typically enabled only in **Production**.

---

# Difference Between HTTPS Redirection and HSTS

This is a very common interview question.

## HTTPS Redirection

```csharp
app.UseHttpsRedirection();
```

Behavior

```
HTTP Request

↓

301/307 Redirect

↓

HTTPS
```

Every HTTP request is redirected.

---

## HSTS

```csharp
app.UseHsts();
```

Behavior

```
Browser remembers

↓

Always use HTTPS

↓

No HTTP request sent
```

---

# Comparison

| HTTPS Redirection | HSTS |
|-------------------|------|
| Redirects HTTP requests to HTTPS | Instructs browser to always use HTTPS |
| Works on every HTTP request | Browser remembers the policy |
| First request may still use HTTP | Future requests use HTTPS directly |
| Redirect happens on the server | Decision happens in the browser |

---

# HSTS Response Header

When enabled,

the server sends:

```
Strict-Transport-Security

max-age=31536000
```

Meaning

```
Remember HTTPS

for

1 year
```

---

# Example Header

```
Strict-Transport-Security:
max-age=31536000;
includeSubDomains
```

Meaning

- Use HTTPS for one year.
- Apply the rule to all subdomains.

---

# Common HSTS Options

```csharp
builder.Services.AddHsts(options =>
{
    options.MaxAge = TimeSpan.FromDays(365);

    options.IncludeSubDomains = true;

    options.Preload = true;
});
```

---

## MaxAge

```
365 Days
```

Browser remembers HTTPS for one year.

---

## IncludeSubDomains

Protects

```
example.com

api.example.com

admin.example.com

shop.example.com
```

---

## Preload

Allows the domain to be submitted to browser preload lists.

Many modern browsers ship with a built-in list of HSTS-enabled websites, meaning they use HTTPS **even on the very first visit** once the domain is accepted into the preload list.

---

# Middleware Pipeline

```
Browser
    │
    ▼
HSTS Middleware
    │
    ▼
HTTPS Redirection
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

---

# Real-World Example

Online Banking

```
User

↓

http://bank.com
```

Without HSTS

```
HTTP

↓

Possible Attack

↓

Redirect

↓

HTTPS
```

---

With HSTS

```
Browser

↓

HTTPS Directly

↓

Secure Banking
```

---

# Benefits

- Prevents protocol downgrade attacks.
- Protects against SSL stripping attacks.
- Forces secure communication.
- Improves overall website security.
- Works automatically after the browser learns the HSTS policy.

---

# Common Mistakes

### Enabling HSTS in Development

Not recommended.

Use HSTS primarily in Production.

---

### Assuming HSTS Replaces HTTPS Redirection

It does not.

A common production setup is:

```csharp
app.UseHsts();

app.UseHttpsRedirection();
```

---

### Setting an Extremely Long Max-Age Without Testing

Browsers remember the policy.

If HTTPS is misconfigured later, users may be unable to access the site until the HSTS policy expires.

---

# Best Practices

- Enable HSTS only in Production.
- Use `UseHttpsRedirection()` together with `UseHsts()`.
- Set an appropriate `max-age` after validating your HTTPS configuration.
- Use `IncludeSubDomains` only if every subdomain supports HTTPS.
- Consider HSTS preload only after meeting all preload requirements.

---

# Frequently Asked Interview Questions

### What is HSTS?

HSTS (HTTP Strict Transport Security) is a security policy that instructs browsers to always use HTTPS when communicating with a website.

---

### What does `UseHsts()` do?

It adds the `Strict-Transport-Security` response header so that supported browsers remember to use HTTPS for future requests.

---

### Does HSTS replace HTTPS Redirection?

No.

`UseHttpsRedirection()` redirects incoming HTTP requests.

`UseHsts()` tells browsers to avoid HTTP in future requests.

They complement each other.

---

### Why isn't HSTS usually enabled in Development?

Because browsers cache the HSTS policy, which can interfere with local development and testing, especially when using self-signed certificates or changing ports.

---

# Complete Flow

```
User Types

http://example.com
        │
        ▼
Server Sends
Strict-Transport-Security Header
        │
        ▼
Browser Stores Policy
        │
        ▼
Future Requests
        │
        ▼
https://example.com
        │
        ▼
Secure Connection
```

---

# Interview Answer (2-Minute Version)

**What is HSTS Middleware in ASP.NET Core?**

HSTS (HTTP Strict Transport Security) Middleware enhances web application security by instructing browsers to always communicate with the application over HTTPS. It does this by sending the `Strict-Transport-Security` response header, which browsers cache for a specified duration. Unlike `UseHttpsRedirection()`, which redirects HTTP requests to HTTPS, HSTS prevents future HTTP requests by making the browser automatically use HTTPS. HSTS is typically enabled only in production environments and is commonly used together with HTTPS redirection to protect against protocol downgrade and SSL stripping attacks.