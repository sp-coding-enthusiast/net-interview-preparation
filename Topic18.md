# Authentication & Authorization — OAuth 2.0 and OpenID Connect (OIDC)

OAuth 2.0 and OpenID Connect are among the most important authentication and authorization standards used in modern applications.

Examples:

- Login with Google
- Login with Microsoft
- Login with GitHub
- Login with Facebook

Almost every large application uses them.

---

# 1. Why OAuth Exists

Imagine a travel website wants access to your Google Calendar.

Without OAuth:

```text
Travel Website asks:
Give me your Gmail username and password.
```

This is dangerous.

Why?

- Website can read emails
- Website can change password
- Website has full account access

---

OAuth solves this problem.

Instead of giving your password:

```text
Google gives the website a limited access token.
```

The website never sees your password.

---

# 2. Layman Analogy

Imagine a hotel.

Without OAuth:

```text
You hand over your house key.
```

The hotel can access everything.

---

With OAuth:

```text
You give a temporary guest pass.
```

The pass only allows:

- entering one room
- for limited time

Not full access.

That guest pass is similar to:

```text
Access Token
```

---

# 3. OAuth 2.0 vs OpenID Connect

This is the most common interview question.

---

## OAuth 2.0

Purpose:

```text
Authorization
```

Answers:

```text
What can this application access?
```

---

## OpenID Connect (OIDC)

Purpose:

```text
Authentication
```

Answers:

```text
Who is the user?
```

---

# Easy Memory Trick

OAuth:

```text
Access Control
```

OIDC:

```text
Identity
```

---

# 4. Real Example

Login with Google.

---

## OAuth 2.0

Google says:

```text
Application can access:
✓ Calendar
✓ Contacts
```

---

## OIDC

Google says:

```text
User is John
Email = john@gmail.com
```

---

# 5. Important OAuth Components

---

## Resource Owner

The user.

Example:

```text
John
```

---

## Client

The application requesting access.

Example:

```text
Travel Website
```

---

## Authorization Server

Issues tokens.

Example:

```text
Google
Microsoft
Okta
Auth0
```

---

## Resource Server

Hosts protected data.

Example:

```text
Google Calendar API
```

---

# 6. OAuth 2.0 Flow (Authorization Code Flow)

Most commonly used flow.

---

## Step 1

User clicks:

```text
Login with Google
```

---

## Step 2

Application redirects to Google.

```http
https://accounts.google.com
```

---

## Step 3

User enters credentials.

```text
Username
Password
```

---

## Step 4

Google asks:

```text
Allow access to your profile?
```

---

## Step 5

User approves.

---

## Step 6

Google returns:

```text
Authorization Code
```

---

## Step 7

Application exchanges code for token.

---

## Step 8

Google returns:

```text
Access Token
Refresh Token
```

---

## Step 9

Application calls Google API.

```http
Authorization: Bearer access_token
```

---

# Flow Diagram

```text
User
 ↓
Application
 ↓
Google Login
 ↓
Authorization Code
 ↓
Access Token
 ↓
Google API
```

---

# 7. OAuth Tokens

---

## Access Token

Used to access resources.

Example:

```text
Valid for 15 minutes
```

---

## Refresh Token

Used to obtain new access tokens.

Example:

```text
Valid for 30 days
```

---

# 8. OAuth Scopes

Scopes define permissions.

Example:

```text
read_profile
read_email
calendar.read
calendar.write
```

---

Google consent screen:

```text
Allow access to:
✓ Email
✓ Profile
✓ Calendar
```

These are scopes.

---

# 9. What OpenID Connect Adds

OAuth only tells:

```text
What can be accessed
```

OIDC additionally tells:

```text
Who logged in
```

---

OIDC introduces:

```text
ID Token
```

---

# 10. ID Token

Usually a JWT.

Contains:

```json
{
  "sub": "123",
  "name": "John",
  "email": "john@gmail.com"
}
```

---

Application now knows:

```text
User identity
```

---

# 11. Tokens in OIDC

| Token | Purpose |
|---------|---------|
| Access Token | Access APIs |
| Refresh Token | Get new access token |
| ID Token | User identity |

---

# 12. OAuth vs JWT

Another common interview question.

---

## OAuth

Framework for authorization.

Example:

```text
Google Login
```

---

## JWT

Token format.

Example:

```text
Header.Payload.Signature
```

---

Relationship:

```text
OAuth may use JWT as access token
```

---

# 13. OAuth in Microservices

Typical architecture:

```text
Client
 ↓
Identity Provider
 ↓
JWT Token
 ↓
API Gateway
 ↓
Microservices
```

---

Identity Providers:

- Microsoft Entra ID
- Auth0
- Okta
- Keycloak

---

# 14. ASP.NET Core OIDC Example

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options =>
{
    options.Authority = "https://login.microsoftonline.com/tenant-id";

    options.ClientId = "client-id";

    options.ClientSecret = "client-secret";

    options.ResponseType = "code";
});
```

---

# 15. Why Large Companies Use OIDC

Without OIDC:

Every application manages:

```text
Users
Passwords
MFA
Security
```

Huge maintenance effort.

---

With OIDC:

```text
Identity Provider handles everything.
```

Applications only trust tokens.

---

# 16. Security Best Practices

---

## Use Authorization Code Flow

Most secure for web applications.

---

## Use PKCE

Especially for:

- Mobile apps
- SPA applications

Prevents code interception attacks.

---

## Use HTTPS

Always.

---

## Validate Tokens

Check:

- issuer
- audience
- expiry
- signature

---

## Use Least Privilege

Request only required scopes.

Bad:

```text
Request everything
```

Good:

```text
Request only email
```

---

# 17. OAuth Grant Types (Interview Overview)

---

## Authorization Code Flow

Most common.

Used by:

```text
Web Applications
```

---

## Authorization Code + PKCE

Most secure.

Used by:

```text
Mobile Apps
SPA Apps
```

---

## Client Credentials Flow

Machine-to-machine.

Example:

```text
Microservice → Microservice
```

No user involved.

---

## Device Flow

Used by:

```text
Smart TVs
IoT devices
```

---

# 18. Real Production Example

Microsoft Login.

```text
User clicks Login
 ↓
Redirect to Microsoft
 ↓
User authenticates
 ↓
Microsoft returns ID Token + Access Token
 ↓
Application validates token
 ↓
User signed in
```

---

# 19. Common Interview Questions

## Q: What is OAuth?

Authorization framework that allows delegated access without sharing passwords.

---

## Q: What is OpenID Connect?

Identity layer built on top of OAuth 2.0.

---

## Q: Difference between OAuth and OIDC?

OAuth:

```text
What can user access?
```

OIDC:

```text
Who is the user?
```

---

## Q: What is ID Token?

JWT containing user identity information.

---

## Q: What is PKCE?

Security enhancement for Authorization Code Flow.

---

# 20. Senior-Level Interview Answer

OAuth 2.0 is an authorization framework that enables applications to obtain delegated access to protected resources without exposing user credentials. OpenID Connect is an identity layer built on top of OAuth 2.0 that adds authentication capabilities through an ID Token, usually in JWT format. OAuth determines what resources an application can access, while OpenID Connect identifies who the authenticated user is. Together they provide secure, scalable authentication and authorization for modern web, mobile, and microservice-based applications.

---

# 21. Easy Way to Remember

```text
OAuth = Permissions
OIDC = Identity
JWT = Token Format
```

Or:

```text
OAuth asks:
"What can I access?"

OIDC asks:
"Who are you?"
```