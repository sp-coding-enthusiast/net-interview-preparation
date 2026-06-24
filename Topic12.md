# Web API Design — REST Principles (ASP.NET Core / General)

REST (Representational State Transfer) is a **set of architectural principles** for designing scalable and maintainable web APIs.

It is NOT a framework or a protocol — it is a design style.

---

# 1. What REST Really Means

REST is based on the idea:

> “Everything is a resource, and you operate on it using standard HTTP methods.”

Example resources:

```text
/users
/orders
/products
/payments
```

---

# 2. Core REST Principles

## 1. Stateless

Each request must contain all required information.

```text
Server does NOT store client session state
```

### Example:

```http
GET /orders
Authorization: Bearer token123
```

Server does NOT remember previous requests.

---

### Why stateless matters

- scalability (easy horizontal scaling)
- no server memory dependency
- load balancing becomes simple

---

# 2. Client-Server Separation

Client and server are independent.

```text
Frontend (React / Mobile App)
        ↓ HTTP
Backend (ASP.NET Core API)
```

### Benefits:

- independent development
- independent deployment
- better scalability

---

# 3. Resource-Based Design

Everything is a resource (noun-based URLs).

### Good:

```http
GET /users
GET /users/10
POST /orders
```

### Bad:

```http
GET /getUser
POST /createOrder
```

REST prefers **nouns, not verbs**.

---

# 4. Uniform Interface

Uses standard HTTP methods consistently:

| Method | Meaning |
|--------|--------|
| GET | Read data |
| POST | Create |
| PUT | Full update |
| PATCH | Partial update |
| DELETE | Remove |

---

## Example:

```http
GET /products/5
```

```http
POST /products
```

```http
PUT /products/5
```

```http
DELETE /products/5
```

---

# 5. Stateless Communication (Important Interview Point)

Each request is independent.

Example:

```text
Request 1: Login
Request 2: Get Orders
Request 3: Place Order
```

Each request contains authentication token.

---

# 6. Cacheable

Responses should define whether they can be cached.

Example:

```http
Cache-Control: public, max-age=60
```

### Benefits:

- faster responses
- reduced server load

---

# 7. Layered System

Client does NOT know if it is talking to:

- API server
- load balancer
- gateway
- proxy

Example:

```text
Client → API Gateway → Microservice → Database
```

---

# 8. Code on Demand (Optional)

Server can send executable code (rare in APIs).

Example:

- JavaScript to browser

Not commonly used in modern APIs.

---

# 3. REST Resource Naming Best Practices

## Good API Design

```http
GET /users
GET /users/10
GET /users/10/orders
```

---

## Bad API Design

```http
GET /getAllUsers
GET /fetchUserById
POST /createNewUser
```

---

# 4. HTTP Status Codes (Very Important)

| Code | Meaning |
|------|--------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Server Error |

---

## Example:

### Success

```http
POST /users
Response: 201 Created
```

---

### Not found

```http
GET /users/999
Response: 404 Not Found
```

---

# 5. REST API Design in ASP.NET Core

Example Controller:

```csharp
[ApiController]
[Route("api/[controller]")]
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
    public IActionResult CreateUser()
    {
        return Created();
    }

    [HttpPut("{id}")]
    public IActionResult UpdateUser(int id)
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

# 6. REST vs SOAP (Interview Favorite)

| Feature | REST | SOAP |
|----------|------|------|
| Format | JSON | XML |
| Speed | Fast | Slow |
| Complexity | Simple | Complex |
| Usage | Modern APIs | Legacy systems |

---

# 7. Common REST Mistakes

## ❌ 1. Using verbs in URLs

```http
/getUser
/deleteOrder
```

---

## ❌ 2. Wrong status codes

Always return correct HTTP status.

---

## ❌ 3. Not versioning APIs

Bad:

```http
/api/users
```

Better:

```http
/api/v1/users
```

---

## ❌ 4. Over-fetching data

Returning unnecessary fields increases payload size.

---

## ❌ 5. Not handling errors consistently

Bad:

```json
{ "error": "something failed" }
```

Good:

```json
{
  "error": "Validation failed",
  "code": 400,
  "traceId": "abc123"
}
```

---

# 8. REST API Design Best Practices

- Use nouns for resources
- Use correct HTTP methods
- Use proper status codes
- Keep APIs stateless
- Version your APIs
- Use pagination for large datasets

Example:

```http
GET /users?page=1&size=10
```

---

# 9. Pagination Example

```http
GET /products?page=2&limit=20
```

Response:

```json
{
  "page": 2,
  "limit": 20,
  "total": 200,
  "data": []
}
```

---

# 10. Filtering & Sorting

```http
GET /orders?status=paid&sort=desc
```

---

# 11. Senior-Level Interview Insight

A well-designed REST API should be:

- stateless
- resource-oriented
- consistent in naming
- properly versioned
- resilient with correct status codes
- optimized for performance (pagination, caching)

---

# 12. Real Production Example Flow

```text
Client → API Gateway → Auth → Controller → Service → DB
```

Each layer follows REST principles and remains stateless.

---

# 13. Senior-Level Interview Answer

REST is an architectural style for designing networked applications where everything is treated as a resource identified by a URI, and interactions are performed using standard HTTP methods. It enforces stateless communication, uniform interface, client-server separation, cacheability, and a layered system. REST APIs should use nouns for resources, proper HTTP verbs for operations, correct status codes, and remain stateless to ensure scalability and maintainability in distributed systems.

---

# 14. Easy Way to Remember REST

```text
Nouns = Resources (/users)
Verbs = HTTP Methods (GET, POST, PUT, DELETE)
No memory = Stateless
```