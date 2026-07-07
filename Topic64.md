# 63. Kestrel Configuration in ASP.NET Core

**Kestrel** is the **cross-platform web server** built into ASP.NET Core.

Whenever an ASP.NET Core application starts, **Kestrel listens for incoming HTTP/HTTPS requests**, processes them through the middleware pipeline, and sends the response back to the client.

Configuring Kestrel means telling it:

- Which ports to listen on
- Whether to use HTTP or HTTPS
- Which SSL certificate to use
- Connection limits
- Request limits
- Performance settings

> **Interview Tip:** Kestrel is the **default web server** for ASP.NET Core. In production, it is commonly placed **behind a reverse proxy** such as Nginx, Apache, or IIS, although it can also be exposed directly depending on the hosting environment.

---

# What is Kestrel?

## Definition

Kestrel is a **lightweight, high-performance web server** developed by Microsoft for ASP.NET Core.

It is responsible for:

- Accepting HTTP requests
- Accepting HTTPS requests
- Sending responses
- Managing client connections

---

# Layman Example: Restaurant Receptionist

Imagine a restaurant.

Customers arrive at the entrance.

The receptionist:

- Welcomes customers
- Assigns a table
- Sends them to the waiter

The receptionist does **not** cook the food.

Similarly,

Kestrel receives HTTP requests and forwards them to the ASP.NET Core application.

```
Client

↓

Kestrel

↓

Middleware

↓

Controller

↓

Response
```

---

# Another Example: Airport Security

Passengers arrive.

Airport security checks them.

Then they proceed to the gate.

Kestrel acts like the first checkpoint for incoming requests.

---

# Why Do We Need Kestrel?

Without Kestrel,

your ASP.NET Core application cannot listen for incoming web requests.

```
Browser

↓

???

↓

Application
```

No server means no communication.

With Kestrel,

```
Browser

↓

Kestrel

↓

Application
```

---

# Default Kestrel Flow

```
Browser

↓

Kestrel

↓

Middleware

↓

Controller

↓

Response
```

---

# Default Configuration

A newly created ASP.NET Core application automatically configures Kestrel.

You typically don't need to write any code.

Running

```bash
dotnet run
```

starts Kestrel.

Example output

```
Now listening on:

https://localhost:7001

http://localhost:5000
```

---

# Configure Ports

Kestrel can listen on different ports.

Example

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:8080"
      },
      "Https": {
        "Url": "https://localhost:8443"
      }
    }
  }
}
```

Now the application listens on:

```
8080
```

and

```
8443
```

---

# Configure HTTPS

HTTPS requires an SSL/TLS certificate.

Example

```json
{
  "Kestrel": {
    "Endpoints": {
      "Https": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "certificate.pfx",
          "Password": "your-password"
        }
      }
    }
  }
}
```

In production, certificates are often managed by the hosting platform or reverse proxy instead of storing passwords in configuration.

---

# Configure in Code

You can also configure Kestrel programmatically.

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(5000);

    options.ListenAnyIP(5001, listen =>
    {
        listen.UseHttps();
    });
});
```

---

# Multiple Endpoints

Kestrel can listen on multiple endpoints simultaneously.

```
HTTP

↓

Port 5000
```

AND

```
HTTPS

↓

Port 5001
```

At the same time.

---

# Connection Limits

You can limit concurrent connections.

Example

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.Limits.MaxConcurrentConnections = 100;
});
```

Useful for protecting the server from excessive load.

---

# Request Body Size

Example

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.Limits.MaxRequestBodySize =
        50 * 1024 * 1024;
});
```

Allows uploads up to **50 MB**.

Useful for file upload APIs.

---

# Request Timeout

Example

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.Limits.KeepAliveTimeout =
        TimeSpan.FromMinutes(2);
});
```

Controls how long idle keep-alive connections remain open.

---

# HTTP/2 and HTTP/3

Kestrel supports modern protocols including:

- HTTP/1.1
- HTTP/2
- HTTP/3 (when supported by the operating system and hosting environment)

These improve performance and efficiency for modern applications.

---

# Reverse Proxy Architecture

Production deployment

```
Internet

↓

Nginx / IIS / Apache

↓

Kestrel

↓

ASP.NET Core
```

The reverse proxy can provide:

- SSL termination
- Load balancing
- Compression
- Security filtering
- Static file serving

Kestrel focuses on running the application.

---

# Real-World Example

E-commerce Website

```
Customer

↓

Nginx

↓

Kestrel

↓

ASP.NET Core API

↓

Database
```

The customer never communicates directly with the application code.

Kestrel manages incoming requests.

---

# Kestrel vs IIS

| Kestrel | IIS |
|----------|-----|
| Cross-platform web server | Windows web server |
| Built into ASP.NET Core | Windows feature |
| Runs on Linux, Windows, macOS | Windows only |
| Hosts the application | Can act as a reverse proxy or host ASP.NET Core on Windows |

---

# Kestrel vs Reverse Proxy

| Kestrel | Reverse Proxy |
|----------|---------------|
| Executes the ASP.NET Core application | Receives external traffic first |
| Processes HTTP requests | Can perform SSL termination, load balancing, and request filtering |
| Required for ASP.NET Core hosting | Often used in front of Kestrel in production |

---

# Internal Flow

```
Browser

↓

Kestrel

↓

Middleware

↓

Routing

↓

Controller

↓

Response
```

---

# Advantages

- High performance.
- Cross-platform.
- Built into ASP.NET Core.
- Supports HTTP/1.1, HTTP/2, and HTTP/3.
- Easy configuration.
- Excellent integration with ASP.NET Core.

---

# Disadvantages

- Large production deployments commonly benefit from a reverse proxy in front of Kestrel.
- Advanced web server features such as load balancing are typically handled by external infrastructure.

---

# Common Mistakes

## Exposing Development Certificates in Production

Avoid using development certificates outside development environments.

Use trusted production certificates.

---

## Forgetting HTTPS

Serving sensitive data over HTTP is insecure.

Use HTTPS for production traffic.

---

## Setting Unlimited Upload Sizes

Unlimited request sizes can increase the risk of denial-of-service attacks.

Always define appropriate limits.

---

## Ignoring Connection Limits

Large public APIs should configure sensible limits to protect resources.

---

# Best Practices

- Use HTTPS in production.
- Place Kestrel behind a reverse proxy when appropriate.
- Configure upload size limits.
- Configure connection and timeout limits.
- Use production certificates.
- Monitor server performance and tune limits based on workload.

---

# Visual Representation

```
Internet

↓

Reverse Proxy

↓

Kestrel

↓

Middleware

↓

Controllers

↓

Database
```

---

# Easy Way to Remember

Imagine **a hotel receptionist**.

Guests don't walk directly into hotel rooms.

First,

they meet the receptionist.

The receptionist:

- Welcomes them
- Verifies them
- Directs them

Kestrel is the receptionist of your ASP.NET Core application.

---

# Frequently Asked Interview Questions

### What is Kestrel?

Kestrel is the default cross-platform web server used by ASP.NET Core to receive HTTP/HTTPS requests and send responses.

---

### Can Kestrel host HTTPS?

Yes.

Kestrel supports HTTPS using SSL/TLS certificates.

---

### Can Kestrel listen on multiple ports?

Yes.

It can listen on multiple HTTP and HTTPS endpoints simultaneously.

---

### Why is Kestrel often placed behind IIS or Nginx?

A reverse proxy can provide features such as SSL termination, load balancing, request filtering, and static file serving while forwarding requests to Kestrel.

---

### Can Kestrel be configured in code?

Yes.

Kestrel can be configured using `ConfigureKestrel()` in `Program.cs` or through configuration files such as `appsettings.json`.

---

# Interview Answer (2-Minute Version)

**What is Kestrel and how is it configured in ASP.NET Core?**

Kestrel is the default high-performance, cross-platform web server used by ASP.NET Core. It listens for incoming HTTP and HTTPS requests, forwards them through the middleware pipeline, and returns responses to clients. Kestrel can be configured through `appsettings.json` or programmatically using `ConfigureKestrel()` to define listening ports, HTTPS certificates, request size limits, connection limits, and timeout settings. While Kestrel is fully capable of hosting ASP.NET Core applications, production deployments commonly place it behind a reverse proxy such as IIS, Nginx, or Apache to provide additional capabilities like SSL termination, load balancing, and request filtering.