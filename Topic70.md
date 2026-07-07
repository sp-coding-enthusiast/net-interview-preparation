# ASP.NET Core Revision Sheet (Topics 1–70)

> **Purpose:** Quick interview revision for ASP.NET Core concepts.
>
> For every topic:
> - **Definition (1 line)**
> - **One real-world analogy**
> - **One code/example**

---

# Middleware

| Topic | One-Line Revision | Example |
|-------|-------------------|----------|
| Middleware | Software component that processes HTTP requests and responses. | Logging Middleware |
| Middleware Pipeline | Ordered chain of middleware executed for every request. | Request → Authentication → Authorization → Endpoint |
| Middleware Execution | Request flows top-down, response flows bottom-up. | Browser → Middleware → Controller → Middleware → Browser |
| Use vs Run vs Map | Use = Continue pipeline, Run = End pipeline, Map = Branch pipeline | `app.Use()`, `app.Run()`, `app.Map("/admin")` |
| Custom Middleware | Middleware written by developers. | Request Logging Middleware |
| Exception Middleware | Catches unhandled exceptions globally. | `app.UseExceptionHandler()` |
| Authentication Middleware | Identifies who the user is. | JWT Authentication |
| Authorization Middleware | Checks whether user has permission. | `[Authorize]` |
| Static Files Middleware | Serves CSS, JS, Images. | `app.UseStaticFiles()` |
| Response Compression | Compresses HTTP responses. | Gzip/Brotli |
| Response Caching | Returns cached responses. | `app.UseResponseCaching()` |
| CORS Middleware | Allows/restricts cross-origin requests. | React → ASP.NET Core API |
| HSTS | Forces browser to use HTTPS. | `app.UseHsts()` |
| HTTPS Redirection | Redirects HTTP to HTTPS. | `app.UseHttpsRedirection()` |
| Endpoint Middleware | Executes mapped endpoints. | `app.MapControllers()` |
| Middleware Ordering | Middleware order determines behavior. | Authentication before Authorization |
| Branching Middleware | Different pipeline for different paths. | `/api`, `/admin` |
| Short Circuiting | Ends request pipeline early. | Static file served immediately |
| Performance | Keep middleware lightweight and ordered correctly. | Compression after Static Files |
| Debug Middleware | Use logging, breakpoints, diagnostics. | `ILogger` |

---

# Dependency Injection (DI)

| Topic | One-Line Revision | Example |
|-------|-------------------|----------|
| Built-in DI | ASP.NET Core has built-in Dependency Injection container. | `builder.Services.AddScoped()` |
| Service Lifetimes | Singleton, Scoped, Transient | Repository Registration |
| Singleton Pitfalls | Don't inject Scoped into Singleton. | InvalidOperationException |
| Scoped Service | One instance per HTTP request. | DbContext |
| Transient Service | New instance every injection. | EmailService |
| Constructor Injection | Inject dependencies through constructor. | `ILogger<ProductService>` |
| Method Injection | Inject service into action method. | `[FromServices]` |
| Factory Pattern | Creates complex objects. | `Func<T>` |
| Options Pattern | Strongly typed configuration. | `IOptions<AppSettings>` |
| IOptions vs Snapshot vs Monitor | Static vs Per Request vs Dynamic | Configuration Reload |
| Circular Dependency | Service A → B → A | Avoid redesign needed |
| Multiple Implementations | Register multiple services for same interface. | Notification Services |
| Open Generic Registration | Register generic services. | `IRepository<>` |
| Decorator Pattern | Add behavior without changing original class. | Logging Decorator |
| Replace Built-in DI | Use Autofac, Lamar, etc. | Autofac |

---

# Configuration

| Topic | One-Line Revision | Example |
|-------|-------------------|----------|
| Configuration Providers | Multiple configuration sources. | JSON + Env Variables |
| appsettings.json Hierarchy | Environment-specific configuration. | `appsettings.Production.json` |
| Environment Variables | OS-level configuration. | `Database__Server` |
| Secret Manager | Development secret storage. | API Keys |
| Azure Key Vault | Production secret storage. | Database Password |
| Configuration Precedence | Last provider wins. | Env Variable overrides JSON |
| Logging Configuration | Configure logging levels/providers. | Information, Warning |
| Kestrel Configuration | Configure web server. | Port 5001 |
| launchSettings.json | Development startup configuration. | Launch Browser |

---

# Hosting

| Topic | One-Line Revision | Example |
|-------|-------------------|----------|
| Generic Host | Manages DI, Logging, Configuration. | `WebApplication.CreateBuilder()` |
| Worker Service | Long-running background application. | Queue Processor |
| BackgroundService | Base class for background jobs. | `ExecuteAsync()` |
| Hosted Services | Background services managed by Generic Host. | Email Worker |
| Graceful Shutdown | Finish work before stopping. | `CancellationToken` |
| Health Checks | Check application health. | `/health` |

---

# Most Important Middleware Order

```
Request

↓

Exception Middleware

↓

HTTPS Redirection

↓

Static Files

↓

Routing

↓

CORS

↓

Authentication

↓

Authorization

↓

Response Caching

↓

Endpoint

↓

Response
```

---

# DI Lifetime Summary

```
Singleton

↓

One Instance

↓

Entire Application
```

```
Scoped

↓

One Instance

↓

Per HTTP Request
```

```
Transient

↓

New Instance

↓

Every Injection
```

---

# Configuration Precedence

```
Lowest Priority

↓

appsettings.json

↓

appsettings.{Environment}.json

↓

User Secrets

↓

Environment Variables

↓

Command-Line Arguments

↓

Highest Priority
```

**Rule:** **Last provider loaded wins.**

---

# IOptions Summary

| Interface | Lifetime | Use Case |
|-----------|----------|----------|
| IOptions<T> | Singleton | Static configuration |
| IOptionsSnapshot<T> | Scoped | Reload per request |
| IOptionsMonitor<T> | Singleton | Dynamic configuration changes |

---

# Generic Host Architecture

```
Application

↓

Generic Host

├── Configuration

├── Dependency Injection

├── Logging

├── Hosted Services

└── Kestrel

↓

Running
```

---

# Worker Service Flow

```
Application

↓

Generic Host

↓

BackgroundService

↓

ExecuteAsync()

↓

Loop

↓

Graceful Shutdown
```

---

# Health Check Flow

```
Monitoring Tool

↓

/health

↓

Application

↓

Database

↓

Redis

↓

Queue

↓

Healthy
```

---

# Hosting Components

```
Browser

↓

Kestrel

↓

Middleware

↓

Controller

↓

Database

↓

Response
```

---

# Common Interview Questions

### What is Middleware?

A component that handles HTTP requests and responses.

---

### What is Dependency Injection?

A design pattern where dependencies are supplied instead of created manually.

---

### What is the difference between Singleton, Scoped, and Transient?

- Singleton → One instance for the application.
- Scoped → One instance per request.
- Transient → New instance every injection.

---

### What is Kestrel?

The default cross-platform web server for ASP.NET Core.

---

### What is Generic Host?

The infrastructure that manages DI, Logging, Configuration, Hosted Services, and application lifetime.

---

### What is a Worker Service?

A long-running background application using the Generic Host.

---

### What is BackgroundService?

A base class used to implement background tasks.

---

### What are Hosted Services?

Background services started and stopped automatically by the Generic Host.

---

### What is Graceful Shutdown?

Stopping an application safely after completing ongoing work.

---

### What are Health Checks?

Endpoints that report the application's health to monitoring systems.

---

### What is Azure Key Vault?

A secure cloud service for storing secrets, keys, and certificates in production.

---

### What is Secret Manager?

A development-only feature for storing secrets outside the project.

---

### What is Configuration Precedence?

The order in which configuration providers are loaded; the **last provider wins**.

---

### What is launchSettings.json?

A development-only file that defines how the application starts locally.

---

# 30-Second Final Revision

- **Middleware** → Processes HTTP requests.
- **Pipeline** → Ordered middleware execution.
- **DI** → Manages object creation.
- **Singleton** → One instance for app.
- **Scoped** → One per request.
- **Transient** → New every injection.
- **Options Pattern** → Strongly typed configuration.
- **Environment Variables** → Production configuration.
- **Secret Manager** → Development secrets.
- **Azure Key Vault** → Production secrets.
- **Kestrel** → Web server.
- **Generic Host** → Application infrastructure.
- **Worker Service** → Background application.
- **BackgroundService** → Base class for background work.
- **Hosted Service** → Managed background service.
- **Health Checks** → Application monitoring endpoint.
- **Graceful Shutdown** → Finish work before stopping.
- **Configuration Precedence** → Last configuration provider wins.