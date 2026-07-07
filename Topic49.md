# 49. Options Pattern in ASP.NET Core

The **Options Pattern** is the recommended way to read and manage **application configuration** in ASP.NET Core.

Instead of scattering configuration values throughout your code, the Options Pattern binds configuration to **strongly typed C# classes**.

This makes your application:

- Cleaner
- Type-safe
- Easier to maintain
- Easier to test
- Easier to validate

> **Interview Tip:** Almost every production ASP.NET Core application uses the Options Pattern to read settings from `appsettings.json`, environment variables, Azure App Configuration, or other configuration providers.

---

# What is the Options Pattern?

## Definition

The Options Pattern maps configuration values from configuration sources into a **strongly typed class**.

Instead of writing:

```csharp
var apiKey = configuration["EmailSettings:ApiKey"];
```

You write:

```csharp
options.ApiKey
```

This is easier to read and less error-prone.

---

# Layman Example: Contact List

Imagine your phone.

Without contact names

```
9876543210

9123456789

9988776655
```

You must remember every number.

---

With contact names

```
Mom

Dad

Office
```

Much easier.

The Options Pattern gives meaningful names to configuration values.

---

# Another Example: Recipe Card

Without Options Pattern

```
Sugar = 2

Milk = 500

Tea = 10
```

Random values.

---

With Options Pattern

```
TeaRecipe

↓

Sugar

Milk

Tea Powder
```

Everything is organized into one object.

---

# Typical Configuration File

`appsettings.json`

```json
{
  "EmailSettings": {
    "Sender": "admin@company.com",
    "ApiKey": "ABC123",
    "Host": "smtp.company.com",
    "Port": 587
  }
}
```

---

# Without Options Pattern

```csharp
public class EmailService
{
    private readonly IConfiguration _configuration;

    public EmailService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public void Send()
    {
        var host = _configuration["EmailSettings:Host"];
        var port = _configuration["EmailSettings:Port"];
    }
}
```

Problems:

- String keys everywhere
- Easy to make typos
- Difficult to maintain
- No compile-time checking

---

# With Options Pattern

Create a class.

```csharp
public class EmailSettings
{
    public string Sender { get; set; } = string.Empty;

    public string ApiKey { get; set; } = string.Empty;

    public string Host { get; set; } = string.Empty;

    public int Port { get; set; }
}
```

---

# Bind Configuration

In `Program.cs`

```csharp
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));
```

ASP.NET Core automatically maps JSON values to the `EmailSettings` class.

---

# Inject Options

```csharp
using Microsoft.Extensions.Options;

public class EmailService
{
    private readonly EmailSettings _settings;

    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;
    }

    public void Send()
    {
        Console.WriteLine(_settings.Host);
    }
}
```

---

# How It Works

```
appsettings.json
        │
        ▼
Configuration
        │
        ▼
Bind To EmailSettings
        │
        ▼
IOptions<EmailSettings>
        │
        ▼
EmailService
```

---

# Complete Example

## Step 1 - appsettings.json

```json
{
  "DatabaseSettings": {
    "Server": "localhost",
    "Database": "ShopDB",
    "Timeout": 30
  }
}
```

---

## Step 2 - Create Class

```csharp
public class DatabaseSettings
{
    public string Server { get; set; } = string.Empty;

    public string Database { get; set; } = string.Empty;

    public int Timeout { get; set; }
}
```

---

## Step 3 - Register

```csharp
builder.Services.Configure<DatabaseSettings>(
    builder.Configuration.GetSection("DatabaseSettings"));
```

---

## Step 4 - Inject

```csharp
public class ProductRepository
{
    private readonly DatabaseSettings _settings;

    public ProductRepository(
        IOptions<DatabaseSettings> options)
    {
        _settings = options.Value;
    }
}
```

---

# Options Interfaces

ASP.NET Core provides three commonly used options interfaces.

| Interface | When to Use |
|-----------|-------------|
| `IOptions<T>` | Configuration rarely changes after startup |
| `IOptionsSnapshot<T>` | Read updated configuration for each HTTP request |
| `IOptionsMonitor<T>` | React to configuration changes while the application is running |

---

# 1. IOptions<T>

```csharp
public EmailService(
    IOptions<EmailSettings> options)
{
    var settings = options.Value;
}
```

Characteristics:

- Singleton-friendly
- Values are read once
- Most common option for stable configuration

---

# 2. IOptionsSnapshot<T>

```csharp
public ProductService(
    IOptionsSnapshot<DatabaseSettings> options)
{
}
```

Characteristics:

- Scoped
- Reads updated values on the next HTTP request
- Useful for web applications where configuration may change

---

# 3. IOptionsMonitor<T>

```csharp
public CacheService(
    IOptionsMonitor<CacheSettings> options)
{
}
```

Characteristics:

- Singleton-friendly
- Supports change notifications
- Can react immediately to configuration updates

---

# Real-World Example

An e-commerce application

```
appsettings.json

↓

PaymentSettings

↓

Options Pattern

↓

PaymentService
```

The service doesn't know where the values came from.

It simply consumes a strongly typed object.

---

# Configuration Sources

The Options Pattern can bind values from:

- `appsettings.json`
- `appsettings.Development.json`
- Environment variables
- User secrets
- Command-line arguments
- Azure App Configuration
- Azure Key Vault (often combined with configuration)

---

# Why Use Options Pattern?

Without Options

```
Configuration

↓

Magic Strings

↓

Typos
```

---

With Options

```
Configuration

↓

Strongly Typed Class

↓

Compile-Time Safety
```

---

# Validating Options

You can validate configuration during application startup.

```csharp
builder.Services
    .AddOptions<EmailSettings>()
    .Bind(builder.Configuration.GetSection("EmailSettings"))
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

This helps detect missing or invalid configuration before the application starts serving requests.

---

# Common Mistakes

## Using Magic Strings Everywhere

Bad

```csharp
_configuration["EmailSettings:Host"]
```

Repeated throughout the application.

Prefer strongly typed options.

---

## Injecting IConfiguration Everywhere

While `IConfiguration` is useful, injecting it into every service makes code harder to maintain.

Prefer `IOptions<T>` for related configuration.

---

## Forgetting to Register Options

Bad

```csharp
public EmailService(IOptions<EmailSettings> options)
```

without

```csharp
builder.Services.Configure<EmailSettings>(...);
```

The object will not be bound with the expected configuration values.

---

## Putting Business Logic in Options Classes

Options classes should contain configuration data only.

Avoid methods or business rules inside them.

---

# Best Practices

- Use strongly typed options classes.
- Group related settings together.
- Use `IOptions<T>` for stable configuration.
- Use `IOptionsSnapshot<T>` for per-request refreshed settings.
- Use `IOptionsMonitor<T>` when runtime updates are required.
- Validate configuration during startup.
- Keep options classes simple and focused.

---

# Options Pattern Workflow

```
appsettings.json
        │
        ▼
Configuration Provider
        │
        ▼
Bind Section
        │
        ▼
Strongly Typed Class
        │
        ▼
Inject Into Service
        │
        ▼
Use Configuration
```

---

# Easy Way to Remember

Think of **a passport application form**.

Instead of carrying dozens of loose papers,

everything is organized into one form.

Similarly,

instead of reading dozens of configuration keys,

the Options Pattern groups related settings into one class.

---

# Frequently Asked Interview Questions

### What is the Options Pattern?

The Options Pattern is a way to bind configuration values to strongly typed C# classes and inject them into services using Dependency Injection.

---

### Why is the Options Pattern preferred over `IConfiguration`?

It provides strong typing, reduces the use of magic strings, improves readability, and makes configuration easier to maintain and test.

---

### What is the difference between `IOptions`, `IOptionsSnapshot`, and `IOptionsMonitor`?

- `IOptions<T>` reads configuration once and is suitable for stable settings.
- `IOptionsSnapshot<T>` refreshes values for each HTTP request.
- `IOptionsMonitor<T>` supports runtime configuration changes and change notifications.

---

### Where are options typically loaded from?

Common sources include `appsettings.json`, environment variables, Azure App Configuration, Azure Key Vault, command-line arguments, and user secrets.

---

# Interview Answer (2-Minute Version)

**What is the Options Pattern in ASP.NET Core?**

The Options Pattern is the recommended way to access application configuration in ASP.NET Core. It binds configuration values from sources such as `appsettings.json` into strongly typed C# classes, which are then injected into services using Dependency Injection. This eliminates the need for string-based configuration lookups, improves readability, provides compile-time safety, and simplifies testing. ASP.NET Core offers `IOptions<T>` for stable configuration, `IOptionsSnapshot<T>` for per-request updates, and `IOptionsMonitor<T>` for runtime configuration changes, allowing developers to choose the most appropriate approach based on application requirements.