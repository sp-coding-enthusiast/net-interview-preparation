# Authentication & Authorization — Role-Based vs Policy-Based Authorization

Authorization determines:

```text
"What is the authenticated user allowed to do?"
```

In ASP.NET Core, the two most common authorization approaches are:

1. Role-Based Authorization
2. Policy-Based Authorization

This is a very common Senior/Lead-level interview topic.

---

# 1. Authentication vs Authorization

Before understanding roles and policies:

## Authentication

Answers:

```text
Who are you?
```

Example:

```text
User logged in using JWT
```

---

## Authorization

Answers:

```text
What are you allowed to do?
```

Example:

```text
Can this user delete an order?
```

---

# 2. Role-Based Authorization

## What is it?

Access is granted based on a user's role.

Example roles:

```text
Admin
Manager
Employee
Customer
Support
```

---

# Layman Analogy

Think of a company.

Employees have job titles:

```text
CEO
Manager
Engineer
Intern
```

Permissions are assigned based on title.

Example:

```text
CEO can approve budget
Intern cannot
```

That's Role-Based Authorization.

---

# ASP.NET Core Example

## JWT Claims

```json
{
  "name": "John",
  "role": "Admin"
}
```

---

## Protect Endpoint

```csharp
[Authorize(Roles = "Admin")]
public IActionResult DeleteUser()
{
    return Ok();
}
```

Only Admin can access.

---

# Multiple Roles

```csharp
[Authorize(Roles = "Admin,Manager")]
public IActionResult ApproveOrder()
{
    return Ok();
}
```

Meaning:

```text
Admin OR Manager
```

can access.

---

# Advantages of Role-Based Authorization

✔ Simple

✔ Easy to understand

✔ Quick implementation

✔ Works well for small systems

---

# Disadvantages

As applications grow:

```text
Roles become huge and complex
```

Example:

```text
Admin
SuperAdmin
RegionalAdmin
GlobalAdmin
OperationsAdmin
FinanceAdmin
```

Now authorization becomes difficult to manage.

---

# Real Problem

Suppose:

```text
Manager can approve orders under ₹1,00,000
Director can approve orders under ₹10,00,000
```

Roles alone cannot easily express this rule.

---

# 3. Policy-Based Authorization

## What is it?

Authorization is based on rules (requirements) instead of roles.

Think:

```text
If condition is true → allow access
```

---

# Layman Analogy

At an airport:

Access depends on:

- Valid ticket
- Passport
- Visa
- Security clearance

Not on job title.

Multiple conditions determine access.

That's Policy-Based Authorization.

---

# Example Requirement

```text
User must:
✓ Be authenticated
✓ Have age >= 18
✓ Belong to India
```

Role alone cannot handle this.

Policy can.

---

# ASP.NET Core Policy Example

## Configure Policy

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdultOnly", policy =>
    {
        policy.RequireClaim("Age", "18");
    });
});
```

---

## Use Policy

```csharp
[Authorize(Policy = "AdultOnly")]
public IActionResult GetAdultContent()
{
    return Ok();
}
```

---

# Example JWT

```json
{
  "name": "John",
  "age": "18"
}
```

Policy evaluates claim.

---

# More Realistic Policy

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminFromIndia", policy =>
    {
        policy.RequireRole("Admin");
        policy.RequireClaim("Country", "India");
    });
});
```

---

# Access Rule

```text
Must be:
✓ Admin
✓ From India
```

Both conditions required.

---

# 4. Custom Policy (Most Powerful)

Suppose:

```text
Order value < ₹1,00,000
AND
Manager experience > 5 years
```

Cannot be solved using simple roles.

Create custom requirement.

---

## Requirement

```csharp
public class MinimumExperienceRequirement
    : IAuthorizationRequirement
{
    public int Years { get; }

    public MinimumExperienceRequirement(int years)
    {
        Years = years;
    }
}
```

---

## Handler

```csharp
public class ExperienceHandler
    : AuthorizationHandler<MinimumExperienceRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumExperienceRequirement requirement)
    {
        var experience =
            int.Parse(context.User.FindFirst("Experience")?.Value ?? "0");

        if (experience >= requirement.Years)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

---

# Registration

```csharp
options.AddPolicy("SeniorManager",
    policy =>
    {
        policy.Requirements.Add(
            new MinimumExperienceRequirement(5));
    });
```

---

# Real Enterprise Example

Banking system:

```text
User must:
✓ Be authenticated
✓ Be employee
✓ Have security clearance level 4
✓ Belong to finance department
✓ Access only during office hours
```

Role-Based becomes messy.

Policy-Based handles this easily.

---

# 5. Role-Based vs Policy-Based

| Feature | Role-Based | Policy-Based |
|----------|------------|-------------|
| Based on | Roles | Rules/Requirements |
| Complexity | Simple | Advanced |
| Flexibility | Low | High |
| Dynamic Conditions | Difficult | Easy |
| Enterprise Usage | Moderate | High |
| Custom Logic | Limited | Excellent |

---

# 6. Real-World Examples

## Role-Based

```text
Admin
Manager
User
```

Access based on designation.

---

## Policy-Based

```text
Age > 18
Country = India
Department = Finance
Experience > 5 years
```

Access based on business rules.

---

# 7. When to Use Role-Based

Use when:

```text
Small application
Simple permissions
Few roles
```

Examples:

- Internal tools
- Small business apps
- Admin portals

---

# 8. When to Use Policy-Based

Use when:

```text
Complex business rules
Large enterprise systems
Banking systems
Insurance systems
Government systems
Microservices
```

---

# 9. Combining Both (Very Common)

Most enterprise applications use both.

Example:

```csharp
options.AddPolicy("FinanceApprover",
    policy =>
    {
        policy.RequireRole("Manager");
        policy.RequireClaim("Department", "Finance");
    });
```

Rule:

```text
Role = Manager
AND
Department = Finance
```

---

# 10. Microservices Example

JWT Token:

```json
{
  "role": "Manager",
  "country": "India",
  "department": "Finance",
  "experience": "7"
}
```

Policy checks claims.

Each microservice can enforce its own rules.

---

# 11. Common Interview Questions

## Q: Which is more flexible?

Answer:

```text
Policy-Based Authorization
```

---

## Q: Which is easier?

Answer:

```text
Role-Based Authorization
```

---

## Q: Can policies use roles?

Answer:

```text
Yes
```

Very common.

---

## Q: Which is preferred in enterprise systems?

Answer:

```text
Policy-Based Authorization
```

Because business rules are usually more complex than simple roles.

---

# 12. Senior-Level Interview Answer

Role-Based Authorization grants access based on predefined user roles such as Admin, Manager, or User. It is simple to implement and suitable for small to medium-sized applications. Policy-Based Authorization is more flexible and evaluates one or more requirements such as claims, roles, custom business rules, or contextual conditions before granting access. In enterprise applications, policy-based authorization is generally preferred because it supports complex authorization scenarios and provides better maintainability and scalability.

---

# 13. Easy Way to Remember

```text
Role-Based:
"What is your designation?"

Policy-Based:
"Do you satisfy all business rules?"
```

Or:

```text
Role = Job Title

Policy = Rule Engine
```