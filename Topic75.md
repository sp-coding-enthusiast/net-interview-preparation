# 81. IActionResult vs ActionResult

---

# Introduction

One of the most common ASP.NET Core interview questions is:

> **What is the difference between `IActionResult` and `ActionResult`?**

Many developers think they are the same because both are used as return types for controller actions.

They are related, but they are **not identical**.

Understanding the difference helps you write cleaner APIs and choose the appropriate return type.

---

# What is IActionResult?

`IActionResult` is an **interface**.

It represents **any HTTP response** that can be returned from a controller action.

Think of it as a **contract**.

It says:

> "Any class implementing me can be returned as an HTTP response."

---

## Real-Life Analogy

Imagine a **remote control**.

A remote defines buttons like:

- Power
- Volume
- Channel

It doesn't actually play TV channels.

Different televisions implement those buttons differently.

Similarly,

`IActionResult` only defines the contract.

Actual result classes implement it.

---

# Examples of IActionResult Implementations

Many built-in result types implement `IActionResult`.

```
OkResult

OkObjectResult

BadRequestResult

BadRequestObjectResult

NotFoundResult

NotFoundObjectResult

CreatedResult

CreatedAtActionResult

NoContentResult

UnauthorizedResult

ForbidResult

FileResult

RedirectResult
```

When you write:

```csharp
return Ok();
```

You're actually returning an `OkResult`, which implements `IActionResult`.

---

# Example

```csharp
[HttpGet]
public IActionResult Get()
{
    return Ok("Hello");
}
```

Possible responses:

```csharp
return Ok();
```

or

```csharp
return BadRequest();
```

or

```csharp
return NotFound();
```

or

```csharp
return Unauthorized();
```

All are valid because they implement `IActionResult`.

---

# What is ActionResult?

`ActionResult` is an **abstract class**.

It also represents an HTTP response.

Unlike `IActionResult`, it provides a **base implementation** that other result classes inherit from.

---

## Simplified Class Hierarchy

```
                IActionResult
                      ▲
                      │
               ActionResult
                      ▲
      ┌───────────────┼───────────────┐
      │               │               │
   OkResult     NotFoundResult   BadRequestResult
```

`ActionResult` itself implements `IActionResult`.

So:

```
ActionResult

IS-A

IActionResult
```

But:

```
IActionResult

IS NOT

ActionResult
```

because one is an interface and the other is a class.

---

# Real-Life Analogy

Imagine

```
Vehicle (Interface)

↓

Car (Abstract Base)

↓

BMW

↓

Audi
```

The interface defines the contract.

The abstract class provides common functionality.

Concrete classes inherit from it.

---

# Example

```csharp
public ActionResult Get()
{
    return Ok();
}
```

This works because `OkResult` derives from `ActionResult`.

---

# IActionResult vs ActionResult

| IActionResult | ActionResult |
|---------------|--------------|
| Interface | Abstract class |
| Defines a contract | Provides a base implementation |
| Cannot contain implementation | Can contain implementation |
| More flexible | Slightly more specific |
| Implemented by `ActionResult` | Implements `IActionResult` |

---

# Returning Different Responses

Suppose an API may return:

Success

```csharp
return Ok(user);
```

Not Found

```csharp
return NotFound();
```

Validation Error

```csharp
return BadRequest();
```

Both return types work.

Using `IActionResult`

```csharp
public IActionResult Get(int id)
{
    if (id <= 0)
        return BadRequest();

    return Ok();
}
```

Using `ActionResult`

```csharp
public ActionResult Get(int id)
{
    if (id <= 0)
        return BadRequest();

    return Ok();
}
```

---

# Generic ActionResult<T>

ASP.NET Core introduced a better option:

```
ActionResult<T>
```

This combines

- Strong typing
- Multiple HTTP responses

---

## Example

```csharp
[HttpGet("{id}")]
public ActionResult<User> Get(int id)
{
    var user = FindUser(id);

    if (user == null)
        return NotFound();

    return user;
}
```

Notice:

```csharp
return user;
```

No need to write:

```csharp
return Ok(user);
```

ASP.NET Core automatically wraps the object in an `OkObjectResult` (HTTP 200).

---

# Why ActionResult<T> is Better

Without generic version

```csharp
public IActionResult Get()
{
    return Ok(user);
}
```

Swagger cannot always infer the response type automatically.

With

```csharp
public ActionResult<User> Get()
```

Swagger knows

```
200

returns User
```

Better documentation.

Better IntelliSense.

Better compile-time safety.

---

# IActionResult Example

```csharp
[HttpGet("{id}")]
public IActionResult Get(int id)
{
    var user = repository.Find(id);

    if (user == null)
        return NotFound();

    return Ok(user);
}
```

Possible responses

```
200 OK

404 Not Found

400 Bad Request

401 Unauthorized
```

---

# ActionResult<T> Example

```csharp
[HttpGet("{id}")]
public ActionResult<User> Get(int id)
{
    var user = repository.Find(id);

    if (user == null)
        return NotFound();

    return user;
}
```

Cleaner.

More expressive.

---

# Which One Should You Use?

### Use `IActionResult`

When your action returns **many different types of responses** and you don't want to specify a success payload type.

Example

```csharp
return Ok();

return BadRequest();

return Unauthorized();

return NotFound();
```

---

### Use `ActionResult<T>`

When your action normally returns a specific model but may also return error responses.

Example

```csharp
public ActionResult<User> Get(int id)
```

Success

```
User
```

Failure

```
404

400

401
```

This is the recommended approach for most Web APIs.

---

# Common Mistake

Wrong

```csharp
public User Get()
```

If an error occurs

```
How do you return 404?
```

You cannot easily return different HTTP status codes.

Instead

```csharp
public ActionResult<User> Get()
```

Now you can return

```csharp
return NotFound();
```

or

```csharp
return user;
```

---

# Complete Comparison

| Feature | IActionResult | ActionResult | ActionResult<T> |
|----------|---------------|--------------|-----------------|
| Type | Interface | Abstract class | Generic abstract class |
| Can return HTTP responses | ✅ | ✅ | ✅ |
| Strongly typed success response | ❌ | ❌ | ✅ |
| Supports multiple status codes | ✅ | ✅ | ✅ |
| Best for Web APIs | Good | Good | **Best** |

---

# Execution Example

```
Client

↓

GET /users/10

↓

Controller

↓

User Found?

        Yes
         │
         ▼
Return User

↓

200 OK

------------------

User Not Found?

↓

Return NotFound()

↓

404
```

---

# Interview Questions

## What is IActionResult?

`IActionResult` is an interface representing an HTTP response returned by a controller action.

---

## What is ActionResult?

`ActionResult` is an abstract base class that implements `IActionResult` and serves as the foundation for built-in result types.

---

## What is ActionResult<T>?

`ActionResult<T>` is a generic return type that allows a controller action to return either a strongly typed object (`T`) or an HTTP error response like `NotFound()` or `BadRequest()`.

---

## Which return type is recommended for Web APIs?

`ActionResult<T>` is generally recommended because it provides strong typing, better API documentation (such as Swagger/OpenAPI), and still allows returning different HTTP status codes.

---

## Can IActionResult return multiple response types?

Yes. It can return any object that implements `IActionResult`, such as `Ok()`, `BadRequest()`, `NotFound()`, `Created()`, or `Unauthorized()`.

---

# Quick Revision

- `IActionResult` is an **interface**.
- `ActionResult` is an **abstract class** that implements `IActionResult`.
- `ActionResult<T>` is the preferred choice for most Web APIs because it combines strong typing with flexible HTTP responses.
- Use `IActionResult` when there is no single success payload type.
- Use `ActionResult<T>` when the API usually returns a specific model but may also return error responses.