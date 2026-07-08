# 92. JWT (JSON Web Token)

---

# What is JWT?

**JWT (JSON Web Token)** is a compact, secure, URL-safe token used to **authenticate users and securely exchange information** between a client and a server.

Instead of sending the username and password with every request, the client sends a JWT.

---

# Simple Definition

JWT is like a **digital identity card**.

Once the server verifies who you are, it issues a token.

For future requests, you show the token instead of logging in again.

---

# Real-Life Analogy

Imagine entering an amusement park.

Step 1

You buy a ticket.

↓

The staff verifies your payment.

↓

You receive a wristband.

Now, every ride checks only your wristband.

Nobody asks you to pay again.

The wristband is your **JWT**.

---

# JWT Authentication Flow

```
User

↓

Login

↓

Username + Password

↓

Server Verifies

↓

JWT Token Created

↓

Client Stores Token

↓

Future Requests

↓

Authorization: Bearer <JWT>

↓

Server Validates Token

↓

Access Granted
```

---

# JWT Structure

A JWT has **three parts**.

```
Header

.

Payload

.

Signature
```

Example

```
xxxxx.yyyyy.zzzzz
```

Each part is Base64Url encoded.

---

# Header

Contains metadata.

Example

```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```

---

# Payload

Contains claims.

Example

```json
{
    "sub": "101",
    "name": "Saurabh",
    "role": "Admin"
}
```

Common Claims

- sub (Subject)
- name
- email
- role
- exp (Expiration)
- iss (Issuer)
- aud (Audience)

---

# Signature

Created using

```
Header

+

Payload

+

Secret Key
```

The signature prevents tampering.

If anyone modifies the payload,

the signature becomes invalid.

---

# JWT Example

Request

```http
POST /login
```

Response

```json
{
    "token":"eyJhbGciOiJIUzI1NiIs..."
}
```

Future request

```http
GET /users

Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

---

# ASP.NET Core JWT Authentication

Register JWT authentication.

```csharp
builder.Services
.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true
    };
});
```

Protect API

```csharp
[Authorize]
public IActionResult GetUsers()
{
    return Ok();
}
```

---

# Advantages

- Stateless
- Fast
- Cross-platform
- Scalable
- Works well for Web APIs and mobile apps

---

# Disadvantages

- Difficult to revoke before expiration
- Larger than session IDs
- Payload is encoded, not encrypted

---

# Interview Questions

## What is JWT?

A JWT is a digitally signed token used for authentication and secure information exchange.

---

## Why is JWT stateless?

Because the server does not store session data. The token itself contains the information needed to identify the user.

---

# Quick Revision

- JWT = Header + Payload + Signature.
- Sent in the `Authorization: Bearer` header.
- Commonly used for Web APIs.
- Stateless authentication mechanism.

---

# 93. OAuth 2.0

---

# What is OAuth 2.0?

**OAuth 2.0** is an **authorization framework**.

It allows one application to access another application's resources **without sharing the user's password**.

---

# Important

OAuth 2.0 is **NOT an authentication protocol**.

It is primarily used for **authorization**.

---

# Real-Life Analogy

Imagine you hire a valet driver.

You don't give the valet your house keys.

You only give your **car key**.

The valet can:

- Drive your car

But cannot:

- Enter your house

OAuth works similarly.

Applications receive limited access.

---

# Example

You visit Canva.

Instead of creating a new account,

you click

```
Continue with Google
```

Google asks:

```
Allow Canva to access:

✓ Email

✓ Profile
```

You click

```
Allow
```

Google sends Canva an **Access Token**.

Canva never sees your password.

---

# OAuth Roles

```
Resource Owner

↓

User

---------------

Client

↓

Application

---------------

Authorization Server

↓

Google

---------------

Resource Server

↓

Google API
```

---

# OAuth Flow

```
User

↓

Login with Google

↓

Google Login

↓

Permission Screen

↓

Access Token

↓

Client

↓

Google API
```

---

# Access Token

Example

```
eyJhbGc...
```

The access token proves that permission has been granted.

---

# Common Grant Types

Modern applications primarily use:

- Authorization Code Flow (with PKCE for public clients)
- Client Credentials Flow
- Refresh Token Flow

*(Older flows like the Implicit Flow are no longer recommended for new applications.)*

---

# Advantages

- No password sharing
- Secure
- Limited permissions
- Widely supported
- Industry standard

---

# Interview Questions

## Is OAuth authentication?

No.

OAuth is an authorization framework.

Authentication is usually handled by OpenID Connect.

---

# Quick Revision

- OAuth grants permission.
- Does not expose user passwords.
- Uses Access Tokens.
- Common for Google, Microsoft, Facebook, and GitHub integrations.

---

# 94. OpenID Connect (OIDC)

---

# What is OpenID Connect?

**OpenID Connect (OIDC)** is an **authentication protocol** built on top of OAuth 2.0.

If OAuth answers

```
Can this application access your data?
```

OpenID Connect answers

```
Who is the user?
```

---

# Real-Life Analogy

Imagine entering a company.

OAuth

```
Can you enter the parking lot?
```

OIDC

```
Who are you?
```

One grants access.

The other verifies identity.

---

# Why Was OpenID Connect Created?

OAuth tells applications

```
Permission Granted
```

But it doesn't identify the user.

OIDC adds an **ID Token** containing identity information.

---

# OIDC Tokens

Access Token

↓

Used to access APIs.

ID Token

↓

Contains user identity.

Example

```json
{
    "name":"Saurabh",
    "email":"abc@gmail.com"
}
```

---

# Login with Google

```
User

↓

Google Login

↓

ID Token

↓

Application Knows User Identity
```

---

# OAuth vs OpenID Connect

| OAuth | OpenID Connect |
|---------|----------------|
| Authorization | Authentication |
| Access Token | ID Token + Access Token |
| Grants permission | Verifies identity |

---

# Advantages

- Single Sign-On (SSO)
- Secure login
- Standard protocol
- Works with OAuth 2.0
- Provides identity information

---

# Interview Questions

## What is OpenID Connect?

OpenID Connect is an authentication protocol built on top of OAuth 2.0 that provides user identity using an ID Token.

---

## What token contains user identity?

The **ID Token**.

---

# Quick Revision

- OIDC = Authentication.
- Built on OAuth 2.0.
- Uses an ID Token.
- Commonly used for "Login with Google" and Microsoft Entra ID sign-in.

---

# 95. Cookie Authentication

---

# What is Cookie Authentication?

Cookie Authentication is a mechanism where the server stores the user's identity in an **encrypted authentication cookie** after login.

The browser automatically sends the cookie with every request.

---

# Real-Life Analogy

Imagine joining a club.

After showing your membership card,

the receptionist gives you a stamp on your hand.

Each time you enter,

the guard looks at the stamp.

You don't need to show your ID again.

The stamp is the authentication cookie.

---

# Cookie Authentication Flow

```
User

↓

Login

↓

Username + Password

↓

Server Validates

↓

Authentication Cookie

↓

Browser Stores Cookie

↓

Future Requests

↓

Cookie Sent Automatically

↓

Server Validates Cookie

↓

Access Granted
```

---

# Login Example

```http
POST /login
```

Server Response

```
Set-Cookie:

AuthCookie=abc123
```

Browser stores it automatically.

---

# Future Request

```http
GET /profile
```

Browser automatically sends

```
Cookie:

AuthCookie=abc123
```

The server validates the cookie and authenticates the user.

---

# ASP.NET Core Configuration

```csharp
builder.Services
.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
.AddCookie();
```

Protect an endpoint

```csharp
[Authorize]
public IActionResult Dashboard()
{
    return View();
}
```

---

# Cookie vs JWT

| Cookie Authentication | JWT Authentication |
|-----------------------|--------------------|
| Browser stores cookie | Client stores token |
| Cookie sent automatically | Client sends `Authorization: Bearer` header |
| Common for MVC/Web Apps | Common for REST APIs and mobile apps |
| Can rely on server-side session if desired | Typically stateless |

---

# Advantages

- Simple for browser-based applications
- Automatic cookie handling by browsers
- Well suited for MVC applications
- Supports sliding expiration

---

# Disadvantages

- Less suitable for mobile and third-party APIs
- Requires protection against CSRF attacks
- Browser-oriented

---

# Authentication Comparison

| Feature | JWT | OAuth 2.0 | OpenID Connect | Cookie Authentication |
|----------|-----|-----------|----------------|----------------------|
| Purpose | Authentication Token | Authorization Framework | Authentication Protocol | Authentication Mechanism |
| Used For | APIs | Delegated Access | User Login | Web Applications |
| Identity | Yes (claims) | No | Yes (ID Token) | Yes |
| Browser Friendly | Yes | Yes | Yes | Excellent |
| Mobile Friendly | Excellent | Excellent | Excellent | Limited |
| Stateless | Yes | Depends | Depends | Can be stateful or stateless depending on implementation |

---

# Complete Authentication Flow

```
User

↓

Login

↓

Authentication

↓

JWT / Cookie / OpenID Connect

↓

Identity Created

↓

Authorization

↓

Access API
```

---

# Interview Questions

## What is JWT?

A signed token used for stateless authentication, especially in Web APIs.

---

## What is OAuth 2.0?

An authorization framework that allows applications to access protected resources without sharing user passwords.

---

## What is OpenID Connect?

An authentication protocol built on OAuth 2.0 that provides user identity through an ID Token.

---

## What is Cookie Authentication?

A browser-based authentication mechanism where the server issues an encrypted authentication cookie after a successful login.

---

## Which authentication mechanism is best for REST APIs?

JWT Bearer Authentication is commonly preferred because it is stateless and works well across web, mobile, and distributed systems.

---

## Which authentication mechanism is best for traditional MVC applications?

Cookie Authentication is generally the preferred choice because browsers automatically manage cookies and it integrates naturally with server-rendered web applications.

---

# Quick Revision

- **JWT** → Stateless authentication token for APIs.
- **OAuth 2.0** → Authorization framework for delegated access.
- **OpenID Connect** → Authentication protocol built on OAuth 2.0 using an ID Token.
- **Cookie Authentication** → Browser-based authentication using encrypted cookies.
- OAuth answers **"What can this application access?"**
- OpenID Connect answers **"Who is the user?"**
- JWT is ideal for APIs; Cookies are ideal for traditional web applications.