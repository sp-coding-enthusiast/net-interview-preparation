# 7. Authentication Middleware in ASP.NET Core

Authentication Middleware is one of the most important middleware in ASP.NET Core.

It answers one simple question:

> **"Who are you?"**

Authentication **identifies the user**, but it does **not** decide what the user is allowed to do.

That responsibility belongs to **Authorization**.

---

# What is Authentication Middleware?

## Definition

Authentication Middleware verifies the identity of a user or application by validating credentials such as:

- Username & Password
- JWT Token
- Cookies
- OAuth Token
- OpenID Connect
- API Key (Custom)

If the credentials are valid, the user is considered **authenticated**.

---

# Layman Example: Airport Security

Imagine you arrive at an airport.

The first officer asks:

> "Can I see your passport?"

```
Passenger

↓

Passport Check

↓

Verified

↓

Proceed
```

The officer is **not** asking:

- Are you flying Business Class?
- Can you enter the VIP lounge?

They are only checking:

> **"Who are you?"**

This is **Authentication**.

---

# Authentication vs Authorization

Imagine a company office.

### Step 1

Security Guard

```
Who are you?
```

Employee ID verified.

This is **Authentication**.

---

### Step 2

Manager

```
Can you enter the Server Room?
```

Checks your role.

This is **Authorization**.

---

# Easy Way to Remember

Authentication

> **Identity**

```
Who are you?
```

Authorization

> **Permission**

```
What are you allowed to do?
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

Authentication always runs **before** Authorization.

---

# What Happens Internally?

Suppose a user calls:

```
GET /orders
```

with a JWT token.

```
Authorization: Bearer eyJhbGci...
```

Pipeline

```
Request

↓

Authentication Middleware

↓

Read JWT Token

↓

Validate Token

↓

Create User Identity

↓

Store User in HttpContext

↓

Next Middleware
```

If the token is invalid:

```
401 Unauthorized
```

---

# ASP.NET Core Configuration

## Step 1: Register Authentication

```csharp
builder.Services.AddAuthentication("Bearer")
                .AddJwtBearer("Bearer", options =>
                {
                    // JWT configuration
                });
```

This tells ASP.NET Core:

> "Use JWT Bearer authentication."

---

## Step 2: Add Middleware

```csharp
app.UseAuthentication();
```

This enables authentication for every request.

---

## Step 3: Add Authorization

```csharp
app.UseAuthorization();
```

Authorization depends on the authenticated user created by `UseAuthentication()`.

---

# Correct Middleware Order

```csharp
app.UseRouting();

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

This order is essential.

---

# Incorrect Order

```csharp
app.UseAuthorization();

app.UseAuthentication();
```

This is incorrect because Authorization executes before the user's identity has been established.

---

# Example: JWT Authentication

Client sends:

```
GET /products

Authorization: Bearer eyJhbGci...
```

Pipeline

```
Browser

↓

Authentication Middleware

↓

Validate JWT

↓

Valid?

   Yes

↓

Create User

↓

Authorization

↓

Controller

↓

Response
```

---

# If Token Is Invalid

```
Browser

↓

Authentication

↓

Invalid Token

↓

401 Unauthorized

↓

End
```

The controller is never reached.

---

# Cookies Authentication

Web applications often use cookies instead of JWTs.

```
Browser

↓

Cookie

↓

Authentication Middleware

↓

Read Cookie

↓

Validate Cookie

↓

User Logged In
```

---

# Common Authentication Types

| Authentication Type | Common Usage |
|---------------------|--------------|
| Cookie Authentication | Traditional MVC/Web Apps |
| JWT Bearer Token | REST APIs |
| OAuth 2.0 | Third-party login (Google, GitHub, Microsoft) |
| OpenID Connect | Enterprise SSO |
| Windows Authentication | Internal corporate applications |
| API Key (Custom) | Internal APIs and integrations |

---

# What Does Authentication Middleware Create?

After successful authentication, ASP.NET Core creates a **Claims Principal** and stores it in:

```csharp
HttpContext.User
```

Now every controller can access the authenticated user.

Example:

```csharp
var username = User.Identity?.Name;
```

Or check if the user is authenticated:

```csharp
if (User.Identity?.IsAuthenticated == true)
{
    // User is authenticated
}
```

---

# Claims Example

After validating the token:

```
User

↓

Claims Principal

↓

Name = John

Role = Admin

Email = john@example.com

Department = IT
```

These claims are available throughout the request.

---

# Real-Life Example: Online Banking

User wants to view account balance.

```
Browser

↓

JWT Token

↓

Authentication

↓

Token Valid?

↓

Yes

↓

User Identified

↓

Authorization

↓

Can View Balance?

↓

Yes

↓

Controller

↓

Account Balance
```

If the token is invalid:

```
401 Unauthorized
```

If the token is valid but the user lacks permission:

```
403 Forbidden
```

---

# Common Mistakes

### Forgetting to call `UseAuthentication()`

```csharp
app.UseAuthorization();
```

Without authentication middleware, no user identity is created.

---

### Wrong Middleware Order

```csharp
app.UseAuthorization();

app.UseAuthentication();
```

Authorization runs before authentication, leading to failed authorization.

---

### Assuming Authentication Checks Roles

Authentication only verifies **identity**.

Role checks belong to Authorization.

---

# Best Practices

- Always call `UseAuthentication()` before `UseAuthorization()`.
- Use HTTPS to protect credentials and tokens.
- Prefer JWT for stateless APIs.
- Prefer cookies for traditional server-rendered web applications.
- Validate token issuer, audience, signing key, and expiration.
- Store only necessary claims in tokens.

---

# Frequently Asked Interview Questions

### What does `UseAuthentication()` do?

It invokes the configured authentication handler (such as JWT or Cookies), validates the incoming credentials, and creates an authenticated user (`HttpContext.User`) if validation succeeds.

---

### What happens if authentication fails?

The user remains unauthenticated. Protected endpoints typically return **401 Unauthorized**.

---

### Can Authentication Middleware authorize users?

No.

Authentication identifies **who** the user is.

Authorization decides **what** the authenticated user can access.

---

### Why must `UseAuthentication()` come before `UseAuthorization()`?

Authorization requires an authenticated user. If authentication hasn't run yet, Authorization has no identity to evaluate.

---

# Authentication vs Authorization

| Authentication | Authorization |
|---------------|---------------|
| Verifies identity | Verifies permissions |
| "Who are you?" | "What can you access?" |
| Creates `HttpContext.User` | Evaluates roles, policies, or claims |
| Returns **401 Unauthorized** when credentials are missing/invalid | Returns **403 Forbidden** when the user lacks permission |

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
Controller
    │
    ▼
Response
```

---

# Interview Answer (2-Minute Version)

**What is Authentication Middleware in ASP.NET Core?**

Authentication Middleware verifies the identity of the incoming user or application by validating credentials such as JWT tokens, cookies, OAuth tokens, or other authentication mechanisms. If validation succeeds, ASP.NET Core creates a `ClaimsPrincipal` and stores it in `HttpContext.User`. Authentication answers the question **"Who are you?"** and must execute before Authorization Middleware, which uses the authenticated identity to determine whether the user has permission to access a resource.