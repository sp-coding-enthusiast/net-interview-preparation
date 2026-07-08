# 86. Problem Details (RFC 7807)

---

# What is Problem Details?

**Problem Details** is a **standard format for returning error responses** in REST APIs.

Instead of every API returning different error formats,

everyone follows a common structure.

It is defined by **RFC 7807**.

---

# Why Do We Need It?

Imagine three APIs return errors differently.

API 1

```json
{
   "message":"User not found"
}
```

API 2

```json
{
   "error":"Invalid User"
}
```

API 3

```json
{
   "status":"Failed"
}
```

Every API has a different structure.

Clients become difficult to build.

Problem Details standardizes the response.

---

# Real-Life Analogy

Imagine every traffic signal had different colors.

One city

```
Red = Stop
```

Another city

```
Blue = Stop
```

Another city

```
Yellow = Stop
```

Driving would become confusing.

Standard traffic rules solve this.

Problem Details does the same for API errors.

---

# Standard Problem Details Response

```json
{
    "type": "https://example.com/errors/not-found",
    "title": "User Not Found",
    "status": 404,
    "detail": "User with ID 15 does not exist.",
    "instance": "/users/15"
}
```

---

# Understanding Each Property

## type

A URI identifying the type of error.

```
https://example.com/errors/not-found
```

---

## title

Short summary.

```
User Not Found
```

---

## status

HTTP Status Code.

```
404
```

---

## detail

Detailed explanation.

```
User with ID 15 does not exist.
```

---

## instance

Which request caused the error.

```
/users/15
```

---

# ASP.NET Core Example

```csharp
return Problem(
    title: "User Not Found",
    detail: "User with ID 10 does not exist.",
    statusCode: 404);
```

Response

```json
{
    "type":"https://tools.ietf.org/html/rfc9110",
    "title":"User Not Found",
    "status":404,
    "detail":"User with ID 10 does not exist."
}
```

---

# Validation Errors

When using `[ApiController]`

Invalid requests automatically return Problem Details.

Example

```json
{
    "errors": {
        "Name": [
            "The Name field is required."
        ]
    }
}
```

---

# Benefits

- Standard format
- Easy for frontend developers
- Better debugging
- Consistent APIs
- Supported by ASP.NET Core

---

# Interview Questions

## What is Problem Details?

Problem Details is a standardized JSON format for representing API errors defined by RFC 7807.

---

## Why use Problem Details?

To provide consistent, machine-readable error responses across APIs.

---

# Quick Revision

- Standard API error format.
- Contains `type`, `title`, `status`, `detail`, and `instance`.
- Built into ASP.NET Core.
- Improves consistency and client integration.

---

# 87. Content Negotiation

---

# What is Content Negotiation?

Content Negotiation is the process by which the **client tells the server what response format it prefers**, and the server returns the best supported format.

---

# Real-Life Analogy

Imagine ordering a book.

You ask:

```
English Version
```

Store gives English.

Another customer asks

```
Hindi Version
```

Store gives Hindi.

Same book.

Different formats.

Similarly,

the same API data can be returned in different representations.

---

# Example

Client sends

```
GET /users/1

Accept: application/json
```

Server

```json
{
    "id":1,
    "name":"Saurabh"
}
```

---

Client

```
Accept: application/xml
```

Server

```xml
<User>
   <Id>1</Id>
   <Name>Saurabh</Name>
</User>
```

---

# Request Flow

```
Client

↓

Accept Header

↓

Server

↓

Chooses Formatter

↓

Returns JSON/XML
```

---

# Common MIME Types

JSON

```
application/json
```

XML

```
application/xml
```

Plain Text

```
text/plain
```

HTML

```
text/html
```

---

# ASP.NET Core Example

```csharp
builder.Services.AddControllers()
    .AddXmlSerializerFormatters();
```

Now the API supports both

- JSON
- XML

---

# Benefits

- Flexible APIs
- Multiple clients
- Supports different formats
- Industry standard

---

# Interview Questions

## What is Content Negotiation?

It is the mechanism where the client specifies its preferred response format using the `Accept` header, and the server selects an appropriate formatter.

---

## Which header is used?

```
Accept
```

---

# Quick Revision

- Uses the `Accept` header.
- Client requests a preferred format.
- Server returns JSON, XML, or another supported representation.
- JSON is the default in ASP.NET Core.

---

# 88. OpenAPI

---

# What is OpenAPI?

**OpenAPI** is a specification that describes REST APIs in a machine-readable format.

Think of it as the **blueprint or contract** of an API.

It tells developers:

- Available endpoints
- Request format
- Response format
- Authentication
- Parameters
- Status codes

---

# Real-Life Analogy

Imagine buying furniture.

You receive an instruction manual.

The manual explains:

- Parts
- Dimensions
- Assembly steps

OpenAPI is the instruction manual for APIs.

---

# Example

```yaml
/users:
  get:
    summary: Get all users
```

This describes an API without implementing it.

---

# Why Use OpenAPI?

Instead of asking developers

```
How does this API work?
```

They simply read the OpenAPI document.

---

# OpenAPI Flow

```
API

↓

OpenAPI Specification

↓

Swagger UI

↓

Interactive Documentation
```

---

# What Information Does It Contain?

- Endpoints
- Parameters
- Request body
- Response body
- Authentication
- Models
- Error responses

---

# Benefits

- Automatic documentation
- Client SDK generation
- API testing
- Easy onboarding
- Standardized contracts

---

# Interview Questions

## What is OpenAPI?

OpenAPI is a standard specification for describing REST APIs in a language-independent, machine-readable format.

---

## Is OpenAPI the same as Swagger?

No.

- **OpenAPI** is the specification.
- **Swagger** is a collection of tools that work with the OpenAPI specification.

---

# Quick Revision

- OpenAPI describes APIs.
- Acts as an API contract.
- Language independent.
- Used for documentation, testing, and code generation.

---

# 89. Swagger Customization

---

# What is Swagger?

Swagger is a set of tools used to generate interactive API documentation from an OpenAPI specification.

It allows developers to:

- View endpoints
- Test APIs
- Understand requests and responses

without writing code.

---

# Default Swagger

```
GET /users

POST /users

DELETE /users
```

Works automatically.

---

# Why Customize Swagger?

Customization improves documentation by adding:

- API title
- Version
- Description
- Contact details
- Authentication
- XML comments
- Custom styling

---

# ASP.NET Core Configuration

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new()
    {
        Title = "User API",
        Version = "v1",
        Description = "User Management REST API"
    });
});
```

---

# Adding XML Comments

```csharp
/// <summary>
/// Gets all users.
/// </summary>
[HttpGet]
public IActionResult GetUsers()
{
    return Ok();
}
```

Swagger displays the summary automatically when XML documentation is enabled.

---

# Adding JWT Authentication

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.AddSecurityDefinition("Bearer",
        new Microsoft.OpenApi.Models.OpenApiSecurityScheme
        {
            Name = "Authorization",
            Type = Microsoft.OpenApi.Models.SecuritySchemeType.Http,
            Scheme = "bearer",
            BearerFormat = "JWT",
            In = Microsoft.OpenApi.Models.ParameterLocation.Header
        });
});
```

This enables the **Authorize** button in Swagger UI.

---

# Benefits

- Better documentation
- Interactive testing
- Improved developer experience
- Easier onboarding
- Supports authentication

---

# Interview Questions

## Why customize Swagger?

To provide richer documentation, authentication support, XML comments, API metadata, and a better developer experience.

---

## Can Swagger test APIs?

Yes.

Swagger UI allows developers to execute API requests directly from the browser.

---

# Quick Revision

- Swagger generates interactive API documentation.
- Supports API metadata, XML comments, JWT authentication, and customization.
- Built on top of the OpenAPI specification.

---

# 90. API Gateway

---

# What is an API Gateway?

An API Gateway is a **single entry point** for all client requests in a microservices architecture.

Instead of calling each service directly,

clients call the gateway.

The gateway forwards requests to the appropriate service.

---

# Real-Life Analogy

Imagine a hotel receptionist.

Guests don't contact:

- Housekeeping
- Kitchen
- Laundry
- Security

directly.

Instead,

they speak with the receptionist.

The receptionist forwards the request to the correct department.

The API Gateway plays the same role.

---

# Without API Gateway

```
Client

↓

User Service

↓

Order Service

↓

Payment Service

↓

Inventory Service
```

Client must know every service.

---

# With API Gateway

```
               Client
                  │
                  ▼
           API Gateway
        ┌──────┼──────┐
        ▼      ▼      ▼
   User Service
   Order Service
   Payment Service
   Inventory Service
```

Client only communicates with the gateway.

---

# Responsibilities

An API Gateway can perform:

- Routing
- Authentication
- Authorization
- Rate limiting
- Request aggregation
- Logging
- Caching
- Load balancing
- SSL termination
- Response transformation

---

# Routing Example

Client

```
GET /users
```

Gateway

↓

User Service

---

Client

```
POST /orders
```

Gateway

↓

Order Service

---

# Request Aggregation

Suppose a mobile app needs

- User
- Orders
- Payments

Without Gateway

```
3 API Calls
```

With Gateway

```
1 API Call

↓

Gateway calls all services

↓

Combines responses

↓

Returns one response
```

Improves performance and simplifies clients.

---

# Popular API Gateways

- Azure API Management (APIM)
- Ocelot (.NET)
- YARP (Yet Another Reverse Proxy)
- Kong
- NGINX
- Envoy
- Amazon API Gateway

---

# Advantages

- Single entry point
- Centralized authentication
- Centralized logging
- Rate limiting
- Caching
- Request aggregation
- Better security
- Simpler clients

---

# Disadvantages

- Additional infrastructure
- Can become a bottleneck if not scaled
- Adds operational complexity
- Requires monitoring and high availability

---

# API Gateway vs Reverse Proxy

| Reverse Proxy | API Gateway |
|---------------|-------------|
| Primarily forwards requests | Adds API-specific features |
| Simple routing | Routing + authentication + rate limiting + caching + transformation |
| Example: NGINX | Example: Azure API Management, Kong |

---

# Interview Questions

## What is an API Gateway?

An API Gateway is a centralized entry point that receives client requests and routes them to the appropriate backend services while providing cross-cutting features such as authentication, rate limiting, logging, and caching.

---

## Why do microservices use an API Gateway?

To simplify client communication, centralize common concerns, improve security, and reduce the need for clients to know about individual services.

---

## Name some popular API Gateways.

- Azure API Management (APIM)
- Ocelot
- YARP
- Kong
- NGINX
- Envoy
- Amazon API Gateway

---

# Quick Revision

- API Gateway is the **single entry point** for client requests.
- Performs routing, authentication, authorization, caching, logging, and rate limiting.
- Can aggregate responses from multiple services.
- Simplifies client applications.
- Widely used in microservices architectures.