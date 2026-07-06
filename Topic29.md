# 8. Authorization Middleware in ASP.NET Core

Authorization Middleware is one of the most important security components in ASP.NET Core.

It answers one simple question:

> **"What are you allowed to do?"**

Unlike Authentication, Authorization does **not** verify who the user is.

It verifies whether an **already authenticated user** has permission to access a resource.

---

# Authentication vs Authorization

This is the most commonly asked interview question.

## Authentication

Answers

> **Who are you?**

Example:

```
Username

Password

JWT Token

Cookie
```

Identity is verified.

---

## Authorization

Answers

> **What can you access?**

Example:

```
Admin

Manager

Employee

Customer
```

Permission is verified.

---

# Layman Example: Company Office

Imagine you enter a company building.

## Step 1

Security Guard

```
Who are you?
```

You show your Employee ID.

Security verifies your identity.

This is

**Authentication**

---

## Step 2

You want to enter the Server Room.

The manager asks

```
Are you an IT Administrator?
```

If Yes

```
Door Opens
```

If No

```
Access Denied
```

This is

**Authorization**

---

# Another Example: Netflix

Authentication

```
Login

Email

Password

↓

Logged In
```

Authorization

```
Can watch Premium Movies?

↓

Premium Subscription?

↓

Yes

↓

Play Movie
```

---

# Easy Way to Remember

Authentication

```
Identity
```

Authorization

```
Permission
```

---

# Request Pipeline

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
Authentication Middleware
    │
    ▼
Authorization Middleware
    │
    ▼
Controller
```

Notice

Authorization always comes **after Authentication**.

---

# Internal Flow

Suppose

```
GET /admin
```

Pipeline

```
Request

↓

Authentication

↓

User Identified

↓

Authorization

↓

Role Check

↓

Controller
```

---

# If User Is Not Logged In

```
Request

↓

Authentication

↓

Failed

↓

401 Unauthorized
```

---

# If User Is Logged In But Not Allowed

```
Authentication

↓

Success

↓

Authorization

↓

Permission Failed

↓

403 Forbidden
```

---

# Difference Between 401 and 403

| Status Code | Meaning |
|-------------|----------|
| **401 Unauthorized** | User is not authenticated (identity not verified) |
| **403 Forbidden** | User is authenticated but does not have permission |

This distinction is frequently tested in interviews.

---

# Configuring Authorization

## Step 1

Register services

```csharp
builder.Services.AddAuthorization();
```

---

## Step 2

Enable middleware

```csharp
app.UseAuthorization();
```

---

# Correct Order

```csharp
app.UseRouting();

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

---

# Wrong Order

```csharp
app.UseAuthorization();

app.UseAuthentication();
```

Authorization cannot evaluate permissions because the user's identity has not yet been established.

---

# Using `[Authorize]`

Protect an entire controller:

```csharp
[Authorize]
public class OrdersController : Controller
{
}
```

Only authenticated users can access it.

---

# Role-Based Authorization

Suppose only administrators should access an endpoint.

```csharp
[Authorize(Roles = "Admin")]
public IActionResult Dashboard()
{
    return View();
}
```

Execution

```
User

↓

Role = Admin ?

↓

Yes

↓

Dashboard

```

```
No

↓

403 Forbidden
```

---

# Multiple Roles

```csharp
[Authorize(Roles = "Admin,Manager")]
```

Either role is allowed.

---

# Policy-Based Authorization

Policies are recommended for larger applications because they centralize authorization rules.

Register a policy:

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("EmployeeOnly",
        policy => policy.RequireRole("Employee"));
});
```

Use the policy:

```csharp
[Authorize(Policy = "EmployeeOnly")]
public IActionResult EmployeePortal()
{
    return View();
}
```

---

# Claim-Based Authorization

Authorization can also be based on claims.

Suppose the JWT contains:

```
Department = IT
```

Policy

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("ITOnly",
        policy =>
            policy.RequireClaim("Department", "IT"));
});
```

Controller

```csharp
[Authorize(Policy = "ITOnly")]
```

Only users with the required claim are allowed.

---

# Real-Life Banking Example

Customer requests

```
Transfer Money
```

Pipeline

```
Browser

↓

Authentication

↓

Token Valid

↓

Authorization

↓

Transfer Permission?

↓

Yes

↓

Transfer Controller

↓

Success
```

If permission is missing

```
403 Forbidden
```

---

# Access Levels Example

Imagine a school.

```
Principal

Teacher

Student

Visitor
```

### Visitor

Can only enter reception.

### Student

Can enter classrooms.

### Teacher

Can access classrooms and staff room.

### Principal

Can access everything.

Authorization determines which areas each role can access.

---

# Authorization Based on Roles

```
Admin

↓

Users

Settings

Reports
```

```
Manager

↓

Reports

Orders
```

```
Customer

↓

Orders Only
```

Each role receives different permissions.

---

# Authentication vs Authorization

| Authentication | Authorization |
|---------------|---------------|
| Verifies identity | Verifies permissions |
| "Who are you?" | "What are you allowed to do?" |
| Creates `HttpContext.User` | Evaluates roles, policies, and claims |
| Runs first | Runs after authentication |
| Failure returns **401 Unauthorized** | Failure returns **403 Forbidden** |

---

# Common Mistakes

### Forgetting Authentication

```csharp
app.UseAuthorization();
```

Without authentication, Authorization has no authenticated user to evaluate.

---

### Wrong Middleware Order

```csharp
app.UseAuthorization();

app.UseAuthentication();
```

Always authenticate first.

---

### Using Roles Everywhere

While role-based authorization is simple, policy-based authorization is more flexible and maintainable for complex business rules.

---

# Best Practices

- Always call `UseAuthentication()` before `UseAuthorization()`.
- Prefer **policy-based authorization** for enterprise applications.
- Use **claims** for fine-grained permissions.
- Keep authorization logic out of controllers whenever possible.
- Apply the principle of least privilege—grant only the permissions users actually need.

---

# Frequently Asked Interview Questions

### What does `UseAuthorization()` do?

It evaluates authorization requirements (roles, policies, or claims) for the current request using the authenticated user stored in `HttpContext.User`.

---

### Can Authorization work without Authentication?

Generally, no. Authorization needs an authenticated identity to evaluate permissions.

---

### Why is `UseAuthorization()` placed after `UseAuthentication()`?

Because Authorization depends on the authenticated user created by Authentication Middleware.

---

### What is the difference between role-based and policy-based authorization?

- **Role-based** checks whether the user belongs to one or more roles.
- **Policy-based** allows more advanced rules using roles, claims, custom requirements, and business logic.

---

# Complete Request Flow

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
Authentication Middleware
    │
    ▼
Validate JWT/Cookie
    │
    ▼
Create Claims Principal
    │
    ▼
Authorization Middleware
    │
    ▼
Check Roles / Policies / Claims
    │
    ▼
Controller
    │
    ▼
Response
```

---

# Interview Answer (2-Minute Version)

**What is Authorization Middleware in ASP.NET Core?**

Authorization Middleware determines whether an authenticated user has permission to access a specific resource. It evaluates roles, policies, or claims associated with `HttpContext.User` and either allows the request to continue or blocks it. It must always execute after Authentication Middleware because it relies on the authenticated user's identity. If authentication fails, the response is typically **401 Unauthorized**. If authentication succeeds but the user lacks the required permission, the response is **403 Forbidden**.