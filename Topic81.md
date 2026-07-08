# 96. Claims

---

# What are Claims?

A **Claim** is a piece of information about a user.

Think of a claim as a **fact** or **attribute** that describes the user.

Examples:

- Name
- Email
- Age
- Department
- Country
- Employee ID
- Subscription
- Role

Instead of just knowing **who the user is**, claims tell us **more about the user**.

---

# Real-Life Analogy

Imagine your Aadhaar card or Passport.

It contains information like:

```
Name

Date of Birth

Gender

Nationality

Address
```

Each piece of information is a **claim**.

Similarly, a logged-in user has claims.

---

# Example JWT

```json
{
    "sub": "101",
    "name": "Saurabh",
    "email": "saurabh@gmail.com",
    "department": "IT",
    "country": "India"
}
```

Each field is a claim.

---

# Common Claims

| Claim | Meaning |
|--------|---------|
| sub | User ID |
| name | User Name |
| email | Email Address |
| role | User Role |
| country | Country |
| department | Department |
| exp | Expiration Time |
| iss | Issuer |
| aud | Audience |

---

# Accessing Claims

```csharp
var name = User.FindFirst("name")?.Value;

var email = User.FindFirst("email")?.Value;
```

---

# ASP.NET Core Example

```csharp
[Authorize]
public IActionResult Profile()
{
    var username = User.Identity?.Name;

    var department = User.FindFirst("department")?.Value;

    return Ok();
}
```

---

# Why Use Claims?

Instead of querying the database for every request,

the application reads user information directly from the authenticated identity.

---

# Claims Flow

```
Login

↓

Authentication

↓

Claims Created

↓

Stored in Identity

↓

Controller Reads Claims
```

---

# Advantages

- Flexible
- Lightweight
- No database lookup for every request
- Supports fine-grained authorization

---

# Interview Questions

## What is a Claim?

A claim is a key-value pair representing information about an authenticated user.

---

## Give examples of claims.

Name, Email, Role, Department, Country, Employee ID.

---

# Quick Revision

- Claims are facts about a user.
- Stored in the authenticated identity.
- Used for authorization decisions.
- Common in JWT and Cookie Authentication.

---

# 97. Policies

---

# What is a Policy?

A **Policy** is a set of authorization rules.

Instead of checking permissions directly in every controller,

you define the rules once and reuse them.

---

# Real-Life Analogy

Imagine entering a research laboratory.

Rules:

```
Employee

AND

Department = Research

AND

Security Clearance = High
```

All three conditions must be satisfied.

That collection of rules is a **policy**.

---

# Why Policies?

Without policies:

Every controller repeats the same checks.

```csharp
if(user.Department=="IT" &&
   user.Role=="Admin")
{
}
```

Repeated everywhere.

Policies centralize the logic.

---

# Creating a Policy

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("ITOnly",
        policy => policy.RequireClaim("department", "IT"));
});
```

---

# Using a Policy

```csharp
[Authorize(Policy = "ITOnly")]
public IActionResult GetData()
{
    return Ok();
}
```

Only users with

```
department = IT
```

can access it.

---

# Multiple Requirements

```csharp
options.AddPolicy("ManagersOnly", policy =>
{
    policy.RequireRole("Manager");
    policy.RequireClaim("department", "Sales");
});
```

User must satisfy both requirements.

---

# Policy Flow

```
User

↓

Claims

↓

Policy Evaluation

↓

Access Granted / Denied
```

---

# Advantages

- Centralized authorization
- Reusable
- Easy to maintain
- Supports complex rules

---

# Interview Questions

## What is a Policy?

A policy is a reusable authorization rule composed of one or more requirements.

---

## Why use policies?

To centralize and simplify authorization logic.

---

# Quick Revision

- Policy = Collection of authorization rules.
- Defined once.
- Reused everywhere.
- Can check roles, claims, or custom requirements.

---

# 98. Roles

---

# What are Roles?

A **Role** is a group that represents what type of user someone is.

Examples:

```
Admin

Manager

Customer

HR

Employee
```

Instead of assigning permissions individually,

users are assigned to roles.

---

# Real-Life Analogy

Imagine a hospital.

```
Doctor

Nurse

Receptionist

Patient
```

Each role has different permissions.

---

# Example

Admin

Can

- Create Users
- Delete Users
- Manage System

Customer

Can

- View Products
- Place Orders

---

# ASP.NET Core Example

```csharp
[Authorize(Roles = "Admin")]
public IActionResult DeleteUser()
{
    return Ok();
}
```

Only Admins can access.

---

# Multiple Roles

```csharp
[Authorize(Roles = "Admin,Manager")]
public IActionResult Reports()
{
    return Ok();
}
```

Both Admin and Manager are allowed.

---

# Role Flow

```
Login

↓

User Role

↓

Authorization

↓

Controller
```

---

# Roles vs Claims

Role

```
Admin
```

Claim

```
Department = IT

Country = India

Age = 35
```

Role is just one type of claim conceptually, but it is treated specially by the framework for role-based authorization.

---

# Advantages

- Simple
- Easy to understand
- Good for broad access control

---

# Disadvantages

- Not flexible for complex permissions
- Can lead to "role explosion" if many combinations are needed

---

# Interview Questions

## What is a Role?

A role is a collection of users with similar permissions, such as Admin or Manager.

---

## Difference between Role and Claim?

A role represents a broad category of users, while claims represent specific user attributes.

---

# Quick Revision

- Roles group users.
- Used in Role-Based Authorization.
- Simple and easy.
- Best for high-level permissions.

---

# 99. Identity

---

# What is Identity?

**ASP.NET Core Identity** is Microsoft's membership system used for managing:

- Users
- Passwords
- Roles
- Claims
- Login
- Registration
- Password Reset
- Email Confirmation
- Multi-Factor Authentication (MFA)

Instead of building all of this yourself,

Identity provides it out of the box.

---

# Real-Life Analogy

Imagine a company's HR system.

It manages:

- Employee records
- Login IDs
- Passwords
- Departments
- Access permissions

ASP.NET Core Identity does the same for application users.

---

# Identity Components

```
User

↓

Role

↓

Claim

↓

Authentication

↓

Authorization
```

---

# Identity Database Tables

Some common tables:

```
AspNetUsers

AspNetRoles

AspNetUserRoles

AspNetUserClaims

AspNetRoleClaims

AspNetUserLogins

AspNetUserTokens
```

---

# Registering Identity

```csharp
builder.Services.AddIdentity<IdentityUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();
```

---

# Creating a User

```csharp
var user = new IdentityUser
{
    UserName = "saurabh",
    Email = "saurabh@gmail.com"
};

await userManager.CreateAsync(user, "Password@123");
```

---

# Adding a Role

```csharp
await userManager.AddToRoleAsync(user, "Admin");
```

---

# Login

```csharp
var result = await signInManager.PasswordSignInAsync(
    "saurabh",
    "Password@123",
    false,
    false);
```

---

# Identity Flow

```
Register

↓

User Stored

↓

Login

↓

Authentication

↓

Claims Principal Created

↓

Authorization

↓

Application Access
```

---

# Identity Features

- User Registration
- Login
- Logout
- Password Hashing
- Roles
- Claims
- Email Confirmation
- Password Reset
- MFA
- External Login (Google, Microsoft, Facebook, etc.)
- Account Lockout

---

# Identity vs Authentication

| Identity | Authentication |
|----------|----------------|
| User management framework | Identity verification process |
| Stores users and roles | Verifies credentials |
| Registration and password reset | Login |
| Password hashing | Token/Cookie validation |

Identity **provides** authentication features, but authentication itself is the process of verifying identity.

---

# Identity vs JWT

Identity

```
Manages Users
```

JWT

```
Authenticates API Requests
```

Many applications use ASP.NET Core Identity to manage users and then issue JWTs for API authentication.

---

# Complete Security Flow

```
Register

↓

Identity Stores User

↓

Login

↓

Authentication

↓

JWT / Cookie Created

↓

Claims Principal

↓

Authorization

↓

Controller
```

---

# Claims vs Roles vs Policies vs Identity

| Claims | Roles | Policies | Identity |
|---------|--------|----------|----------|
| User attributes | User groups | Authorization rules | User management framework |
| Example: Department = IT | Example: Admin | Example: ITOnly Policy | Handles users, roles, passwords, MFA, etc. |

---

# Interview Questions

## What is ASP.NET Core Identity?

ASP.NET Core Identity is Microsoft's framework for user management, authentication, and authorization features such as users, roles, claims, password hashing, and account management.

---

## What tables are created by Identity?

Examples include:

- AspNetUsers
- AspNetRoles
- AspNetUserRoles
- AspNetUserClaims
- AspNetRoleClaims
- AspNetUserLogins
- AspNetUserTokens

---

## Can Identity work with JWT?

Yes.

Identity manages users and credentials, while JWT can be used to authenticate API requests after a successful login.

---

## Difference between Claims, Roles, Policies, and Identity?

- **Claims** describe user attributes.
- **Roles** group users with similar permissions.
- **Policies** define reusable authorization rules.
- **Identity** manages users, credentials, roles, claims, and authentication features.

---

# Quick Revision

- **Claims** = Facts about a user (Name, Email, Department, Country).
- **Policies** = Reusable authorization rules.
- **Roles** = User groups such as Admin and Manager.
- **Identity** = Complete user management framework.
- Identity supports registration, login, password hashing, roles, claims, MFA, and external logins.
- Roles are useful for broad permissions; Claims and Policies provide more fine-grained authorization.