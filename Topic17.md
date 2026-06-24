# Authentication & Authorization — JWT (JSON Web Token)

JWT is one of the most widely used authentication mechanisms in modern Web APIs, especially in ASP.NET Core and microservices.

---

# 1. Core Idea (Layman View)

Think of JWT like a **digital boarding pass at an airport**.

- You login once → you get a token (boarding pass)
- You show that token every time you travel (API request)
- Server checks the token instead of asking you to login again

---

# 2. Authentication vs Authorization

## Authentication = Who are you?

Example:

```text
Login with username + password → system verifies identity
```

---

## Authorization = What are you allowed to do?

Example:

```text
Admin can delete users
Normal user cannot
```

---

# 3. What is JWT?

JWT is a **self-contained token** that stores user information securely.

It is:

- Compact
- Digitally signed
- Stateless (server does NOT store session)

---

# 4. JWT Structure

A JWT has 3 parts:

```text
Header.Payload.Signature
```

Example:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJ1c2VySWQiOjEsInJvbGUiOiJBZG1pbiJ9
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

---

# 5. JWT Parts Explained

---

## 1. Header

Contains metadata:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

## 2. Payload (Claims)

Contains user data:

```json
{
  "userId": 1,
  "name": "John",
  "role": "Admin",
  "exp": 1710000000
}
```

---

## 3. Signature

Used to verify token is NOT tampered.

```text
HMACSHA256(header + payload + secretKey)
```

---

# 6. JWT Authentication Flow

## Step-by-step:

### 1. User logs in

```http
POST /login
```

```json
{
  "username": "john",
  "password": "12345"
}
```

---

### 2. Server validates credentials

If correct → generate JWT

---

### 3. Server returns token

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

---

### 4. Client stores token

- LocalStorage (web)
- Secure storage (mobile)

---

### 5. Client sends token in every request

```http
GET /api/orders
Authorization: Bearer <token>
```

---

### 6. Server validates token

- checks signature
- checks expiry
- extracts user info

---

# 7. Stateless Nature of JWT

Server does NOT store session.

```text
No session table
No server memory dependency
```

Each request is independent.

---

# 8. ASP.NET Core JWT Setup

---

## Step 1: Install package

```text
Microsoft.AspNetCore.Authentication.JwtBearer
```

---

## Step 2: Configure Authentication

```csharp
builder.Services.AddAuthentication("Bearer")
.AddJwtBearer("Bearer", options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,

        ValidIssuer = "MyApp",
        ValidAudience = "MyUsers",
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes("super_secret_key"))
    };
});
```

---

## Step 3: Enable middleware

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

---

## Step 4: Protect API

```csharp
[Authorize]
[HttpGet("orders")]
public IActionResult GetOrders()
{
    return Ok();
}
```

---

## Step 5: Role-based access

```csharp
[Authorize(Roles = "Admin")]
[HttpDelete("users/{id}")]
public IActionResult DeleteUser(int id)
{
    return Ok();
}
```

---

# 9. Why JWT is Popular

## ✔ Stateless

No server session storage needed

---

## ✔ Scalable

Works easily in microservices

---

## ✔ Fast

No database lookup required for every request

---

## ✔ Cross-platform

Works in:

- Web apps
- Mobile apps
- APIs
- Microservices

---

# 10. JWT vs Session Authentication

| Feature | JWT | Session |
|----------|-----|---------|
| Storage | Client | Server |
| Scalability | High | Low |
| Performance | Fast | Slower |
| Stateless | Yes | No |
| Microservices | Excellent | Poor |

---

# 11. Token Expiration

JWT includes expiry:

```json
"exp": 1710000000
```

---

## Why expiry is important:

- prevents misuse of stolen tokens
- forces re-login
- improves security

---

# 12. Refresh Tokens (Advanced Concept)

## Problem:

JWT expires → user must login again

---

## Solution:

Use Refresh Token:

```text
Access Token (short life)
Refresh Token (long life)
```

---

## Flow:

```text
Login → get Access + Refresh token
Access token expires
Use refresh token → get new access token
```

---

# 13. Security Best Practices

---

## ❌ Never store JWT in localStorage (if possible)

Risk: XSS attack

---

## ✔ Prefer:

- HttpOnly cookies (web apps)
- Secure storage (mobile apps)

---

## ❌ Do NOT put sensitive data in JWT

Bad:

```json
{
  "password": "12345"
}
```

---

## ✔ Good:

```json
{
  "userId": 1,
  "role": "Admin"
}
```

---

## ✔ Always use HTTPS

JWT can be intercepted if not encrypted in transit.

---

# 14. Common JWT Attacks

---

## 1. Token Theft

If token is stolen → attacker impersonates user

---

## 2. Token Replay

Old token reused after expiry

---

## 3. Weak Secret Key

If secret is weak → token can be forged

---

# 15. Real Production Flow

```text
Client
  ↓
Login API
  ↓
JWT Token Generated
  ↓
Client stores token
  ↓
Every API call:
    Authorization: Bearer token
  ↓
API Gateway / Middleware
  ↓
JWT Validation
  ↓
Authorization Check
  ↓
Business Logic
  ↓
Response
```

---

# 16. Common Mistakes

---

## ❌ 1. Not validating token expiry

Leads to security risk

---

## ❌ 2. Using weak signing key

```text
"12345"
```

Easily cracked

---

## ❌ 3. Storing too much data in JWT

Increases token size

---

## ❌ 4. Not using HTTPS

Token can be intercepted

---

# 17. Senior-Level Interview Answer

JWT authentication is a stateless authentication mechanism where the server issues a digitally signed token after successful login. This token contains user claims such as user ID and role, and is sent by the client in every request via the Authorization header. The server validates the token signature and expiry without storing session state. JWT is widely used in modern web APIs and microservices due to its scalability, performance, and cross-platform compatibility. Security best practices include using strong signing keys, enforcing token expiration, using HTTPS, and avoiding sensitive data inside the token.

---

# 18. Easy Way to Remember

```text
Login → Get Token → Send Token → Verify Token → Allow Access
```

Or:

```text
JWT = Digital ID card for APIs
```