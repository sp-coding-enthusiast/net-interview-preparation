# 85. HATEOAS (Hypermedia As The Engine Of Application State)

---

# What is HATEOAS?

**HATEOAS (Hypermedia As The Engine Of Application State)** is one of the **REST architectural principles**.

It means:

> **The server not only sends data but also sends links (actions) that tell the client what it can do next.**

Instead of the client hardcoding API URLs, the server guides the client by providing hyperlinks.

---

# Simple Definition

Normally, an API returns only data.

Example

```json
{
    "id": 101,
    "name": "Laptop",
    "price": 50000
}
```

With HATEOAS, the API returns both the data **and the available actions**.

```json
{
    "id": 101,
    "name": "Laptop",
    "price": 50000,

    "links": [
        {
            "rel": "self",
            "href": "/api/products/101",
            "method": "GET"
        },
        {
            "rel": "update",
            "href": "/api/products/101",
            "method": "PUT"
        },
        {
            "rel": "delete",
            "href": "/api/products/101",
            "method": "DELETE"
        }
    ]
}
```

The response tells the client:

- Where the resource is
- What operations are available
- Which HTTP methods to use

---

# Real-Life Analogy

Imagine visiting a shopping mall.

Instead of just handing you a map, the mall also places signboards everywhere.

```
Food Court →

Parking →

Exit →

Restrooms →
```

You don't need to memorize the building layout.

You simply follow the signs.

HATEOAS works the same way.

The API response contains "signboards" (links) that guide the client.

---

# Without HATEOAS

The client must know all URLs in advance.

```
Client

↓

GET /products/101

↓

Server

↓

Product Data
```

To update the product, the client must already know:

```
PUT /products/101
```

To delete it:

```
DELETE /products/101
```

Everything is hardcoded into the client.

---

# With HATEOAS

The server tells the client what can be done next.

```
Client

↓

GET /products/101

↓

Server

↓

Product Data

+

Available Links
```

The client simply follows the provided links.

---

# Example

Request

```
GET /api/orders/500
```

Response

```json
{
    "orderId": 500,
    "status": "Pending",

    "links": [
        {
            "rel": "self",
            "href": "/api/orders/500",
            "method": "GET"
        },
        {
            "rel": "cancel",
            "href": "/api/orders/500/cancel",
            "method": "POST"
        },
        {
            "rel": "payment",
            "href": "/api/orders/500/payment",
            "method": "POST"
        }
    ]
}
```

The client knows:

- View order
- Cancel order
- Make payment

without hardcoding those endpoints.

---

# Understanding the Link Properties

Example

```json
{
    "rel": "update",
    "href": "/api/products/101",
    "method": "PUT"
}
```

### rel

Relationship.

Examples

```
self

update

delete

next

previous

payment

cancel
```

---

### href

The URL to call.

```
/api/products/101
```

---

### method

The HTTP method to use.

```
GET

POST

PUT

DELETE
```

---

# Why Use HATEOAS?

Suppose Version 1 API uses

```
/products/101
```

Later Version 2 changes it to

```
/items/101
```

Without HATEOAS

Every client must be updated.

With HATEOAS

The server returns

```json
{
    "links": [
        {
            "href": "/items/101"
        }
    ]
}
```

Clients simply follow the new link.

No hardcoded URLs.

---

# ASP.NET Core Example

Create a simple model.

```csharp
public class Link
{
    public string Rel { get; set; }

    public string Href { get; set; }

    public string Method { get; set; }
}
```

Controller

```csharp
[HttpGet("{id}")]
public IActionResult GetProduct(int id)
{
    var product = new
    {
        Id = id,
        Name = "Laptop",
        Price = 50000,

        Links = new[]
        {
            new Link
            {
                Rel = "self",
                Href = $"/api/products/{id}",
                Method = "GET"
            },
            new Link
            {
                Rel = "update",
                Href = $"/api/products/{id}",
                Method = "PUT"
            },
            new Link
            {
                Rel = "delete",
                Href = $"/api/products/{id}",
                Method = "DELETE"
            }
        }
    };

    return Ok(product);
}
```

Response

```json
{
    "id": 101,
    "name": "Laptop",
    "price": 50000,

    "links": [
        {
            "rel": "self",
            "href": "/api/products/101",
            "method": "GET"
        },
        {
            "rel": "update",
            "href": "/api/products/101",
            "method": "PUT"
        },
        {
            "rel": "delete",
            "href": "/api/products/101",
            "method": "DELETE"
        }
    ]
}
```

---

# HATEOAS Flow

```
Client

↓

GET /products/101

↓

Server

↓

Product

+

Available Links

↓

Client follows links

↓

Update

Delete

View

Purchase
```

---

# Advantages

- Reduces hardcoded URLs.
- Makes APIs more self-discoverable.
- Helps clients navigate dynamically.
- Supports API evolution with fewer client changes.
- Improves API discoverability.

---

# Disadvantages

- Increases response size.
- More complex to implement.
- Many public APIs don't fully implement it.
- Clients must understand hypermedia links.

---

# Is HATEOAS Mandatory?

No.

Although HATEOAS is part of the original REST constraints, **many modern REST APIs do not implement it fully**.

Popular APIs such as many cloud provider APIs and microservice APIs often use REST principles but omit HATEOAS for simplicity.

---

# HATEOAS vs Normal REST

| Normal REST | HATEOAS REST |
|--------------|--------------|
| Returns only data | Returns data + hyperlinks |
| Client hardcodes URLs | Client follows links provided by the server |
| Lower response size | Slightly larger responses |
| Simpler implementation | More dynamic and self-descriptive |

---

# Real-World Examples

### GitHub API

Returns links for related resources and pagination.

Example:

```json
{
    "url": "...",
    "comments_url": "...",
    "commits_url": "...",
    "issues_url": "..."
}
```

---

### Pagination

Instead of requiring the client to construct URLs, the API provides links such as:

```json
{
    "links": {
        "self": "/products?page=2",
        "next": "/products?page=3",
        "previous": "/products?page=1"
    }
}
```

The client simply follows the appropriate link.

---

# Interview Questions

## What is HATEOAS?

HATEOAS (Hypermedia As The Engine Of Application State) is a REST constraint where the server returns hyperlinks along with resource data so that clients can discover available actions dynamically.

---

## Why is HATEOAS used?

It reduces hardcoded API URLs, improves discoverability, and allows clients to navigate the API using server-provided links.

---

## What information does a HATEOAS link contain?

Typically:

- `rel` – relationship (such as `self` or `update`)
- `href` – target URL
- `method` – HTTP method to use

---

## Is HATEOAS mandatory in REST?

It is one of the REST architectural constraints, but many modern REST APIs choose not to implement it fully because of additional complexity.

---

## Name some common relationship (`rel`) values.

- `self`
- `next`
- `previous`
- `update`
- `delete`
- `create`
- `payment`
- `cancel`

---

# Quick Revision

- HATEOAS stands for **Hypermedia As The Engine Of Application State**.
- The server returns **data + hyperlinks**.
- Clients discover available actions instead of hardcoding URLs.
- Links usually include `rel`, `href`, and `method`.
- Improves API discoverability and flexibility.
- Not widely implemented in many modern REST APIs, but it remains an important REST concept for interviews.