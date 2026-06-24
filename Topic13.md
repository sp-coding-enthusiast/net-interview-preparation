# Web API Design — Versioning Strategies (Layman + Practical)

API versioning is how we manage **changes in a Web API without breaking existing clients**.

---

# 1. Why API Versioning is Needed

Imagine your API is used by:

- Mobile App (Android)
- Mobile App (iOS)
- Web Application
- Third-party customers

Now you change your API:

```text
Old response:
{
  "name": "John"
}

New response:
{
  "fullName": "John Doe",
  "email": "john@email.com"
}
```

❌ Problem:
Older clients will break because they expect `name`, not `fullName`.

---

# 2. Layman Analogy

Think of a mobile phone charger.

- Old version: Micro USB
- New version: USB-C

If you change the plug:

- Old phones stop working
- New phones work fine

So instead of replacing everything, you keep **both versions supported**.

That is API versioning.

---

# 3. What is API Versioning?

Versioning means:

> Running multiple versions of the same API at the same time.

Example:

```text
/api/v1/users
/api/v2/users
```

---

# 4. Common Versioning Strategies

There are 4 main approaches used in real-world systems.

---

# 1. URL Path Versioning (MOST COMMON)

```http
GET /api/v1/users
GET /api/v2/users
```

---

## Example in ASP.NET Core

```csharp
[ApiController]
[Route("api/v1/users")]
public class UsersV1Controller : ControllerBase
{
}
```

```csharp
[ApiController]
[Route("api/v2/users")]
public class UsersV2Controller : ControllerBase
{
}
```

---

## Advantages

- Very clear
- Easy to debug
- Easy to cache
- Widely used in industry

---

## Disadvantages

- URL duplication
- More controllers to manage

---

# 2. Query String Versioning

```http
GET /api/users?version=1
GET /api/users?version=2
```

---

## Advantages

- Simple to implement
- Single endpoint

---

## Disadvantages

- Not clean REST design
- Harder to cache
- Less commonly used

---

# 3. Header Versioning (Enterprise Standard)

```http
GET /api/users
api-version: 1
```

or

```http
GET /api/users
Accept: application/vnd.company.v1+json
```

---

## Advantages

- Clean URLs
- Flexible
- No URL duplication
- Used in enterprise APIs

---

## Disadvantages

- Harder to test manually
- Slightly complex for beginners

---

# 4. Media Type Versioning (Content Negotiation)

```http
Accept: application/vnd.myapi.v1+json
```

---

## Example Response

```json
{
  "version": "v1",
  "data": []
}
```

---

## Advantages

- REST-compliant
- Very flexible
- Good for public APIs

---

## Disadvantages

- Complex to implement
- Harder for frontend developers

---

# 5. Comparison Table

| Strategy | Example | Readability | Industry Usage |
|----------|--------|-------------|----------------|
| URL Versioning | /v1/users | High | Very High |
| Query String | ?version=1 | Medium | Low |
| Header Versioning | api-version: 1 | High | High |
| Media Type | Accept header | Low | Medium |

---

# 6. Best Practice in Industry

Most real-world systems use:

## ✔ URL Versioning

```text
/api/v1/
/api/v2/
```

Because it is:

- simple
- explicit
- easy to maintain

---

## ✔ Enterprise Systems

Prefer:

```text
Header Versioning
```

because it keeps URLs clean.

---

# 7. Versioning in ASP.NET Core

## Step 1: Install package

```text
Microsoft.AspNetCore.Mvc.Versioning
```

---

## Step 2: Configure versioning

```csharp
services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});
```

---

## Step 3: Use in Controller

```csharp
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/users")]
public class UsersController : ControllerBase
{
}
```

---

# 8. Real Production Scenario

### Version 1 (basic user)

```json
{
  "id": 1,
  "name": "John"
}
```

---

### Version 2 (enhanced user)

```json
{
  "id": 1,
  "fullName": "John Doe",
  "email": "john@email.com",
  "isActive": true
}
```

Both must coexist.

---

# 9. When to Introduce New Version

Create a new version when:

- Breaking changes occur
- Field names change
- Response structure changes
- Business logic changes significantly

---

# 10. When NOT to Create New Version

Avoid versioning for:

- Adding new optional fields
- Bug fixes
- Internal optimizations

Example:

```json
Old:
{ "name": "John" }

New:
{ "name": "John", "age": 30 }
```

✔ No need for new version

---

# 11. Common Mistakes

---

## ❌ 1. Frequent version creation

```text
v1 → v2 → v3 → v4 → v5 (too fast)
```

Problem:
- maintenance nightmare

---

## ❌ 2. Not supporting old versions

Breaking clients causes production failures.

---

## ❌ 3. Mixing versioning styles

```text
/v1/users?version=2
```

Confusing design.

---

# 12. Versioning Strategy in Microservices

Each microservice:

- owns its own versioning
- evolves independently

Example:

```text
Order Service → v1, v2
Payment Service → v1 only
User Service → v1, v2, v3
```

---

# 13. Senior-Level Interview Answer

API versioning is a strategy used to manage changes in Web APIs without breaking existing clients. It allows multiple versions of an API to coexist simultaneously. Common strategies include URL path versioning, query string versioning, header-based versioning, and media type versioning. URL versioning is the most widely used due to simplicity and clarity, while header and media type versioning are preferred in enterprise-grade systems. Versioning should be introduced only for breaking changes and not for backward-compatible enhancements.

---

# 14. Easy Way to Remember

```text
v1 = old contract
v2 = new contract
both must work together
```

Or:

```text
Never break old clients → add new version instead
```