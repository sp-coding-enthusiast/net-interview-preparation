# ASP.NET Core Fundamentals

# 1. What is ASP.NET Core?

## Definition

ASP.NET Core is Microsoft's modern, open-source, high-performance web development framework used to build:

- REST APIs
- Web Applications
- Microservices
- Real-time applications
- Cloud-native applications
- Enterprise applications

It is a complete rewrite of the old ASP.NET Framework and is designed to be:

- Fast
- Lightweight
- Cross-platform
- Modular
- Cloud-ready

Unlike the old ASP.NET Framework, ASP.NET Core runs on:

- Windows
- Linux
- macOS

using the same codebase.

---

## Simple Layman Explanation

Imagine you want to build a shopping mall.

You need:

- Foundation
- Roads
- Security
- Electricity
- Parking
- Water Supply

Instead of building everything yourself, you hire a construction company that already knows how to build malls.

ASP.NET Core is that construction company.

It already provides:

- Routing
- Authentication
- Authorization
- Dependency Injection
- Configuration
- Logging
- Middleware
- Security
- API support

You only focus on writing your business logic.

---

## Real-Life Example

Suppose Amazon wants to build an Order Service.

Instead of writing:

- HTTP handling
- URL parsing
- JSON serialization
- Authentication
- Logging
- Error handling

from scratch...

ASP.NET Core already provides all these.

Developers only write:

```
PlaceOrder()
CancelOrder()
GetOrder()
```

Everything else is handled by ASP.NET Core.

---

# What Can You Build?

Using ASP.NET Core you can build:

- Banking APIs
- E-commerce websites
- Inventory Management Systems
- Hospital Management Systems
- ERP Systems
- CRM Applications
- Mobile Backend APIs
- AI Applications
- Microservices
- SaaS Products

---

# Simple API Example

```
GET /products

Response

[
  {
    "id":1,
    "name":"Laptop"
  }
]
```

ASP.NET Core automatically handles:

- Incoming request
- Routing
- JSON conversion
- Response generation

---

# Request Flow

```
Browser

   │

   ▼

ASP.NET Core Server

   │

Routing

   │

Controller

   │

Business Logic

   │

Database

   │

JSON Response

   │

Browser
```

---

# Features of ASP.NET Core

- Cross-platform
- Open Source
- High Performance
- Built-in Dependency Injection
- Middleware Pipeline
- REST API support
- Razor Pages
- MVC
- Minimal APIs
- SignalR
- Authentication
- Authorization
- Configuration
- Logging
- Cloud Ready
- Docker Support
- Kubernetes Friendly

---

# Real Enterprise Example

Suppose you're building an Airline Booking System.

Modules:

```
Flight Service

Booking Service

Payment Service

Notification Service

Authentication Service
```

Each module can be built using ASP.NET Core.

---

# Why Companies Love ASP.NET Core

Because it is:

- Fast
- Secure
- Lightweight
- Easy to deploy
- Works in Docker
- Works in Kubernetes
- Excellent cloud integration
- Long-term Microsoft support

---

# 2. How is ASP.NET Core different from ASP.NET MVC?

Many beginners think they are the same.

They are not.

---

## ASP.NET MVC (Old Framework)

Released around 2009.

Runs only on:

- Windows

Requires:

- .NET Framework

Uses:

```
System.Web
```

Heavy framework.

Not cloud-first.

No built-in Dependency Injection.

Limited performance.

---

## ASP.NET Core

Released in 2016.

Completely rewritten.

Runs on:

- Windows
- Linux
- macOS

Uses:

```
.NET
```

No dependency on System.Web.

Designed for:

- Cloud
- Containers
- APIs
- Microservices
- High Performance

---

# Comparison Table

| Feature | ASP.NET MVC | ASP.NET Core |
|----------|-------------|--------------|
| Platform | Windows only | Windows, Linux, macOS |
| Runtime | .NET Framework | .NET |
| Performance | Moderate | Extremely Fast |
| Open Source | Partially | Fully Open Source |
| Dependency Injection | External libraries | Built-in |
| Middleware | HTTP Modules | Middleware Pipeline |
| Deployment | IIS | IIS, Kestrel, Nginx, Docker |
| Cloud Support | Limited | Excellent |
| Docker Support | Difficult | Native |
| Microservices | Not ideal | Excellent |
| Configuration | web.config | appsettings.json |
| Logging | External | Built-in |
| Side-by-side Versioning | No | Yes |

---

# Simple Analogy

Imagine two cars.

Old ASP.NET MVC:

```
Toyota Corolla (2005)
```

Reliable but older.

ASP.NET Core:

```
Tesla Model S
```

Modern.

Faster.

Smarter.

Cloud connected.

Better performance.

---

# Startup Comparison

Old MVC

```
Global.asax

RouteConfig.cs

BundleConfig.cs

FilterConfig.cs

Web.config
```

Many files.

ASP.NET Core

```
Program.cs
```

Almost everything starts here.

Much simpler.

---

# Configuration Comparison

Old MVC

```
Web.config
```

ASP.NET Core

```
appsettings.json
```

Cleaner.

Environment-specific.

Cloud friendly.

---

# Dependency Injection

Old MVC

Need external libraries.

Example:

```
Autofac

Unity

Ninject
```

ASP.NET Core

Already built-in.

```
builder.Services.AddScoped<IProductService, ProductService>();
```

Done.

---

# Hosting Comparison

Old MVC

Mostly:

```
IIS
```

ASP.NET Core

Can run:

- IIS
- Kestrel
- Docker
- Linux
- Azure
- AWS
- Kubernetes

---

# Performance

ASP.NET Core is among the fastest web frameworks in the world because:

- Lightweight pipeline
- Optimized runtime
- Better memory management
- Minimal APIs
- Efficient JSON serialization

---

# Enterprise Preference

Today almost every new Microsoft-based enterprise application uses ASP.NET Core instead of old ASP.NET MVC.

---

# 3. What are the advantages of ASP.NET Core?

There are many advantages.

Let's understand each one.

---

# 1. Cross Platform

Write once.

Run anywhere.

```
Windows

Linux

macOS
```

Example:

Develop on Windows.

Deploy to Linux Server.

No code changes.

---

# 2. High Performance

ASP.NET Core is optimized for speed.

Example:

An API serving:

```
100 requests/second
```

on old MVC

might serve

```
300+ requests/second
```

on ASP.NET Core under similar conditions (actual performance depends on hardware, application design, and workload).

---

# 3. Built-in Dependency Injection

No third-party library needed.

```
builder.Services.AddScoped<IOrderService, OrderService>();
```

Cleaner code.

Better testing.

---

# 4. Middleware Pipeline

Only execute required components.

Request

```
Authentication

↓

Logging

↓

Authorization

↓

Controller
```

This makes applications faster and easier to customize.

---

# 5. Cloud Ready

Perfect for:

- Azure
- AWS
- Google Cloud

Supports:

- Containers
- Load balancing
- Auto scaling

---

# 6. Docker Support

Containerization is simple.

```
Docker Build

↓

Docker Image

↓

Docker Container
```

Deploy anywhere.

---

# 7. Open Source

Anyone can view the source code.

Community contributions improve the framework.

---

# 8. Built-in Logging

No need to configure logging from scratch.

Example:

```
logger.LogInformation("Order Created");
```

---

# 9. Configuration Management

Settings stored in:

```
appsettings.json
```

Different environments:

```
Development

Testing

Production
```

can each have their own configuration files.

---

# 10. Security

Built-in:

- Authentication
- Authorization
- HTTPS
- Identity
- JWT Support
- OAuth
- OpenID Connect

---

# 11. Excellent API Support

Creating APIs is straightforward.

```
GET /products

POST /products

PUT /products

DELETE /products
```

---

# 12. Supports Modern Development

Works well with:

- Docker
- Kubernetes
- Microservices
- CI/CD
- GitHub Actions
- Azure DevOps

---

# 13. Easy Testing

Supports:

- Unit Testing
- Integration Testing
- Mocking

Because Dependency Injection is built-in.

---

# 14. Minimal APIs

Very small APIs can be written with minimal code.

Example:

```csharp
app.MapGet("/hello", () => "Hello World");
```

---

# Enterprise Benefits

Large companies choose ASP.NET Core because it provides:

- Better scalability
- Lower hosting costs
- Faster applications
- Easier maintenance
- Easier deployment
- Better cloud integration

---

# 4. What is the architecture of ASP.NET Core?

ASP.NET Core follows a layered architecture.

The request travels through multiple components before reaching your business logic.

---

# High-Level Architecture

```
                 Client
          (Browser / Mobile App)

                     │

                     ▼

              Kestrel Web Server

                     │

                     ▼

           Middleware Pipeline

                     │

        Authentication Middleware

                     │

           Authorization Middleware

                     │

             Routing Middleware

                     │

                     ▼

          MVC / Minimal API Endpoint

                     │

                     ▼

              Controller / Endpoint

                     │

                     ▼

             Business Service Layer

                     │

                     ▼

             Repository / Data Access

                     │

                     ▼

                Database
```

---

# Step 1 — Client Sends Request

Example:

```
GET /products
```

The request comes from:

- Browser
- Mobile App
- React
- Angular
- Blazor

---

# Step 2 — Kestrel Server

Kestrel is ASP.NET Core's built-in cross-platform web server.

Responsibilities:

- Listen for HTTP requests
- Accept client connections
- Forward requests into the ASP.NET Core pipeline

---

# Step 3 — Middleware Pipeline

Every request passes through middleware.

Example:

```
Request

↓

Logging

↓

Authentication

↓

Authorization

↓

Routing

↓

Controller

↓

Response
```

Each middleware can:

- Continue processing
- Modify the request/response
- Stop the request early (short-circuit)

---

# Step 4 — Routing

Routing decides:

```
Which controller?

Which action?

Which endpoint?
```

Example:

```
GET /products
```

becomes

```
ProductController

↓

GetProducts()
```

---

# Step 5 — Controller

Controller receives the request.

Example:

```csharp
public IActionResult GetProducts()
{
    return Ok();
}
```

The controller should contain minimal business logic and typically delegates work to services.

---

# Step 6 — Service Layer

Business rules live here.

Example:

```
Calculate Discount

Validate Order

Generate Invoice

Check Inventory
```

Controllers call services rather than implementing these rules directly.

---

# Step 7 — Repository Layer

Handles database operations.

Example:

```
Get Products

Insert Product

Delete Product
```

This separates business logic from data access.

---

# Step 8 — Database

The repository communicates with databases such as:

- SQL Server
- PostgreSQL
- MySQL
- Oracle
- Cosmos DB

---

# Step 9 — Response

The result flows back through the middleware pipeline to the client.

```
Database

↓

Repository

↓

Service

↓

Controller

↓

Middleware

↓

Browser
```

---

# Complete End-to-End Example

A user opens an e-commerce website.

```
GET /orders/1001
```

### Flow

```
Browser

↓

Kestrel

↓

Logging Middleware

↓

Authentication Middleware

↓

Authorization Middleware

↓

Routing

↓

OrderController

↓

OrderService

↓

OrderRepository

↓

SQL Server

↓

Repository

↓

Service

↓

Controller

↓

JSON Response

↓

Browser
```

---

# Why This Architecture Matters

Separating responsibilities into layers provides several benefits:

- Easier maintenance
- Better testability
- Improved scalability
- Reusable business logic
- Clear separation of concerns
- Easier debugging
- Cleaner codebase
- Support for large enterprise applications

---

# Key Takeaways

- **ASP.NET Core** is Microsoft's modern, open-source, cross-platform framework for building high-performance web applications and APIs.
- It is a complete redesign of the older ASP.NET Framework and offers better performance, modularity, cloud support, and built-in Dependency Injection.
- Its major advantages include cross-platform support, high performance, middleware architecture, cloud readiness, container support, security, and excellent API capabilities.
- The ASP.NET Core architecture follows a layered request-processing model: **Client → Kestrel → Middleware → Routing → Controller/Endpoint → Service → Repository → Database → Response**, making applications easier to maintain, test, and scale.