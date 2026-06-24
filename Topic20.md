# Authentication & Authorization — IdentityServer / Azure AD Basics

This is one of the most important authentication topics for Senior, Lead, Architect, and Microservices interviews.

Modern applications rarely store usernames and passwords in every application.

Instead, they use a centralized Identity Provider (IdP) such as:

- IdentityServer
- Azure Active Directory (Azure AD / Microsoft Entra ID)
- Auth0
- Okta
- Keycloak

---

# 1. Why Do We Need Identity Providers?

Imagine a company has:

```text
HR Portal
Payroll System
Leave Management
CRM
Expense Portal
```

Without Identity Provider:

```text
Each application manages:
- Users
- Passwords
- MFA
- Security
```

Huge duplication.

---

# Problem

If employee changes password:

```text
Need to update in 5 systems
```

Very difficult.

---

# Solution

Use one central authentication server.

```text
Identity Provider (IdP)
```

All applications trust it.

---

# 2. Layman Analogy

Think of an airport.

You show your passport once.

Security verifies identity.

After that:

```text
Board flight
Enter lounge
Check baggage
```

Nobody asks for your passport repeatedly.

---

Identity Provider works similarly.

```text
Login once
Get token
Use token everywhere
```

This is called:

```text
Single Sign-On (SSO)
```

---

# 3. What is IdentityServer?

IdentityServer is a .NET-based Identity Provider.

It provides:

- Authentication
- Authorization
- OAuth2
- OpenID Connect
- JWT token issuance
- Single Sign-On

---

# Responsibilities

IdentityServer:

```text
✓ Login users
✓ Validate credentials
✓ Generate JWT tokens
✓ Handle OAuth2
✓ Handle OpenID Connect
✓ Manage clients
```

---

# Architecture

```text
Client
  ↓
IdentityServer
  ↓
JWT Token
  ↓
API
```

---

# 4. What is Azure AD (Microsoft Entra ID)?

Azure AD is Microsoft's cloud-based Identity Provider.

Current name:

```text
Microsoft Entra ID
```

(Many interviewers still call it Azure AD.)

---

It provides:

```text
Identity Management as a Service
```

---

Used by:

- Microsoft 365
- Outlook
- Teams
- Azure
- Enterprise Applications

---

# 5. IdentityServer vs Azure AD

| Feature | IdentityServer | Azure AD |
|----------|---------------|----------|
| Hosted By | Your organization | Microsoft |
| Infrastructure | Self-managed | Cloud-managed |
| Maintenance | Your responsibility | Microsoft |
| OAuth2 | Yes | Yes |
| OpenID Connect | Yes | Yes |
| SSO | Yes | Yes |
| Enterprise Ready | Yes | Yes |

---

# Easy Explanation

IdentityServer:

```text
Build your own security office
```

Azure AD:

```text
Rent Microsoft's security office
```

---

# 6. Single Sign-On (SSO)

One login.

Many applications.

---

Without SSO

```text
Login HR Portal
Login CRM
Login Payroll
Login Expense App
```

Four separate logins.

---

With SSO

```text
Login once
```

Access everything.

---

Architecture

```text
User
 ↓
Azure AD
 ↓
HR Portal
CRM
Payroll
Expense App
```

---

# 7. Authentication Flow

Step 1

```text
User clicks Login
```

---

Step 2

Redirect to:

```text
Azure AD
IdentityServer
```

---

Step 3

User enters credentials.

---

Step 4

Identity Provider validates user.

---

Step 5

Returns:

```text
ID Token
Access Token
```

---

Step 6

Application trusts token.

---

Step 7

Access granted.

---

# Flow Diagram

```text
User
 ↓
Application
 ↓
Identity Provider
 ↓
JWT Token
 ↓
Application
 ↓
Protected Resources
```

---

# 8. Access Token vs ID Token

Many interviewers ask this.

---

## Access Token

Used for:

```text
Authorization
```

Allows API access.

Example:

```http
Authorization: Bearer token
```

---

## ID Token

Used for:

```text
Authentication
```

Contains user identity.

Example:

```json
{
  "name": "John",
  "email": "john@gmail.com"
}
```

---

# 9. OAuth2 + OpenID Connect

Both IdentityServer and Azure AD support:

---

OAuth2

Answers:

```text
What can user access?
```

---

OIDC

Answers:

```text
Who is the user?
```

---

# 10. Azure AD Concepts

---

## Tenant

Company's identity boundary.

Example:

```text
Contoso Corporation
```

One Azure AD tenant.

---

Think of tenant as:

```text
Company security directory
```

---

## Users

Employees.

Example:

```text
john@contoso.com
```

---

## Groups

Collections of users.

Example:

```text
Finance Team
HR Team
```

---

## App Registration

Application registered in Azure AD.

Example:

```text
Payroll Portal
```

---

# 11. App Registration (Very Important)

Before Azure AD can authenticate users:

Application must be registered.

---

Azure AD generates:

```text
Client ID
Tenant ID
```

---

Used during authentication.

---

Example

```text
ClientId:
123-456-789

TenantId:
abc-xyz
```

---

# 12. JWT Issued by Azure AD

Example Claims:

```json
{
  "sub": "123",
  "name": "John",
  "email": "john@company.com",
  "roles": ["Admin"]
}
```

---

Application reads claims.

---

# 13. ASP.NET Core + Azure AD

Typical configuration:

```csharp
builder.Services
.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
.AddMicrosoftIdentityWebApi(builder.Configuration);
```

---

appsettings.json

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "tenant-id",
    "ClientId": "client-id"
  }
}
```

---

# 14. Authorization Using Azure AD Roles

JWT contains:

```json
{
  "role": "Admin"
}
```

---

Controller

```csharp
[Authorize(Roles = "Admin")]
public IActionResult DeleteUser()
{
    return Ok();
}
```

---

# 15. Microservices Architecture

Very common enterprise setup.

```text
User
 ↓
Azure AD
 ↓
JWT Token
 ↓
API Gateway
 ↓
Order Service
Payment Service
Inventory Service
```

---

Benefits:

- Centralized Authentication
- Single Sign-On
- No repeated logins

---

# 16. Why Enterprises Prefer Azure AD

Because Microsoft manages:

```text
Passwords
MFA
Identity Security
Token Issuance
Threat Detection
```

Teams focus on business features.

---

# 17. Common Interview Questions

## Q: What is IdentityServer?

A .NET-based Identity Provider implementing OAuth2 and OpenID Connect.

---

## Q: What is Azure AD?

Microsoft's cloud identity platform (now Microsoft Entra ID).

---

## Q: Difference?

IdentityServer:

```text
Self-hosted
```

Azure AD:

```text
Microsoft-hosted
```

---

## Q: What is SSO?

Single Sign-On.

Login once and access multiple applications.

---

## Q: Why use Azure AD?

Centralized authentication, MFA, SSO, enterprise security.

---

## Q: What does Azure AD issue?

```text
Access Token
ID Token
Refresh Token
```

---

# 18. Real Production Example

Employee opens:

```text
Expense Portal
```

---

Application redirects to:

```text
Microsoft Entra ID
```

---

Employee authenticates.

---

Azure AD issues:

```text
JWT Token
```

---

Expense Portal validates token.

---

Employee accesses system.

---

No password stored in Expense Portal.

---

# 19. Senior-Level Interview Answer

IdentityServer and Azure AD (Microsoft Entra ID) are Identity Providers that centralize authentication and authorization for applications. They implement OAuth 2.0 and OpenID Connect standards, enabling secure login, token issuance, Single Sign-On (SSO), and federated identity management. IdentityServer is a self-hosted .NET solution, while Azure AD is a cloud-managed service provided by Microsoft. In modern enterprise and microservice architectures, these platforms issue JWT access and ID tokens that allow applications and APIs to authenticate users and enforce authorization without managing passwords directly.

---

# 20. Easy Way to Remember

```text
IdentityServer = Build your own Identity Provider

Azure AD (Entra ID) = Use Microsoft's Identity Provider
```

And:

```text
Login Once
↓
Get Token
↓
Access Many Applications
```