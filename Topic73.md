# 74. Model Binding

---

# What is Model Binding?

**Model Binding** is the process by which **ASP.NET Core automatically takes data from an HTTP request and converts it into C# objects, variables, or models.**

In simple words:

> **The framework reads the incoming request and fills your C# objects automatically.**

You don't have to manually read JSON, query strings, or form data.

---

# Real-Life Analogy

Imagine you're filling out a **bank account opening form**.

You write:

```
Name : Saurabh
Age  : 34
City : Bengaluru
```

The bank employee copies your information into the bank's computer.

You don't manually type into the bank's database.

The employee transfers the data.

**Model Binding works exactly like that employee.**

```
Request Data
      │
      ▼
Model Binder
      │
      ▼
C# Object
```

---

# Why Do We Need Model Binding?

Without Model Binding, you'd have to manually extract every value.

Example (conceptually):

```
Read Name

Read Age

Convert Age to integer

Create User object

Assign values
```

Lots of repetitive work.

Model Binding does it automatically.

---

# Example Request

Suppose a client sends:

```
POST /users
```

Request Body

```json
{
    "name": "Saurabh",
    "age": 34
}
```

ASP.NET Core automatically creates:

```csharp
public class User
{
    public string Name { get; set; }

    public int Age { get; set; }
}
```

and fills it as:

```text
User
 ├── Name = "Saurabh"
 └── Age = 34
```

---

# ASP.NET Core Example

```csharp
[HttpPost]
public IActionResult CreateUser(User user)
{
    return Ok(user);
}
```

The `user` parameter is automatically populated from the JSON request body.

You don't need to parse the JSON yourself.

---

# Where Can Model Binding Read Data From?

## 1. Route Values

URL

```
GET /users/10
```

Controller

```csharp
[HttpGet("{id}")]
public IActionResult GetUser(int id)
{
    return Ok(id);
}
```

Result

```
id = 10
```

---

## 2. Query String

Request

```
GET /users?page=2
```

Controller

```csharp
public IActionResult GetUsers(int page)
```

Result

```
page = 2
```

---

## 3. Request Body (JSON)

Request

```json
{
    "name": "Rahul",
    "age": 30
}
```

Controller

```csharp
public IActionResult Create(User user)
```

Result

```
User.Name = Rahul

User.Age = 30
```

---

## 4. Form Data

HTML Form

```
Name = Amit

Age = 25
```

Model Binder creates

```text
User
 ├── Name = Amit
 └── Age = 25
```

---

## 5. Headers

Request

```
Authorization: Bearer xyz
```

Controller

```csharp
public IActionResult Get(
    [FromHeader] string authorization)
```

Result

```
authorization = Bearer xyz
```

---

# Binding Source Attributes

Sometimes you explicitly tell ASP.NET Core where to read data from.

```csharp
[FromRoute]
```

Reads from URL route.

---

```csharp
[FromQuery]
```

Reads from query string.

---

```csharp
[FromBody]
```

Reads JSON request body.

---

```csharp
[FromHeader]
```

Reads HTTP headers.

---

```csharp
[FromForm]
```

Reads HTML form data.

---

# Complete Example

Request

```
POST /users/10?country=India
```

Headers

```
Authorization: Bearer xyz
```

Body

```json
{
    "name": "Saurabh",
    "age": 34
}
```

Controller

```csharp
[HttpPost("{id}")]
public IActionResult Save(
    [FromRoute] int id,
    [FromQuery] string country,
    [FromHeader] string authorization,
    [FromBody] User user)
{
    return Ok();
}
```

Model Binding creates:

```
id = 10

country = India

authorization = Bearer xyz

user.Name = Saurabh

user.Age = 34
```

---

# Model Binding Process

```
HTTP Request
      │
      ▼
Model Binder
      │
Reads Route
Reads Query
Reads Headers
Reads Body
Reads Form
      │
      ▼
Creates C# Object
      │
      ▼
Controller Action
```

---

# Interview Questions

## What is Model Binding?

Model Binding is the process by which ASP.NET Core automatically maps incoming request data to action parameters or C# model objects.

---

## What sources does Model Binding use?

- Route values
- Query string
- Request body
- Form data
- Headers

---

## Why is Model Binding useful?

It eliminates manual parsing of request data, reduces boilerplate code, and improves readability.

---

# Quick Revision

- Converts HTTP request data into C# objects.
- Supports Route, Query, Body, Form, and Headers.
- Happens automatically before the controller action executes.
- Can be customized using attributes like `[FromBody]` and `[FromQuery]`.

---

# 75. Validation

---

# What is Validation?

**Validation** is the process of checking whether the incoming data is correct before processing it.

In simple words:

> **Validation ensures users send valid and complete information.**

---

# Real-Life Analogy

Imagine you're applying for a passport.

The officer checks:

- Is the name provided?
- Is the age valid?
- Is the date of birth correct?
- Is the photo attached?

If something is missing:

```
Application Rejected
```

The application isn't processed until the required information is valid.

API validation works the same way.

---

# Why Do We Need Validation?

Suppose a client sends:

```json
{
    "name": "",
    "age": -10
}
```

Without validation:

```
Invalid data gets stored.
```

This leads to bad data in the database.

Validation prevents this.

---

# Example Model

```csharp
using System.ComponentModel.DataAnnotations;

public class User
{
    [Required]
    public string Name { get; set; }

    [Range(18, 60)]
    public int Age { get; set; }

    [EmailAddress]
    public string Email { get; set; }
}
```

---

# Common Validation Attributes

## Required

```csharp
[Required]
```

Value cannot be empty.

---

## StringLength

```csharp
[StringLength(50)]
```

Maximum length is 50 characters.

---

## MinLength

```csharp
[MinLength(5)]
```

Minimum number of characters.

---

## MaxLength

```csharp
[MaxLength(100)]
```

Maximum number of characters.

---

## Range

```csharp
[Range(1,100)]
```

Numeric value must be between 1 and 100.

---

## EmailAddress

```csharp
[EmailAddress]
```

Must be a valid email format.

---

## Phone

```csharp
[Phone]
```

Validates phone number format.

---

## Url

```csharp
[Url]
```

Checks whether the value is a valid URL.

---

## RegularExpression

```csharp
[RegularExpression(@"^[A-Z]{3}[0-9]{4}$")]
```

Validates custom patterns.

---

# Controller Example

```csharp
[HttpPost]
public IActionResult Create(User user)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    return Ok(user);
}
```

If validation fails:

```
400 Bad Request
```

is returned.

---

# Validation Flow

```
Client
      │
Sends Request
      │
      ▼
Model Binding
      │
Creates User Object
      │
      ▼
Validation
      │
 ┌───────────────┐
 │               │
Valid         Invalid
 │               │
 ▼               ▼
Controller   400 Bad Request
```

---

# Example

Request

```json
{
    "name": "",
    "age": 10,
    "email": "invalid"
}
```

Validation results:

- Name → Required ❌
- Age → Must be between 18 and 60 ❌
- Email → Invalid format ❌

Response

```json
{
    "errors": {
        "Name": [
            "The Name field is required."
        ],
        "Age": [
            "The field Age must be between 18 and 60."
        ],
        "Email": [
            "The Email field is not a valid e-mail address."
        ]
    }
}
```

---

# Validation vs Model Binding

| Model Binding | Validation |
|--------------|------------|
| Creates C# objects from request data | Checks whether the data is valid |
| Reads Route, Query, Body, Form, Headers | Checks rules like Required, Range, Email |
| Happens first | Happens after Model Binding |
| Converts data | Verifies data |

---

# Best Practices

- Validate all user input.
- Use Data Annotation attributes for common rules.
- Return `400 Bad Request` for invalid input.
- Keep business validation separate from simple field validation where appropriate.
- Never trust client-side validation alone; always validate on the server.

---

# Interview Questions

## What is validation?

Validation verifies that incoming request data satisfies the expected rules before processing.

---

## What is `ModelState`?

`ModelState` stores the results of model binding and validation. It indicates whether the incoming request is valid.

---

## What happens if validation fails?

ASP.NET Core returns validation errors. With the `[ApiController]` attribute, invalid models automatically result in a **400 Bad Request** response unless customized.

---

## Difference between Model Binding and Validation?

Model Binding converts request data into C# objects, while Validation checks whether those objects satisfy defined rules.

---

# Quick Revision

- Validation ensures request data is correct.
- Uses Data Annotation attributes like `[Required]`, `[Range]`, and `[EmailAddress]`.
- Runs after Model Binding.
- Invalid input typically returns `400 Bad Request`.
- `ModelState` contains validation results.
- `[ApiController]` can automatically return validation errors without manually checking `ModelState.IsValid`.