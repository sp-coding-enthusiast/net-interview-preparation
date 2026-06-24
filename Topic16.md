# API Security Best Practices (Web API / ASP.NET Core)

API security is about **protecting your backend from unauthorized access, data leaks, and abuse** while ensuring only valid users and systems can access resources.

---

# 1. Core Idea (Layman View)

Think of your API like a building:

- API = building
- Users = visitors
- Data = valuables inside

Security ensures:

- Only allowed people enter
- They can only access permitted rooms
- Nothing is stolen or misused

---

# 2. Authentication vs Authorization (MOST IMPORTANT)

## Authentication = Who are you?

Example:

```text
Login using username/password or JWT token
```

---

## Authorization = What can you do?

Example:

```text
Admin can delete users
Normal user cannot
```

---

## In API terms:

```http
Authorization: Bearer <token>
```

---

# 3. Use HTTPS Everywhere

## Rule:

```text
Never use HTTP in production
```

---

## Why?

Without HTTPS:

- data is visible in transit
- passwords can be intercepted

With HTTPS:

- encrypted communication
- secure data transfer

---

# 4. Use Strong Authentication (JWT / OAuth2)

## JWT Example:

```text
Header.Payload.Signature
```

---

## Flow:

```text
User logs in → Server issues JWT → Client sends JWT in every request
```

---

## API request:

```http
GET /api/orders
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

---

# 5. Proper Authorization (Role-based Access Control)

## Example:

```csharp
[Authorize(Roles = "Admin")]
public IActionResult DeleteUser()
```

---

## Real-world roles:

- Admin
- User
- Manager
- Support

---

# 6. Input Validation (VERY IMPORTANT)

Never trust client input.

---

## Bad example:

```csharp
var query = "SELECT * FROM Users WHERE Name = '" + name + "'";
```

❌ SQL Injection risk

---

## Good example:

```csharp
var user = _db.Users.FirstOrDefault(x => x.Name == name);
```

---

## Always validate:

- required fields
- data types
- ranges
- formats

---

# 7. Prevent SQL Injection

## Use:

- Parameterized queries
- ORM (Entity Framework)

---

## Example:

```csharp
db.Users.Where(u => u.Email == email)
```

---

# 8. Rate Limiting (Prevent Abuse)

## Problem:

Attacker sends:

```text
10000 requests per second
```

---

## Solution:

Limit requests:

```text
100 requests per minute per user
```

---

## Result:

- prevents DDoS attacks
- protects backend resources

---

# 9. Use API Gateway (Microservices)

API Gateway provides:

- authentication
- routing
- rate limiting
- logging

---

## Architecture:

```text
Client → API Gateway → Microservices
```

---

# 10. Secure Sensitive Data

Never expose:

- passwords
- credit card numbers
- internal IDs

---

## Bad response:

```json
{
  "password": "12345"
}
```

---

## Good response:

```json
{
  "name": "John"
}
```

---

# 11. Use DTOs (Data Transfer Objects)

Prevent exposing domain models directly.

---

## Why?

- hides internal structure
- improves security
- avoids accidental leaks

---

# 12. Use CORS Carefully

## CORS controls:

```text
Which frontend apps can call your API
```

---

## Example:

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend",
        policy => policy.WithOrigins("https://myapp.com")
                        .AllowAnyHeader()
                        .AllowAnyMethod());
});
```

---

# 13. Secure Headers

Add security headers:

- X-Content-Type-Options
- X-Frame-Options
- Content-Security-Policy

---

## Example:

```text
Prevents XSS and clickjacking attacks
```

---

# 14. Logging & Monitoring

Track:

- failed logins
- suspicious activity
- API usage patterns

---

## Example:

```text
User attempted 50 failed logins → block IP
```

---

# 15. Avoid Sensitive Data in URLs

## Bad:

```http
GET /api/user?password=12345
```

---

## Good:

```http
POST /api/login
Body: { "password": "12345" }
```

---

# 16. Use Secrets Management

Never store secrets in code.

---

## Bad:

```csharp
string apiKey = "123456";
```

---

## Good:

- Azure Key Vault
- AWS Secrets Manager
- Environment variables

---

# 17. Use Token Expiration

JWT tokens must expire:

```text
15 min / 1 hour / 1 day
```

---

## Why?

- reduces risk of token theft
- forces re-authentication

---

# 18. Protect Against Brute Force Attacks

## Strategy:

- account lockout
- captcha
- rate limiting

---

# 19. Version Your API

Security includes stability:

```text
/v1/login
/v2/login
```

Prevents breaking secure flows.

---

# 20. Common Mistakes

---

## ❌ 1. No authentication on APIs

Open endpoints = security risk

---

## ❌ 2. Using weak passwords

No complexity rules

---

## ❌ 3. Storing secrets in code

Hardcoded credentials

---

## ❌ 4. Overexposing APIs

Too many public endpoints

---

## ❌ 5. Not validating input

Leads to injection attacks

---

# 21. Real Production Security Flow

```text
Client
  ↓
HTTPS
  ↓
API Gateway (Rate Limit + Auth)
  ↓
JWT Validation
  ↓
Authorization (Role check)
  ↓
Controller
  ↓
Service Layer
  ↓
Database (Parameterized queries)
```

---

# 22. Senior-Level Interview Answer

API security best practices include using HTTPS for secure communication, implementing strong authentication (JWT/OAuth2), enforcing proper authorization using roles and policies, validating all inputs, preventing SQL injection through parameterized queries, and protecting APIs using rate limiting and API gateways. Sensitive data should never be exposed in responses, and secrets must be stored securely using environment variables or secret managers. Additional measures such as CORS configuration, security headers, logging, token expiration, and monitoring are essential for building secure and scalable APIs.

---

# 23. Easy Way to Remember

```text
Authenticate → Identify user
Authorize → Control access
Validate → Trust no input
Encrypt → Protect data
Limit → Prevent abuse
```