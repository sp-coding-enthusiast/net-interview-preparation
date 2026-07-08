# 104. Multiple Authentication Schemes in ASP.NET Core

---

# What are Multiple Authentication Schemes?

A **Multiple Authentication Scheme** means an ASP.NET Core application supports **more than one way of authenticating users**.

Instead of allowing only one authentication method (like JWT), the application can support multiple methods simultaneously.

For example:

- JWT Authentication
- Cookie Authentication
- Google Login
- Microsoft Entra ID
- API Key Authentication

The application decides which scheme should authenticate each request.

---

# Layman Explanation

Imagine a company building.

Employees can enter using:

- Employee ID Card
- Fingerprint
- Face Recognition

Visitors can enter using:

- Visitor Pass

VIPs can enter using:

- QR Code

The security guard accepts multiple identification methods.

Similarly, ASP.NET Core can accept multiple authentication schemes.

---

# Why Do We Need Multiple Authentication Schemes?

Consider an e-commerce application.

```
Website

↓

Cookie Authentication

---------------

Mobile App

↓

JWT Authentication

---------------

Admin Portal

↓

Microsoft Login

---------------

Partner APIs

↓

API Key
```

Instead of creating four separate applications,

one ASP.NET Core application can support all of them.

---

# Real-World Example

Suppose your application has:

```
MVC Website

↓

Cookie Authentication

---------------

React SPA

↓

JWT Authentication

---------------

Google Login

↓

OpenID Connect

---------------

Internal APIs

↓

Microsoft Entra ID
```

Each client uses the authentication method that best fits it.

---

# Authentication Flow

```
Request

↓

Authentication Middleware

↓

Which Scheme?

↓

Cookie

OR

JWT

OR

Google

OR

Microsoft

↓

Authenticate User

↓

Authorization

↓

Controller
```

---

# Authentication Schemes in ASP.NET Core

Common authentication schemes include:

| Scheme | Used For |
|----------|-----------|
| Cookie | MVC/Web Applications |
| JWT Bearer | REST APIs |
| OpenID Connect | Google, Microsoft, Okta Login |
| OAuth | Third-party authorization |
| API Key (Custom) | Internal APIs |
| Windows Authentication | Intranet Applications |

---

# Configuring Multiple Schemes

```csharp
builder.Services
.AddAuthentication()
.AddCookie("Cookies")
.AddJwtBearer("Jwt", options =>
{
    // JWT configuration
});
```

Now the application supports both:

- Cookie Authentication
- JWT Authentication

---

# Using a Specific Scheme

Suppose an MVC controller should only use Cookies.

```csharp
[Authorize(AuthenticationSchemes = "Cookies")]
public IActionResult Dashboard()
{
    return View();
}
```

---

API Controller

```csharp
[Authorize(AuthenticationSchemes = "Jwt")]
public IActionResult GetProducts()
{
    return Ok();
}
```

Only JWT tokens are accepted.

---

# Multiple Schemes Together

You can allow multiple schemes.

```csharp
[Authorize(AuthenticationSchemes = "Cookies,Jwt")]
public IActionResult Profile()
{
    return Ok();
}
```

A user authenticated by **either Cookies or JWT** can access the endpoint.

---

# Default Authentication Scheme

Instead of specifying the scheme on every controller,

configure a default.

```csharp
builder.Services
.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme =
        JwtBearerDefaults.AuthenticationScheme;

    options.DefaultChallengeScheme =
        JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer();
```

Now JWT is used automatically unless another scheme is explicitly specified.

---

# Common Scenario 1

## MVC + API

```
Browser

↓

Cookie Authentication

---------------

Mobile App

↓

JWT Authentication
```

Both use the same ASP.NET Core application.

---

# Common Scenario 2

## Internal Employees + Customers

Employees

↓

Microsoft Entra ID

Customers

↓

Identity + JWT

---

# Common Scenario 3

## External Login

Application

↓

Google Login

↓

OpenID Connect

↓

Cookie Authentication

Here, OpenID Connect authenticates the user with Google, and the application typically creates a local authentication cookie afterward.

---

# Request Flow Example

Browser Request

```
Cookie

↓

Cookie Authentication

↓

Authenticated
```

---

Mobile Request

```
Authorization:

Bearer eyJhbGc...

↓

JWT Authentication

↓

Authenticated
```

---

# Authentication Pipeline

```
Incoming Request

↓

Authentication Middleware

↓

Select Scheme

↓

Validate Credentials

↓

Create Claims Principal

↓

Authorization

↓

Controller
```

---

# AuthenticationScheme Attribute

```csharp
[Authorize(AuthenticationSchemes =
JwtBearerDefaults.AuthenticationScheme)]
```

Or

```csharp
[Authorize(AuthenticationSchemes =
CookieAuthenticationDefaults.AuthenticationScheme)]
```

Using the framework constants helps avoid typos in scheme names.

---

# Policy-Based Authentication Scheme

Policies can require a specific authentication scheme.

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("ApiPolicy", policy =>
    {
        policy.AuthenticationSchemes.Add("Jwt");
        policy.RequireAuthenticatedUser();
    });
});
```

Use it

```csharp
[Authorize(Policy = "ApiPolicy")]
public IActionResult GetOrders()
{
    return Ok();
}
```

---

# Advantages

- Supports multiple client types
- Flexible authentication
- Single application
- Easier migration
- Enterprise-ready

---

# Disadvantages

- More configuration
- More testing required
- Increased maintenance
- Can become confusing if schemes are not clearly separated

---

# Best Practices

- Use Cookies for MVC applications.
- Use JWT Bearer for REST APIs.
- Use OpenID Connect for external identity providers.
- Keep a clear default authentication scheme.
- Prefer policy-based authorization for complex scenarios.
- Avoid unnecessary authentication schemes.

---

# Common Interview Scenario

Suppose an interviewer asks:

**"Your application has an MVC website and a mobile application. Which authentication schemes would you use?"**

Answer:

```
MVC Website

↓

Cookie Authentication

---------------

Mobile App

↓

JWT Bearer Authentication

---------------

Same ASP.NET Core Backend
```

This is one of the most common real-world architectures.

---

# Multiple Authentication Example

```
                Client
                  │
     ┌────────────┼─────────────┐
     ▼            ▼             ▼
 Browser      Mobile App     Google Login
     │            │             │
 Cookie        JWT Token     OpenID Connect
     │            │             │
     └────────────┼─────────────┘
                  ▼
      ASP.NET Core Authentication
                  │
                  ▼
            Authorization
                  │
                  ▼
              Controller
```

---

# Authentication Schemes Comparison

| Scheme | Best For | Stateless | Browser Friendly |
|----------|----------|-----------|------------------|
| Cookie | MVC Apps | Usually No | Excellent |
| JWT | REST APIs | Yes | Good |
| OpenID Connect | External Login | Depends | Excellent |
| OAuth 2.0 | Delegated Authorization | Depends | Excellent |
| API Key | Machine-to-Machine APIs | Yes | Limited |

---

# Interview Questions

## What are Multiple Authentication Schemes?

Multiple Authentication Schemes allow an ASP.NET Core application to support more than one authentication mechanism, such as Cookies, JWT Bearer, and OpenID Connect, at the same time.

---

## Why use Multiple Authentication Schemes?

To support different types of clients, such as browsers, mobile apps, partner systems, and external identity providers, within the same application.

---

## Can one application use both JWT and Cookies?

Yes.

This is a common approach where:

- Browser users authenticate with Cookies.
- API and mobile clients authenticate with JWT Bearer tokens.

---

## How do you specify which authentication scheme an endpoint should use?

Using the `AuthenticationSchemes` property of the `[Authorize]` attribute.

Example:

```csharp
[Authorize(AuthenticationSchemes =
JwtBearerDefaults.AuthenticationScheme)]
```

---

## Can an endpoint accept multiple authentication schemes?

Yes.

Specify multiple scheme names separated by commas.

```csharp
[Authorize(AuthenticationSchemes = "Cookies,Jwt")]
```

A user authenticated by either scheme can access the endpoint.

---

# Quick Revision

- Multiple Authentication Schemes allow one application to support multiple authentication methods.
- Common schemes include Cookie Authentication, JWT Bearer, OpenID Connect, OAuth, API Keys, and Windows Authentication.
- Use Cookies for MVC applications.
- Use JWT Bearer for REST APIs and mobile apps.
- Use OpenID Connect for external identity providers.
- Use the `AuthenticationSchemes` property or authorization policies to choose which scheme protects an endpoint.
- One application can securely support browsers, mobile apps, and external login providers simultaneously.