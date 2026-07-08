# 101. Refresh Tokens

---

# What is a Refresh Token?

A **Refresh Token** is a long-lived credential used to obtain a **new Access Token** when the current Access Token expires.

Instead of asking the user to log in again, the application silently requests a new Access Token using the Refresh Token.

---

# Simple Definition

Think of two cards:

- **Access Token** → Short-term entry pass
- **Refresh Token** → Card that lets you get a new entry pass

---

# Real-Life Analogy

Imagine working in an office.

You receive:

- A **Visitor Pass** valid for **1 hour**
- An **Employee ID** valid for **1 year**

When the visitor pass expires,

you show your employee ID to get a new visitor pass.

You don't register again.

```
Employee ID

↓

Get New Visitor Pass

↓

Continue Working
```

The Employee ID is the **Refresh Token**.

The Visitor Pass is the **Access Token**.

---

# Why Do We Need Refresh Tokens?

Suppose an Access Token expires every **15 minutes**.

Without Refresh Tokens

```
15 Minutes

↓

Token Expires

↓

User Logs In Again
```

Poor user experience.

With Refresh Tokens

```
15 Minutes

↓

Access Token Expires

↓

Refresh Token Used

↓

New Access Token

↓

Continue Working
```

The user usually doesn't notice this happening.

---

# Authentication Flow

```
User Login

↓

Username + Password

↓

Server

↓

Access Token (15 min)

+

Refresh Token (30 days)

↓

Client Stores Both

↓

Access Token Expires

↓

Refresh Token

↓

Server

↓

New Access Token
```

---

# Example Response

```json
{
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "a8f7c91d7b5e42..."
}
```

---

# Using the Refresh Token

Request

```http
POST /refresh-token
```

Body

```json
{
    "refreshToken":"a8f7c91d7b5e42..."
}
```

Response

```json
{
    "accessToken":"newJwtToken"
}
```

---

# ASP.NET Core Example

Generate Refresh Token

```csharp
var refreshToken = Guid.NewGuid().ToString();
```

Store it securely in the database.

---

Validate Refresh Token

```csharp
var token = db.RefreshTokens
              .FirstOrDefault(x => x.Token == request.Token);
```

If valid,

issue a new JWT.

---

# Where Should Refresh Tokens Be Stored?

Server

- Database (recommended)
- Encrypted
- Associated with a user
- Includes expiration date
- Can be revoked

Client

- Secure, HTTP-only cookies (recommended for browsers)
- Platform-secure storage for mobile apps

Avoid storing long-lived refresh tokens in browser local storage.

---

# Best Practices

- Rotate refresh tokens after use.
- Revoke refresh tokens on logout.
- Store them securely.
- Set reasonable expiration times.
- Detect token reuse if rotation is implemented.

---

# Advantages

- Better user experience
- Short-lived Access Tokens
- Improved security
- Supports long-lived sessions

---

# Interview Questions

## What is a Refresh Token?

A Refresh Token is a long-lived credential used to obtain a new Access Token after the current one expires without requiring the user to log in again.

---

## Why not make the Access Token long-lived?

Long-lived Access Tokens increase the security risk if they are stolen. Short-lived Access Tokens limit the impact of token theft.

---

# Quick Revision

- Refresh Token gets a new Access Token.
- Access Token authenticates API requests.
- Refresh Tokens are long-lived and should be stored securely.
- Rotate and revoke Refresh Tokens for better security.

---

# 102. Token Expiration

---

# What is Token Expiration?

Every JWT should have an **expiration time**.

After that time,

the token is no longer valid.

This prevents stolen tokens from being usable indefinitely.

---

# Real-Life Analogy

Imagine a movie ticket.

```
Valid

10:00 AM

↓

1:00 PM

↓

Expired
```

After the movie starts,

the ticket is no longer accepted.

JWTs work the same way.

---

# Why Token Expiration?

Without expiration,

a stolen JWT could be used forever.

With expiration,

its lifetime is limited.

---

# JWT Expiration Claim

JWT uses the **exp** (Expiration) claim.

Example

```json
{
    "sub":"101",
    "exp":1730000000
}
```

The `exp` value is a Unix timestamp indicating when the token expires.

---

# Token Lifecycle

```
Generate JWT

↓

Valid

↓

Expires

↓

Refresh Token

↓

New JWT
```

---

# ASP.NET Core Example

Create JWT

```csharp
var token = new JwtSecurityToken(
    expires: DateTime.UtcNow.AddMinutes(15)
);
```

---

Validate Expiration

```csharp
ValidateLifetime = true;
```

---

# What Happens After Expiration?

Request

```
Authorization:

Bearer expired_token
```

Server

↓

401 Unauthorized

The client can then use the Refresh Token (if available) to obtain a new Access Token.

---

# Recommended Expiration Times

Access Token

```
10–30 Minutes
```

Refresh Token

```
Days or Weeks
```

The exact values depend on your application's security requirements.

---

# Benefits

- Limits damage from stolen tokens
- Improves security
- Supports secure session management
- Encourages token rotation

---

# Interview Questions

## Why should JWTs expire?

To reduce the risk of stolen tokens being used indefinitely and to improve overall security.

---

## Which JWT claim stores expiration?

```
exp
```

---

# Quick Revision

- JWTs should expire.
- `exp` claim stores expiration.
- Expired tokens return **401 Unauthorized**.
- Refresh Tokens are commonly used to obtain new Access Tokens.

---

# 103. JWT Validation

---

# What is JWT Validation?

JWT Validation is the process of checking whether a JWT is:

- Authentic
- Untampered
- Not expired
- Issued by a trusted issuer
- Intended for this application

Only if all checks succeed is the user authenticated.

---

# Real-Life Analogy

Imagine boarding an airplane.

Security checks:

```
Passport

↓

Ticket

↓

Visa

↓

Expiry Date

↓

Identity Match

↓

Board Flight
```

JWT validation performs similar checks.

---

# JWT Validation Flow

```
Client

↓

JWT

↓

Server

↓

Validate Signature

↓

Validate Expiration

↓

Validate Issuer

↓

Validate Audience

↓

Read Claims

↓

Authenticated User
```

---

# What Does the Server Validate?

## 1. Signature

Checks whether the token has been modified.

If the payload changes,

the signature becomes invalid.

---

## 2. Expiration

Checks the `exp` claim.

Expired token

↓

Rejected

---

## 3. Issuer (`iss`)

Verifies the token was created by the expected authority.

Example

```
https://login.microsoftonline.com
```

---

## 4. Audience (`aud`)

Verifies the token is intended for this API.

Example

```
MyShoppingAPI
```

---

## 5. Lifetime

Ensures the token is currently valid based on its validity period.

---

# ASP.NET Core Configuration

```csharp
builder.Services
.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
.AddJwtBearer(options =>
{
    options.TokenValidationParameters =
        new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,

            ValidIssuer = "MyApp",

            ValidAudience = "MyApi",

            IssuerSigningKey =
                new SymmetricSecurityKey(
                    Encoding.UTF8.GetBytes("VerySecretKey"))
        };
});
```

---

# Request Example

```
Authorization:

Bearer eyJhbGc...
```

Middleware validates the token before the request reaches the controller.

---

# Validation Result

Valid Token

```
Signature ✔

Issuer ✔

Audience ✔

Expiration ✔

↓

Access Granted
```

Invalid Token

```
Expired

↓

401 Unauthorized
```

---

# Common Validation Errors

Expired Token

```
401 Unauthorized
```

---

Invalid Signature

```
401 Unauthorized
```

---

Wrong Issuer

```
401 Unauthorized
```

---

Wrong Audience

```
401 Unauthorized
```

---

# Why Validate JWTs?

Without validation,

anyone could create or modify a token and gain unauthorized access.

Validation ensures the token is trustworthy.

---

# JWT Validation Lifecycle

```
Client

↓

JWT

↓

Authentication Middleware

↓

Validation

↓

Claims Principal

↓

Authorization

↓

Controller
```

---

# Best Practices

- Always validate the signature.
- Validate issuer and audience.
- Validate token lifetime.
- Use HTTPS.
- Keep signing keys secure.
- Rotate signing keys periodically.
- Keep Access Tokens short-lived.

---

# Advantages

- Secure
- Stateless
- Fast
- Scalable
- Standardized

---

# JWT Validation vs Authentication

| JWT Validation | Authentication |
|----------------|----------------|
| Verifies the JWT itself | Verifies the user identity |
| Checks signature, issuer, audience, lifetime | Establishes the authenticated user |
| Runs as part of JWT Bearer authentication | Overall login/authentication process |

---

# Complete JWT Authentication Flow

```
Login

↓

Verify Credentials

↓

Generate JWT

↓

Client Stores JWT

↓

API Request

↓

JWT Validation

↓

Claims Principal Created

↓

Authorization

↓

Controller
```

---

# Interview Questions

## What is JWT Validation?

JWT Validation is the process of verifying that a token is authentic, has not been tampered with, has not expired, and was issued by a trusted issuer for the intended audience.

---

## What are the main validation checks?

- Signature
- Expiration (`exp`)
- Issuer (`iss`)
- Audience (`aud`)
- Signing Key

---

## What happens if JWT validation fails?

The request is rejected, typically with a **401 Unauthorized** response.

---

## Which middleware validates JWTs in ASP.NET Core?

The **JWT Bearer Authentication Middleware** performs JWT validation before the request reaches the controller.

---

# Quick Revision

- **Refresh Token** → Used to obtain a new Access Token.
- **Access Token** → Used to authenticate API requests.
- **Token Expiration** → Controlled by the `exp` claim.
- **JWT Validation** checks:
  - Signature
  - Expiration
  - Issuer
  - Audience
  - Signing Key
- Invalid or expired JWTs typically result in **401 Unauthorized**.
- Use short-lived Access Tokens and securely managed Refresh Tokens for a good balance of security and usability.