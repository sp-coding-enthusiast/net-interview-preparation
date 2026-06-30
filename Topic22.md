# ASP.NET Core Fundamentals

# 5. What is Kestrel?

## Definition

Kestrel is the **cross-platform, high-performance web server** built into ASP.NET Core.

Whenever an ASP.NET Core application starts, Kestrel is the server that actually listens for incoming HTTP requests from clients and sends back responses.

Think of Kestrel as the **front door** of your application.

Without a web server like Kestrel, your ASP.NET Core application cannot receive requests from browsers, mobile apps, or other services.

---

# Simple Layman Explanation

Imagine you own a restaurant.

Customers arrive at the entrance.

Someone must:

- Open the door
- Welcome customers
- Take them to their table
- Inform the waiter

That person is like **Kestrel**.

Your chef (business logic) never interacts directly with customers.

Instead:

```
Customer
      ↓
Receptionist (Kestrel)
      ↓
Waiter (Middleware)
      ↓
Chef (Controller + Service)
      ↓
Food Returned
      ↓
Customer
```

Kestrel acts as the receptionist.

---

# Real-Life Example

Suppose a user visits:

```
https://amazon.com/products
```

The browser sends a request.

Kestrel:

- Accepts the TCP connection
- Reads the HTTP request
- Passes the request into the ASP.NET Core middleware pipeline

After processing,

Kestrel sends the HTTP response back to the browser.

---

# Where Does Kestrel Fit?

```
Browser

      │

      ▼

  Kestrel Server

      │

      ▼

Middleware Pipeline

      │

      ▼

Controller

      │

      ▼

Business Logic

      │

      ▼

Database
```

---

# Features of Kestrel

- Extremely fast
- Lightweight
- Cross-platform
- Built into ASP.NET Core
- Supports HTTP/1.1
- Supports HTTP/2
- Supports HTTP/3 (when supported by the platform and configuration)
- HTTPS support
- WebSocket support
- Async request processing
- High concurrency
- Optimized for cloud environments

---

# Default Hosting

When you execute:

```bash
dotnet run
```

Your application starts Kestrel automatically.

Example output:

```
Now listening on:

https://localhost:5001

http://localhost:5000
```

Those ports are served by Kestrel.

---

# Can Kestrel Serve Production Traffic?

Yes.

Kestrel is production-ready.

However, in many production environments it sits behind a reverse proxy such as:

- IIS
- Nginx
- Apache
- Azure App Service front-end

This provides additional features like request filtering, load balancing, and centralized TLS management.

---

# Simple Diagram

```
Internet

     │

     ▼

Kestrel

     │

Middleware

     │

Controllers

     │

Database
```

---

# 6. How does Kestrel work internally?

Understanding Kestrel internally is a favorite interview topic for Senior and Lead .NET developers.

---

# High-Level Flow

```
Client

      │

TCP Connection

      │

Kestrel

      │

HTTP Parser

      │

Middleware Pipeline

      │

Routing

      │

Controller

      │

Service

      │

Repository

      │

Database

      │

Response

      │

Kestrel

      │

Client
```

---

# Step 1 — Client Sends Request

Example:

```
GET /products HTTP/1.1
```

The browser sends the request over TCP.

---

# Step 2 — Kestrel Listens on a Port

When your application starts,

Kestrel opens ports such as:

```
5000

5001

8080
```

It continuously waits for incoming connections.

```
Listening...

Listening...

Listening...
```

---

# Step 3 — Accept Connection

A browser connects.

```
Chrome

↓

TCP Connection

↓

Kestrel
```

Kestrel accepts the connection.

---

# Step 4 — Parse HTTP Request

The browser sends:

```
GET /products HTTP/1.1

Host: localhost

Accept: application/json
```

Kestrel parses:

- HTTP method
- URL
- Headers
- Cookies
- Query string
- Request body

It converts this into an internal HTTP context that ASP.NET Core can work with.

---

# Step 5 — Create HttpContext

Kestrel creates an `HttpContext` object.

It contains:

- Request
- Response
- User information
- Cookies
- Headers
- Services
- Session (if enabled)

Think of `HttpContext` as the container holding everything about the current request.

---

# Step 6 — Send Request to Middleware

```
Kestrel

↓

Middleware 1

↓

Middleware 2

↓

Middleware 3

↓

Controller
```

Each middleware can:

- Continue the request
- Modify the request
- End the request immediately

---

# Step 7 — Controller Executes

Example:

```csharp
public IActionResult GetProducts()
{
    return Ok(products);
}
```

The controller performs the requested action.

---

# Step 8 — Response Generated

Example:

```json
[
  {
    "id":1,
    "name":"Laptop"
  }
]
```

The controller creates the response.

---

# Step 9 — Middleware Executes Again

The response travels back through the middleware pipeline.

```
Controller

↓

Middleware

↓

Middleware

↓

Kestrel
```

This allows middleware to:

- Log responses
- Compress content
- Add headers
- Handle caching

---

# Step 10 — Kestrel Sends Response

Finally,

Kestrel converts the response into raw HTTP data and sends it back over the network.

```
HTTP/1.1 200 OK

Content-Type: application/json

Body...
```

The browser receives it.

---

# Complete Internal Flow

```
Browser

      │

TCP

      │

Kestrel

      │

HTTP Parser

      │

HttpContext

      │

Middleware

      │

Routing

      │

Controller

      │

Business Layer

      │

Repository

      │

Database

      │

Repository

      │

Controller

      │

Middleware

      │

Kestrel

      │

Browser
```

---

# Why Is Kestrel So Fast?

Kestrel is designed for modern high-performance workloads.

Reasons include:

- Asynchronous I/O
- Efficient memory usage
- Optimized networking
- Minimal allocations
- High concurrency
- Tight integration with ASP.NET Core

This allows it to handle thousands of simultaneous requests efficiently, depending on the application's design and available hardware.

---

# 7. Kestrel vs IIS

Many beginners think Kestrel and IIS are competitors.

They are not.

They often work together.

---

# What is IIS?

IIS (Internet Information Services) is Microsoft's full-featured web server for Windows.

It provides:

- Website hosting
- SSL/TLS management
- Application pools
- Request filtering
- Windows Authentication
- Logging
- Reverse proxy capabilities
- Process management

---

# Relationship

In many Windows deployments:

```
Internet

     │

     ▼

IIS

     │

Reverse Proxy

     │

Kestrel

     │

ASP.NET Core App
```

IIS receives the external request first and forwards it to Kestrel.

---

# Analogy

Imagine a large corporate office.

```
Visitor

↓

Security Gate (IIS)

↓

Reception

↓

Employee (Kestrel)

↓

Department

↓

Manager
```

Security checks visitors.

The receptionist forwards them to the right department.

Both work together.

---

# Feature Comparison

| Feature | Kestrel | IIS |
|----------|----------|-----|
| Type | Lightweight web server | Full-featured Windows web server |
| Platform | Windows, Linux, macOS | Windows only |
| Built into ASP.NET Core | Yes | No |
| Performance | Extremely fast | Very good |
| Cross-platform | Yes | No |
| Reverse Proxy | No | Yes |
| Windows Authentication | Limited directly; commonly handled through IIS integration | Excellent |
| Static File Hosting | Yes | Yes |
| Process Management | Basic | Advanced |
| SSL/TLS Termination | Yes | Yes |
| Load Balancing | No | Can integrate with Windows infrastructure |
| Ideal for Containers | Excellent | Not commonly used inside containers |

---

# When Should You Use Only Kestrel?

Examples:

- Docker containers
- Kubernetes
- Linux servers
- Azure Container Apps
- Microservices
- Internal APIs

Architecture:

```
Internet

↓

Kestrel

↓

Application
```

---

# When Should You Use IIS + Kestrel?

Typical Windows enterprise deployments.

Architecture:

```
Internet

↓

IIS

↓

Kestrel

↓

Application
```

Benefits:

- Windows Authentication
- Centralized SSL certificate management
- Request filtering
- Process monitoring and recycling
- Additional security features

---

# Interview Question

### Why do we use IIS if Kestrel is already a web server?

**Answer:**

Kestrel is a fast, lightweight web server that processes ASP.NET Core requests. IIS complements it by acting as a reverse proxy, providing enterprise features such as request filtering, Windows Authentication, SSL/TLS termination, logging, process management, and centralized administration. Together, they offer both high performance and robust hosting capabilities.

---

# Key Takeaways

- **Kestrel** is the default, built-in web server for ASP.NET Core and is responsible for accepting HTTP requests and returning responses.
- Internally, Kestrel listens on network ports, accepts TCP connections, parses HTTP requests, creates an `HttpContext`, sends the request through the middleware pipeline, and returns the generated response to the client.
- **IIS** is a full-featured Windows web server that is commonly used as a reverse proxy in front of Kestrel for Windows-hosted applications.
- **Kestrel and IIS are complementary, not competing technologies.** Kestrel executes the ASP.NET Core application, while IIS provides enterprise-grade hosting, security, and management features.