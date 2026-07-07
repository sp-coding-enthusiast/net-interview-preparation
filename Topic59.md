# 58. Environment Variables in ASP.NET Core

Environment Variables are one of the **most important configuration providers** in ASP.NET Core.

They allow you to **change application settings without modifying the code or configuration files**.

Instead of storing configuration inside:

- `appsettings.json`
- Source code

you store it in the **operating system or hosting environment**.

> **Interview Tip:** Environment Variables are heavily used in **Docker, Kubernetes, Azure App Service, AWS, CI/CD pipelines, and production deployments** because they separate configuration from the application.

---

# What are Environment Variables?

## Definition

Environment Variables are **key-value pairs** maintained by the operating system or hosting platform.

Example

```
DatabaseServer = SQLPROD01
```

```
ApiKey = ABC123XYZ
```

```
ASPNETCORE_ENVIRONMENT = Production
```

ASP.NET Core automatically reads these values through its configuration system.

---

# Layman Example: Hotel Room Number

Imagine you check into a hotel.

Instead of changing who you are,

the receptionist gives you:

```
Room Number = 305
```

Tomorrow,

you visit another hotel.

Now you get:

```
Room Number = 702
```

You are the same person.

Only your room number changes.

Environment Variables work the same way.

The application stays the same.

Only the configuration changes.

---

# Another Example: GPS

Suppose your GPS asks:

```
Where are you?
```

Depending on your location,

it gives different directions.

You don't change the GPS software.

You only change the environment.

---

# Why Do We Need Environment Variables?

Suppose your code contains:

```csharp
string connectionString =
    "Server=localhost;Database=ShopDB;";
```

Problems

- Must recompile when the server changes.
- Hard to deploy across environments.
- Unsafe for secrets.
- Difficult to automate.

Instead,

move the value outside the application.

---

# Example

Development

```
Server = localhost
```

Production

```
Server = SQLPROD01
```

Same application.

Different environment.

No code changes.

---

# Configuration Hierarchy

Environment Variables are one of several configuration providers.

Typical precedence is:

```
appsettings.json
```

↓

```
appsettings.{Environment}.json
```

↓

```
User Secrets
```

↓

```
Environment Variables
```

↓

```
Command-Line Arguments
```

Environment Variables override JSON configuration.

---

# Example

## appsettings.json

```json
{
  "Database": {
    "Server": "localhost"
  }
}
```

---

Environment Variable

```
Database__Server = SQLPROD01
```

Application result

```
SQLPROD01
```

The Environment Variable overrides the JSON value.

---

# Double Underscore (`__`)

JSON uses `:` to separate sections.

Example

```json
{
  "EmailSettings": {
    "Host": "smtp.gmail.com"
  }
}
```

Configuration key

```
EmailSettings:Host
```

Most operating systems don't allow `:` in environment variable names consistently.

ASP.NET Core solves this by using:

```
__
```

So the Environment Variable becomes:

```
EmailSettings__Host
```

ASP.NET Core automatically converts:

```
__
```

↓

```
:
```

---

# Reading Environment Variables

You can access them through `IConfiguration`.

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
        var host =
            _configuration["EmailSettings:Host"];
    }
}
```

The value may come from:

- JSON
- Environment Variable
- Command Line

The application doesn't need to know which provider supplied it.

---

# Better Approach

Use the Options Pattern.

Configuration

```json
{
  "EmailSettings": {
    "Host": "smtp.gmail.com",
    "Port": 587
  }
}
```

Options class

```csharp
public class EmailSettings
{
    public string Host { get; set; } = string.Empty;

    public int Port { get; set; }
}
```

Registration

```csharp
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));
```

If an Environment Variable overrides `EmailSettings__Host`,

the options object automatically receives the updated value.

---

# Common Environment Variables

## ASPNETCORE_ENVIRONMENT

Controls which environment the application runs in.

Examples

```
Development
```

```
Staging
```

```
Production
```

---

## Connection String

```
ConnectionStrings__DefaultConnection
```

---

## Logging

```
Logging__LogLevel__Default
```

---

## Feature Flag

```
Features__NewCheckout = true
```

---

# Real-World Example

E-commerce Application

Development

```
Database

↓

localhost
```

Production

```
Database

↓

Azure SQL
```

Docker

```
Database

↓

Container Database
```

The same application works everywhere.

Only Environment Variables change.

---

# Docker Example

Suppose your container starts with:

```
ConnectionStrings__DefaultConnection

=

Server=sql;
Database=ShopDB;
```

Your application automatically uses that connection string.

No code changes.

No JSON changes.

---

# Kubernetes Example

Kubernetes commonly injects configuration into containers using Environment Variables or ConfigMaps and Secrets.

```
Kubernetes

↓

Environment Variables

↓

ASP.NET Core
```

---

# Azure App Service

Azure App Service lets you define **Application Settings**.

These are exposed to your application as Environment Variables.

```
Azure Portal

↓

Application Settings

↓

Environment Variables

↓

ASP.NET Core
```

---

# Advantages

- No code changes between environments.
- Easy cloud deployment.
- Secure separation of configuration.
- Ideal for containers and CI/CD.
- Overrides JSON configuration when necessary.

---

# Disadvantages

- Large numbers of Environment Variables can become difficult to manage.
- Naming mistakes (especially missing `__`) can prevent values from being read.
- Sensitive values still require proper access control on the hosting platform.

---

# Common Mistakes

## Hardcoding Production Values

Bad

```csharp
Server=SQLPROD01
```

Use Environment Variables instead.

---

## Using `:` Instead of `__`

Bad

```
Database:Server
```

Preferred

```
Database__Server
```

This works consistently across platforms.

---

## Storing Secrets in Source Control

Avoid placing production passwords or API keys in `appsettings.json`.

Use:

- Environment Variables
- Azure Key Vault
- Another secure secret management solution

---

# Best Practices

- Store environment-specific settings in Environment Variables.
- Keep common defaults in `appsettings.json`.
- Use the Options Pattern for strongly typed configuration.
- Use secret management solutions for highly sensitive values.
- Document Environment Variables required by your application.

---

# Internal Flow

```
Operating System

↓

Environment Variables

↓

Configuration Provider

↓

IConfiguration

↓

Options Pattern

↓

Application
```

---

# Visual Representation

```
appsettings.json

↓

Environment Variables

↓

Final Configuration

↓

Application
```

Environment Variables override JSON values when both define the same key.

---

# Easy Way to Remember

Imagine **changing your clothes for the weather**.

Summer

```
T-Shirt
```

Winter

```
Jacket
```

You don't change yourself.

You only adapt to the environment.

Similarly,

the application stays the same,

but Environment Variables adapt it to different environments.

---

# Frequently Asked Interview Questions

### What are Environment Variables?

Environment Variables are operating system or hosting platform key-value pairs used to configure an application without modifying its source code or configuration files.

---

### Why are Environment Variables preferred in production?

They allow environment-specific configuration, integrate well with cloud platforms and containers, and keep deployment settings separate from the application code.

---

### Why do we use `__` instead of `:`?

ASP.NET Core maps double underscores (`__`) to colons (`:`) because `:` is not consistently supported in environment variable names across operating systems.

---

### Do Environment Variables override `appsettings.json`?

Yes.

In the default configuration pipeline, Environment Variables are loaded after `appsettings.json`, so they override matching values.

---

### Where are Environment Variables commonly used?

They are widely used in Docker, Kubernetes, Azure App Service, AWS, CI/CD pipelines, and other cloud-hosted environments.

---

# Interview Answer (2-Minute Version)

**What are Environment Variables in ASP.NET Core?**

Environment Variables are a configuration provider that allows application settings to be supplied by the operating system or hosting platform instead of hardcoding them in source code or storing them in configuration files. They are commonly used for environment-specific settings such as database connection strings, API endpoints, feature flags, and the `ASPNETCORE_ENVIRONMENT` value. In the default ASP.NET Core configuration pipeline, Environment Variables are loaded after `appsettings.json`, so they override matching settings. Nested configuration keys use double underscores (`__`), which ASP.NET Core automatically maps to colons (`:`). Environment Variables are widely used in production deployments, Docker, Kubernetes, Azure App Service, and CI/CD pipelines because they separate configuration from application code.