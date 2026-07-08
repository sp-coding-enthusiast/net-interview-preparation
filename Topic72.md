# 72. Idempotency

---

# What is Idempotency?

**Idempotency** means:

> **Performing the same operation multiple times produces the same final result as performing it once.**

In simple words:

- First request → Changes the system
- Second request (same request) → Does **not** change the result further

Think of it like a **light switch**.

If the light is already OFF:

You press OFF again.

Nothing changes.

```
OFF
 ↓
OFF
 ↓
Still OFF
```

The final state remains the same.

That is **idempotent**.

---

# Real-Life Example 1 – Door Lock

Imagine you lock your house.

```
Lock Door
```

Door becomes locked.

Now you press the lock button again.

```
Lock Door
```

Door is still locked.

Nothing changes.

This is idempotent.

---

# Real-Life Example 2 – ATM Balance Inquiry

You check your account balance.

```
Check Balance
```

₹50,000

Check again.

```
Check Balance
```

₹50,000

Checking multiple times doesn't change the balance.

Idempotent.

---

# Real-Life Example 3 – Updating Profile

Suppose your name is

```
Saurabh
```

API request

```
PUT /users/1

{
   "name":"Rahul"
}
```

Now the name becomes

```
Rahul
```

You send the same request 100 times.

Result remains

```
Rahul
```

No further change.

Idempotent.

---

# Non-Idempotent Example

Suppose you deposit ₹100.

```
POST /deposit

100
```

Balance

```
1000
```

↓

1100

Send again.

↓

1200

Send again.

↓

1300

Every request changes the result.

Not idempotent.

---

# HTTP Methods and Idempotency

## GET

Retrieve data only.

```
GET /products
```

Call 1

↓

Returns products

Call 2

↓

Same products

No data changes.

**Idempotent**

---

## PUT

Replace an entire resource.

```
PUT /users/10

{
   "name":"John"
}
```

Call it once.

Name becomes John.

Call it again.

Still John.

**Idempotent**

---

## DELETE

```
DELETE /users/10
```

First call

↓

User deleted.

Second call

↓

User is already deleted.

Final state is still "deleted."

**Idempotent**

*(The second request may return `404 Not Found` or `204 No Content`, but the resource remains deleted.)*

---

## POST

Usually creates new resources.

```
POST /orders
```

First request

↓

Order 101 created.

Second request

↓

Order 102 created.

Different result.

**Not idempotent**

---

## PATCH

Updates only selected fields.

Sometimes idempotent.

Sometimes not.

Example

```
PATCH

{
   "name":"Rahul"
}
```

Run 100 times.

Still Rahul.

Idempotent.

Example

```
PATCH

Increase Salary by 1000
```

Run 3 times.

Salary increases three times.

Not idempotent.

---

# Summary Table

| HTTP Method | Idempotent? | Reason |
|-------------|-------------|--------|
| GET | ✅ Yes | Reads data only |
| PUT | ✅ Yes | Replaces with same value |
| DELETE | ✅ Yes | Resource stays deleted |
| POST | ❌ No | Usually creates new resources |
| PATCH | ⚠ Depends | Depends on the update |

---

# Why is Idempotency Important?

Imagine your internet becomes slow.

You click

```
Pay Now
```

Nothing happens.

You click again.

Without idempotency:

```
₹500 Paid

₹500 Paid Again

₹500 Paid Again
```

Three payments!

Bad experience.

---

With idempotency, the server recognizes that the request has already been processed and avoids creating duplicates.

Many payment systems use an **Idempotency-Key**.

Example

```
POST /payments

Idempotency-Key: 123ABC
```

If the same request with the same key arrives again:

```
Already processed.

Return previous response.
```

No duplicate payment.

---

# ASP.NET Core Example

```csharp
[HttpPut("{id}")]
public IActionResult UpdateUser(int id, User user)
{
    // Replace the existing user with the supplied data
    return Ok();
}
```

Calling this endpoint repeatedly with the same data results in the same final state.

---

# Interview Questions

## What is idempotency?

Idempotency means making the same request multiple times results in the same final state as making it once.

---

## Which HTTP methods are idempotent?

- GET
- PUT
- DELETE

PATCH may or may not be idempotent, depending on the implementation.

POST is generally not idempotent.

---

## Why is idempotency important?

It prevents duplicate operations when requests are retried because of network failures, client retries, or timeouts.

---

# Quick Revision

- Idempotent = Same request → Same final result.
- GET only reads data.
- PUT replaces a resource.
- DELETE keeps the resource deleted.
- POST usually creates new resources and is not idempotent.
- Payment APIs often use Idempotency Keys to avoid duplicate transactions.

---

# 73. API Versioning

---

# What is API Versioning?

API Versioning means:

> **Keeping multiple versions of an API so that existing applications continue to work even when new features or changes are introduced.**

Think of it as releasing new versions of a mobile app.

```
WhatsApp v1

↓

WhatsApp v2

↓

WhatsApp v3
```

People using the older version can still continue using it until they upgrade.

API versioning follows the same idea.

---

# Why Do We Need API Versioning?

Imagine your API returns

```
GET /users/1
```

Version 1 response

```json
{
    "id":1,
    "name":"Saurabh"
}
```

Later you change it to

```json
{
    "id":1,
    "firstName":"Saurabh",
    "lastName":"Purohit"
}
```

Older mobile apps expect the `name` property.

They break.

Versioning prevents this.

---

# Real-Life Example

A restaurant changes its menu.

Old menu

```
Veg Burger
₹100
```

New menu

```
Cheese Veg Burger
₹130
```

Some customers still want the old menu.

Restaurant keeps both menus for a while.

API versioning works similarly.

---

# Common Versioning Strategies

## 1. URL Versioning (Most Popular)

```
/api/v1/users
```

```
/api/v2/users
```

Easy to understand and widely used.

---

## 2. Query String Versioning

```
/api/users?version=1
```

```
/api/users?version=2
```

Simple but less common.

---

## 3. Header Versioning

```
GET /api/users

API-Version: 2
```

The version is passed in an HTTP header.

Keeps URLs clean.

---

## 4. Media Type (Content Negotiation)

```
Accept:

application/json;version=2
```

Less common and more advanced.

---

# Which Strategy is Best?

For most enterprise APIs:

```
/api/v1/users
```

is the easiest to understand and maintain.

---

# ASP.NET Core Example

```csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/users")]
public class UsersController : ControllerBase
{
    [HttpGet]
    public IActionResult GetUsers()
    {
        return Ok();
    }
}
```

Another version

```csharp
[ApiController]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/users")]
public class UsersV2Controller : ControllerBase
{
    [HttpGet]
    public IActionResult GetUsers()
    {
        return Ok(new
        {
            Message = "Version 2"
        });
    }
}
```

---

# Good Versioning Practices

- Introduce a new version only for breaking changes.
- Keep old versions available for a transition period.
- Document differences between versions.
- Communicate deprecation timelines clearly.
- Avoid unnecessary versions for minor, backward-compatible changes.

---

# Versioning Flow

```
Client
   │
GET /api/v1/users
   │
Server
   │
Returns V1 Response
```

```
Client
   │
GET /api/v2/users
   │
Server
   │
Returns V2 Response
```

Both clients can continue working.

---

# Interview Questions

## What is API versioning?

API versioning allows multiple versions of an API to exist simultaneously so existing clients continue working while new functionality is introduced.

---

## Why is API versioning required?

To avoid breaking existing applications when making incompatible API changes.

---

## What are common API versioning techniques?

- URL versioning
- Query string versioning
- Header versioning
- Media type versioning

---

## Which versioning approach is most common?

URL versioning (`/api/v1/...`) because it is simple, explicit, and easy to document.

---

# Quick Revision

- API versioning enables backward compatibility.
- Create a new version only for breaking changes.
- Popular strategies: URL, Query String, Header, Media Type.
- URL versioning is the most common and beginner-friendly approach.
- Maintain older versions until clients have migrated.