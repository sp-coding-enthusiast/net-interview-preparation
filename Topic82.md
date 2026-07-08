# ASP.NET Core Identity (Complete Guide)

---

# What is ASP.NET Core Identity?

**ASP.NET Core Identity** is Microsoft's built-in **authentication and user management framework**.

It provides everything needed to manage users in an application without writing authentication code from scratch.

It handles:

- User Registration
- Login
- Logout
- Password Hashing
- Roles
- Claims
- Email Confirmation
- Password Reset
- Multi-Factor Authentication (MFA)
- Account Lockout
- External Login (Google, Microsoft, Facebook, etc.)

---

# Layman Explanation

Imagine you are building a society apartment.

The society needs:

- Register residents
- Verify identity
- Issue access cards
- Change passwords
- Block unauthorized people
- Assign security guards
- Allow visitors

Instead of building all this yourself,

you purchase a **complete security management system**.

ASP.NET Core Identity is that ready-made security system.

---

# Why Do We Need Identity?

Without Identity, developers must build:

```
Registration

↓

Login

↓

Password Hashing

↓

Forgot Password

↓

Reset Password

↓

Email Verification

↓

Role Management

↓

Security

↓

Account Lockout

↓

MFA
```

This is a lot of work and easy to get wrong.

Identity provides these features out of the box.

---

# Where is Identity Used?

- Banking Applications
- E-Commerce Websites
- Hospital Systems
- School Portals
- HR Systems
- Admin Portals
- Enterprise Applications
- SaaS Products

---

# Features of ASP.NET Core Identity

- User Registration
- Login
- Logout
- Password Hashing
- Role Management
- Claims Management
- Email Confirmation
- Password Reset
- Account Lockout
- Two-Factor Authentication (2FA/MFA)
- External Login Providers
- Token Generation
- Security Stamp Validation

---

# Identity Architecture

```
               User
                 │
                 ▼
        ASP.NET Core Identity
                 │
   ┌─────────────┼─────────────┐
   ▼             ▼             ▼
UserManager  SignInManager  RoleManager
                 │
                 ▼
         Entity Framework Core
                 │
                 ▼
             SQL Database
```

---

# Main Components

## 1. IdentityUser

Represents an application user.

Example

```csharp
IdentityUser
```

Contains properties like

```
Id

UserName

Email

PhoneNumber

PasswordHash

SecurityStamp

LockoutEnd
```

---

## 2. IdentityRole

Represents a role.

Examples

```
Admin

Manager

Employee

Customer
```

---

## 3. UserManager

Responsible for user management.

It performs operations like:

- Create User
- Update User
- Delete User
- Change Password
- Add Claims
- Add Roles

---

Example

```csharp
await userManager.CreateAsync(user, password);
```

---

## 4. SignInManager

Handles login and logout.

Example

```csharp
await signInManager.PasswordSignInAsync(
    email,
    password,
    false,
    false);
```

---

## 5. RoleManager

Creates and manages roles.

Example

```csharp
await roleManager.CreateAsync(
    new IdentityRole("Admin"));
```

---

# Identity Database Tables

When Identity is installed,

Entity Framework creates several tables.

---

## AspNetUsers

Stores users.

Example

| Id | UserName | Email |
|----|----------|-------|
| 1 | Saurabh | saurabh@gmail.com |

---

## AspNetRoles

Stores roles.

| Id | Role |
|----|------|
| 1 | Admin |
| 2 | Manager |

---

## AspNetUserRoles

Maps users to roles.

| UserId | RoleId |
|---------|--------|
| 1 | Admin |

---

## AspNetUserClaims

Stores user claims.

| User | Claim |
|------|--------|
| Saurabh | Department = IT |

---

## AspNetRoleClaims

Stores claims assigned to roles.

---

## AspNetUserLogins

Stores external logins.

Example

```
Google

Microsoft

Facebook
```

---

## AspNetUserTokens

Stores authentication tokens.

---

# Identity Flow

```
User

↓

Register

↓

AspNetUsers

↓

Login

↓

Password Verification

↓

Claims Principal Created

↓

Cookie/JWT Generated

↓

Authenticated User

↓

Authorization
```

---

# Installing Identity

Install NuGet package

```
Microsoft.AspNetCore.Identity.EntityFrameworkCore
```

---

Register Identity

```csharp
builder.Services
.AddIdentity<IdentityUser, IdentityRole>()
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();
```

---

# Configure Authentication Middleware

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

Authentication must come first.

---

# Register a User

```csharp
public async Task<IActionResult> Register(RegisterModel model)
{
    var user = new IdentityUser
    {
        UserName = model.Email,
        Email = model.Email
    };

    var result = await userManager.CreateAsync(
        user,
        model.Password);

    if(result.Succeeded)
        return Ok();

    return BadRequest(result.Errors);
}
```

---

# Login

```csharp
var result =
await signInManager.PasswordSignInAsync(

model.Email,

model.Password,

false,

false
);
```

If successful,

the user is authenticated.

---

# Logout

```csharp
await signInManager.SignOutAsync();
```

---

# Password Hashing

Identity never stores passwords.

Instead

```
Password

↓

Hash

↓

Database
```

Example

```
Password123

↓

AQAAAAEAACcQAAAA...
```

Even administrators cannot see the original password.

---

# Password Verification

```
User Password

↓

Hash

↓

Compare

↓

Login Success
```

The original password is never stored.

---

# Role Management

Create Role

```csharp
await roleManager.CreateAsync(
new IdentityRole("Admin"));
```

Assign Role

```csharp
await userManager.AddToRoleAsync(
user,
"Admin");
```

Check Role

```csharp
[Authorize(Roles="Admin")]
```

---

# Claims Management

Add Claim

```csharp
await userManager.AddClaimAsync(

user,

new Claim("Department","IT"));
```

Read Claim

```csharp
User.FindFirst("Department")?.Value
```

---

# Email Confirmation

Registration

↓

Generate Token

↓

Send Email

↓

User Clicks Link

↓

Email Confirmed

Example

```csharp
var token =
await userManager.GenerateEmailConfirmationTokenAsync(user);
```

---

# Forgot Password

```
Forgot Password

↓

Generate Token

↓

Email Link

↓

Reset Password

↓

Success
```

---

Generate Token

```csharp
var token =
await userManager.GeneratePasswordResetTokenAsync(user);
```

---

Reset Password

```csharp
await userManager.ResetPasswordAsync(

user,

token,

newPassword);
```

---

# Account Lockout

Suppose

Password entered incorrectly

5 times.

Identity automatically locks account.

```
Attempts

↓

5 Failures

↓

Locked

↓

30 Minutes
```

---

# Two-Factor Authentication (2FA)

Login

↓

Password

↓

OTP

↓

Authenticated

Supported methods

- SMS
- Email
- Authenticator Apps

---

# External Login

Identity supports

```
Google

Microsoft

Facebook

GitHub
```

Flow

```
User

↓

Google Login

↓

Identity

↓

Application
```

---

# Custom User Class

Instead of

```csharp
IdentityUser
```

Create

```csharp
public class ApplicationUser
    : IdentityUser
{
    public string Department { get; set; }

    public string City { get; set; }
}
```

Now register

```csharp
builder.Services
.AddIdentity<ApplicationUser,
IdentityRole>();
```

---

# Identity with JWT

Modern Web APIs commonly use this flow:

```
Register

↓

Identity Stores User

↓

Login

↓

Verify Password

↓

Generate JWT

↓

Client Stores JWT

↓

Future Requests

↓

Authorization Header

↓

Authenticated
```

Identity manages users,

JWT authenticates API requests.

---

# Identity vs Authentication

| Identity | Authentication |
|----------|----------------|
| User management framework | Identity verification process |
| Registration | Login |
| Password Reset | Credential validation |
| Roles | Token/Cookie validation |
| Claims | Authentication result |

---

# Identity vs Authorization

| Identity | Authorization |
|----------|---------------|
| Manages users | Checks permissions |
| Stores roles | Uses roles |
| Stores claims | Uses claims |
| Registration | Access control |

---

# Identity Managers

| Manager | Responsibility |
|----------|----------------|
| UserManager | Manage users |
| SignInManager | Login & Logout |
| RoleManager | Manage roles |

---

# Complete Identity Lifecycle

```
Register

↓

Create User

↓

Hash Password

↓

Store User

↓

Email Confirmation

↓

Login

↓

Verify Password

↓

Create Claims Principal

↓

Cookie/JWT

↓

Authentication

↓

Authorization

↓

Controller
```

---

# Advantages

- Built-in security
- Password hashing
- Role management
- Claims support
- MFA support
- Email confirmation
- Password reset
- External logins
- Account lockout
- Highly customizable

---

# Disadvantages

- More complex than simple JWT-only authentication
- Many database tables
- Learning curve for beginners
- Can be excessive for very small applications

---

# Best Practices

- Never store plain-text passwords.
- Always enable HTTPS.
- Enable email confirmation for production applications.
- Use strong password policies.
- Enable account lockout to prevent brute-force attacks.
- Enable MFA for sensitive applications.
- Use roles for broad permissions and claims/policies for fine-grained authorization.
- Issue JWT tokens from Identity for Web APIs.

---

# Frequently Asked Interview Questions

## What is ASP.NET Core Identity?

ASP.NET Core Identity is Microsoft's built-in membership framework for managing users, authentication, roles, claims, password hashing, and related security features.

---

## What is UserManager?

`UserManager` is responsible for creating, updating, deleting, and managing users.

---

## What is SignInManager?

`SignInManager` handles user sign-in, sign-out, and authentication workflows.

---

## What is RoleManager?

`RoleManager` manages roles such as creating, updating, deleting, and checking roles.

---

## Does Identity store passwords?

No.

Passwords are securely hashed before being stored.

---

## Can Identity work with JWT?

Yes.

Identity manages users and credentials, while JWT is commonly issued after a successful login to authenticate API requests.

---

## What tables are created by Identity?

- AspNetUsers
- AspNetRoles
- AspNetUserRoles
- AspNetUserClaims
- AspNetRoleClaims
- AspNetUserLogins
- AspNetUserTokens

---

## Difference between Identity and JWT?

Identity is the user management framework.

JWT is a token used to authenticate requests after the user has been authenticated.

---

# Quick Revision

- ASP.NET Core Identity is Microsoft's complete authentication and user management framework.
- Core classes: `IdentityUser`, `IdentityRole`, `UserManager`, `SignInManager`, and `RoleManager`.
- Provides registration, login, logout, password hashing, roles, claims, MFA, password reset, and external logins.
- Stores data in tables such as `AspNetUsers` and `AspNetRoles`.
- Frequently used with JWT for Web APIs and Cookie Authentication for MVC applications.
- Passwords are never stored in plain text—they are securely hashed.
- Supports fine-grained authorization using roles, claims, and policies.