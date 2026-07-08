# 91. Authentication vs Authorization

---

# Introduction

One of the most frequently asked ASP.NET Core interview questions is:

> **What is the difference between Authentication and Authorization?**

Although they sound similar, they solve two completely different problems.

A simple way to remember is:

- **Authentication = Who are you?**
- **Authorization = What are you allowed to do?**

Authentication always happens **before** Authorization.

---

# Real-Life Analogy

Imagine entering an office building.

## Step 1: Authentication

The security guard asks:

```
Who are you?
```

You show your employee ID card.

The guard verifies your identity.

You are authenticated.

---

## Step 2: Authorization

Now you want to enter the Server Room.

The guard checks:

```
Are you allowed to enter?
```

If you're an IT Administrator

↓

Access Granted

If you're a Visitor

↓

Access Denied

This is authorization.

---

# Simple Diagram

```
Authentication

↓

Verify Identity

↓

Authorization

↓

Verify Permissions
```

---

# What is Authentication?

Authentication is the process of **verifying the identity of a user or application**.

The system checks:

```
Who are you?
```

Examples:

- Username & Password
- JWT Token
- OAuth Login
- Google Login
- Microsoft Login
- Fingerprint
- Face Recognition

---

# Authentication Example

You log into Gmail.

You enter

```
Email

Password
```

Google verifies them.

If correct

↓

You are authenticated.

---

# ASP.NET Core Example

```http
POST /login

{
    "username": "saurabh",
    "password": "password123"
}
```

Server verifies the credentials.

If valid

↓

Returns a JWT token.

```json
{
    "token": "eyJhbGciOi..."
}
```

Now the client includes the token in future requests.

```
Authorization: Bearer eyJhbGciOi...
```

The server verifies the token and authenticates the user.

---

# What is Authorization?

Authorization is the process of **determining what an authenticated user is allowed to do**.

The system asks:

```
What permissions do you have?
```

Examples:

- Can view reports?
- Can delete users?
- Can approve payments?
- Can access Admin Dashboard?

---

# Authorization Example

Suppose two users log in.

User A

```
Role

Admin
```

User B

```
Role

Customer
```

Both are authenticated.

Now both try to access

```
DELETE /users/5
```

Admin

↓

Allowed

Customer

↓

Forbidden

The identity is valid for both, but the permissions are different.

---

# Authentication vs Authorization Flow

```
User

↓

Login

↓

Authentication

↓

Identity Verified

↓

Authorization

↓

Permission Checked

↓

Controller Action
```

---

# ASP.NET Core Attributes

## Authentication

Configured using middleware.

```csharp
app.UseAuthentication();
```

---

## Authorization

Configured using middleware.

```csharp
app.UseAuthorization();
```

---

## Protecting an API

```csharp
[Authorize]
public IActionResult GetUsers()
{
    return Ok();
}
```

Only authenticated users can access this endpoint.

---

## Role-Based Authorization

```csharp
[Authorize(Roles = "Admin")]
public IActionResult DeleteUser()
{
    return Ok();
}
```

Only users with the **Admin** role are allowed.

---

## Multiple Roles

```csharp
[Authorize(Roles = "Admin,Manager")]
public IActionResult GetReports()
{
    return Ok();
}
```

Users with either **Admin** or **Manager** role can access the endpoint.

---

# Authentication Process

```
Username

↓

Password

↓

Identity Verification

↓

JWT Token Generated

↓

Authenticated User
```

---

# Authorization Process

```
Authenticated User

↓

Read Claims

↓

Read Roles

↓

Permission Check

↓

Allow or Deny
```

---

# Claims-Based Authorization

Instead of checking only roles,

authorization can also check claims.

Example JWT Claims

```json
{
    "sub": "101",
    "name": "Saurabh",
    "role": "Admin",
    "department": "IT"
}
```

Controller

```csharp
[Authorize(Policy = "ITOnly")]
public IActionResult GetInternalData()
{
    return Ok();
}
```

Policy checks

```
Department == IT
```

---

# Authentication Methods

Common authentication methods include:

- Username & Password
- JWT Bearer Token
- OAuth 2.0
- OpenID Connect
- API Keys
- Cookies
- Windows Authentication
- Multi-Factor Authentication (MFA)

---

# Types of Authorization

## Role-Based Authorization

```
Admin

Manager

Customer
```

---

## Claims-Based Authorization

Checks claims stored in the identity.

Examples

```
Department

Country

Subscription

EmployeeId
```

---

## Policy-Based Authorization

Policies combine one or more authorization requirements.

Example

```
Age > 18

AND

Department = HR
```

---

# Authentication vs Authorization

| Authentication | Authorization |
|----------------|---------------|
| Verifies identity | Verifies permissions |
| "Who are you?" | "What can you do?" |
| Happens first | Happens after authentication |
| Creates user identity | Uses identity to make access decisions |
| Login process | Access control process |

---

# ASP.NET Core Pipeline

```
HTTP Request

↓

UseAuthentication()

↓

Identity Verified

↓

UseAuthorization()

↓

Permission Checked

↓

Controller
```

**Important:** The middleware order matters.

Correct order:

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

If you reverse the order,

authorization won't know who the user is.

---

# Real-World Example

Suppose you're building an online banking application.

### Login

```
Username

Password
```

↓

Authentication succeeds.

---

Now the user requests:

```
Transfer ₹10,000
```

Authorization checks:

- Is the user authenticated?
- Is the account active?
- Does the user have permission?
- Is the transfer limit exceeded?

Only after these checks is the transfer allowed.

---

# Common Mistakes

## Mistake 1

Thinking Authentication and Authorization are the same.

They are different.

---

## Mistake 2

Calling

```csharp
UseAuthorization();
```

before

```csharp
UseAuthentication();
```

Incorrect order.

---

## Mistake 3

Using Roles for every permission.

For complex systems,

Claims and Policies are usually more flexible.

---

# Interview Questions

## What is Authentication?

Authentication is the process of verifying the identity of a user or application.

---

## What is Authorization?

Authorization determines what an authenticated user is allowed to access or perform.

---

## Which happens first?

Authentication always happens before Authorization.

---

## Can Authorization work without Authentication?

Generally, no.

The system must first know who the user is before it can decide what they are allowed to do.

---

## What is the difference between Role-Based and Claims-Based Authorization?

- **Role-Based Authorization** grants access based on user roles such as Admin or Manager.
- **Claims-Based Authorization** grants access based on specific user attributes (claims), such as department, subscription level, or employee ID.

---

## What middleware order should be used?

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

Authentication must execute before Authorization.

---

# Quick Revision

- **Authentication = Verify Identity (Who are you?)**
- **Authorization = Verify Permissions (What can you do?)**
- Authentication always runs before Authorization.
- Authentication creates the user identity.
- Authorization checks roles, claims, or policies.
- Protect endpoints using `[Authorize]`.
- Use `UseAuthentication()` before `UseAuthorization()` in the middleware pipeline.
- JWT, OAuth, Cookies, and API Keys are common authentication methods.
- Role-Based, Claims-Based, and Policy-Based are common authorization approaches.