# 71. REST Principles (Explained in Simple Layman Terms)

---

# What is REST?

**REST (Representational State Transfer)** is a set of rules for designing Web APIs so that different applications can communicate over the internet in a simple, predictable, and standard way.

Think of REST as **the rules followed by a restaurant waiter**.

- Customer = Client (Browser/Mobile App)
- Waiter = API
- Kitchen = Server
- Food = Data

The customer never enters the kitchen.

Instead:

1. Customer tells the waiter what they need.
2. Waiter takes the request.
3. Kitchen prepares it.
4. Waiter returns the food.

Exactly how a REST API works.

```
Client
   │
Request
   │
   ▼
REST API
   │
Business Logic
   │
Database
   │
Response
   ▼
Client
```

---

# Real Life Example

Imagine you're using Swiggy.

You open the app.

You search for Pizza.

The app sends

```
GET /restaurants/pizza
```

The server replies

```
List of pizza restaurants
```

You select one pizza.

The app sends

```
GET /pizza/101
```

The server replies

```
Pizza Details
```

You order it.

The app sends

```
POST /orders
```

Server creates the order.

Everything follows REST principles.

---

# Why REST?

Without rules...

One API may use

```
/GetUserInfo
```

Another may use

```
/FetchCustomer
```

Another

```
/LoadPerson
```

Very confusing.

REST standardizes everything.

Everyone follows similar patterns.

---

# REST Principles

There are six major REST principles.

---

# Principle 1 — Client-Server Architecture

## Meaning

Client and server have separate responsibilities.

Client

- Displays UI
- Takes user input

Server

- Processes requests
- Stores data
- Business logic

---

## Example

Instagram App

Client

```
Shows photos
```

Server

```
Stores photos
```

The mobile app never accesses the database directly.

It only communicates through APIs.

---

### Analogy

Restaurant

Customer

```
Orders food
```

Kitchen

```
Cooks food
```

Customer doesn't enter the kitchen.

---

# Principle 2 — Stateless

This is the MOST IMPORTANT REST principle.

---

## Meaning

Every request must contain everything needed to process it.

The server does NOT remember previous requests.

---

### Wrong Way

Customer says

```
Remember my last order.
```

Waiter forgets.

---

### Correct Way

Customer always says

```
I want one Veg Pizza.
Table 4.
Extra Cheese.
```

Every request has complete information.

---

## API Example

Request 1

```
GET /orders
Authorization: Bearer xyz
```

Request 2

```
GET /profile
Authorization: Bearer xyz
```

Request 3

```
POST /cart
Authorization: Bearer xyz
```

Every request contains the token.

The server never says

```
I remember you.
```

---

### Why?

Because requests may reach different servers.

```
Load Balancer

      │
 ┌────┴────┐
 │         │
Server1  Server2
```

Request 1

↓

Server 1

Request 2

↓

Server 2

Both must work independently.

---

# Principle 3 — Cacheable

Some responses can be stored temporarily.

This avoids repeatedly asking the server.

---

Example

Company Logo

It rarely changes.

Instead of downloading

```
100 times
```

Browser stores it.

Next time

```
Use cached version.
```

Much faster.

---

### Example

```
GET /countries
```

Countries don't change often.

Can be cached.

---

Cannot cache

```
Bank Balance
```

Because balance changes.

---

# Principle 4 — Uniform Interface

Every API should follow the same conventions.

---

Example

Good REST APIs

```
GET /users
```

```
POST /users
```

```
PUT /users/10
```

```
DELETE /users/10
```

Easy to understand.

---

Bad Example

```
/LoadAllUsers
```

```
/CreateNewCustomerRecord
```

```
/RemoveUserNow
```

Every developer invents different naming.

REST avoids this.

---

# Principle 5 — Layered System

Client doesn't know how many servers process the request.

Example

```
Client
   │
API Gateway
   │
Load Balancer
   │
Authentication Server
   │
Business Server
   │
Database
```

Client only knows

```
Call API
```

What happens internally is hidden.

---

### Example

When you visit

Amazon

You don't know whether

- CDN served image
- Cache served response
- Database served response
- Microservice served response

Everything is hidden.

---

# Principle 6 — Code on Demand (Optional)

Server can send executable code.

Mostly JavaScript.

Example

You open Gmail.

Server sends

```
HTML
CSS
JavaScript
```

Browser executes JavaScript.

This principle is optional.

Most REST APIs don't use it.

---

# REST Resources

Everything is treated as a resource.

Examples

```
User

Product

Order

Student

Employee

Book

Movie
```

Each resource has a unique URL.

Example

```
/users
```

```
/users/5
```

```
/products
```

```
/orders
```

---

# HTTP Methods in REST

## GET

Retrieve data.

Example

```
GET /products
```

Returns

```
All products
```

---

## POST

Create new data.

Example

```
POST /products
```

Creates

```
New Product
```

---

## PUT

Replace an entire resource.

Example

```
PUT /products/15
```

Updates all fields.

---

## PATCH

Update only some fields.

Example

```
PATCH /products/15
```

Update

```
Price only
```

---

## DELETE

Remove resource.

Example

```
DELETE /products/15
```

Deletes product.

---

# REST URL Examples

Good

```
GET /users
```

```
GET /users/10
```

```
POST /users
```

```
PUT /users/10
```

```
DELETE /users/10
```

Bad

```
/GetAllUsers
```

```
/DeleteUser
```

```
/UpdateEmployeeDataNow
```

REST prefers nouns, not verbs.

---

# Example Flow

Create User

```
POST /users
```

↓

Response

```
201 Created
```

---

Read User

```
GET /users/10
```

↓

```
200 OK
```

---

Update User

```
PUT /users/10
```

↓

```
200 OK
```

---

Delete User

```
DELETE /users/10
```

↓

```
204 No Content
```

---

# ASP.NET Core Example

```csharp
[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase
{
    [HttpGet]
    public IActionResult GetUsers()
    {
        return Ok();
    }

    [HttpGet("{id}")]
    public IActionResult GetUser(int id)
    {
        return Ok();
    }

    [HttpPost]
    public IActionResult CreateUser(User user)
    {
        return Created("", user);
    }

    [HttpPut("{id}")]
    public IActionResult UpdateUser(int id, User user)
    {
        return Ok();
    }

    [HttpDelete("{id}")]
    public IActionResult DeleteUser(int id)
    {
        return NoContent();
    }
}
```

---

# Why Companies Love REST

Because it is

- Simple
- Language independent
- Platform independent
- Lightweight
- Easy to cache
- Easy to scale
- Easy to maintain
- Works over HTTP
- Supported by almost every programming language

---

# REST vs Real Life

| Real Life | REST |
|------------|------|
| Restaurant Menu | API Documentation |
| Customer | Client |
| Waiter | API |
| Kitchen | Server |
| Food | Data |
| Order Slip | HTTP Request |
| Delivered Food | HTTP Response |
| Table Number | Resource URL |
| Receipt | Status Code |

---

# Interview Questions

## What is REST?

REST is an architectural style for designing web APIs that communicate over HTTP using resources, standard HTTP methods, and stateless communication.

---

## Why is REST stateless?

Each request contains all the information needed to process it. The server does not store client session information between requests, making applications easier to scale and more reliable.

---

## Why are nouns preferred in REST URLs?

Resources are represented as nouns (such as `/users` or `/orders`), while actions are represented by HTTP methods (`GET`, `POST`, `PUT`, `DELETE`). This keeps APIs consistent and intuitive.

---

## What is a resource?

A resource is any object or data exposed by the API, such as a user, product, order, or employee, identified by a unique URL.

---

## What are the six REST principles?

1. Client-Server Architecture
2. Stateless Communication
3. Cacheable Responses
4. Uniform Interface
5. Layered System
6. Code on Demand (Optional)

---

# Quick Revision

- REST is a set of rules for designing web APIs.
- Resources are identified using URLs.
- Use nouns in URLs and HTTP methods for actions.
- Communication is stateless.
- Responses can be cached when appropriate.
- Clients and servers have separate responsibilities.
- Internal server architecture is hidden from clients.
- Code on Demand is optional and rarely used in modern REST APIs.
- REST APIs are simple, scalable, and widely adopted.