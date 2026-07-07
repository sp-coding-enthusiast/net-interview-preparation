# 57. appsettings.json Hierarchy in ASP.NET Core

One of the most frequently asked ASP.NET Core interview questions is:

> **"How does `appsettings.json` hierarchy work?"**

The answer is based on **configuration precedence**.

ASP.NET Core can read configuration from **multiple files and providers**, and if the same setting exists in more than one place, **the value from the higher-priority provider overrides the lower-priority one**.

> **Interview Tip:** Think of `appsettings.json` as the **default configuration**, while environment-specific files and other providers override it as needed.

---

# What is appsettings.json?

`appsettings.json` is the **default configuration file** for an ASP.NET Core application.

It stores values such as:

- Database connection strings
- Logging configuration
- API URLs
- SMTP settings
- Feature flags
- Application settings

Example

```json
{
  "ApplicationName": "ShoppingApp",

  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=ShopDB;"
  },

  "EmailSettings": {
    "Host": "smtp.gmail.com",
    "Port": 587
  }
}
```

---

# Why Do We Need Hierarchy?

Imagine one application running in three environments.

```
Development
```

```
Testing
```

```
Production
```

Should all three use:

```
localhost
```

No.

Each environment needs different values.

Instead of changing code,

ASP.NET Core loads different configuration files.

---

# Layman Example: School Uniform

Imagine a school uniform.

Default

```
White Shirt

Blue Pants
```

---

Winter

```
White Shirt

Blue Pants

Sweater
```

---

Sports Day

```
White Shirt

Blue Pants

Sports Shoes
```

The base uniform stays the same.

Only specific items change.

That's exactly how the configuration hierarchy works.

---

# Configuration Files

Normally you'll see:

```
appsettings.json
```

↓

```
appsettings.Development.json
```

↓

```
appsettings.Staging.json
```

↓

```
appsettings.Production.json
```

Only the file matching the current environment is loaded in addition to the base file.

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

## appsettings.Production.json

```json
{
  "Database": {
    "Server": "AzureSQL"
  }
}
```

---

Application running in Production

Result

```
AzureSQL
```

Production overrides the default.

---

# Another Example

## appsettings.json

```json
{
  "Logging": {
    "Level": "Information"
  }
}
```

---

## appsettings.Development.json

```json
{
  "Logging": {
    "Level": "Debug"
  }
}
```

Development result

```
Debug
```

Production result

```
Information
```

because there is no production override.

---

# Configuration Hierarchy

```
appsettings.json

↓

Environment File

↓

User Secrets

↓

Environment Variables

↓

Command-Line Arguments
```

The lower the provider,

the higher the priority.

---

# Default Loading Order

ASP.NET Core typically loads configuration in this order:

```
1. appsettings.json
```

↓

```
2. appsettings.{Environment}.json
```

↓

```
3. User Secrets (Development)
```

↓

```
4. Environment Variables
```

↓

```
5. Command-Line Arguments
```

Later providers override earlier ones.

---

# Visual Representation

```
Default

↓

Development

↓

Environment Variable

↓

Command Line
```

Highest priority wins.

---

# Example of Override

Suppose

## appsettings.json

```json
{
    "Port": 5000
}
```

---

Environment Variable

```
Port=7000
```

---

Command Line

```
--Port=9000
```

Final Value

```
9000
```

Command-line arguments override all earlier providers.

---

# Nested Configuration

Example

```json
{
  "EmailSettings": {
    "Host": "smtp.gmail.com",
    "Port": 587
  }
}
```

Read using

```csharp
builder.Configuration["EmailSettings:Host"];
```

Or, preferably,

bind to a strongly typed options class.

---

# Environment Variable Mapping

JSON

```json
{
  "EmailSettings": {
    "Host": "smtp.gmail.com"
  }
}
```

Environment Variable

```
EmailSettings__Host
```

ASP.NET Core maps

```
__
```

to

```
:
```

So it overrides

```
EmailSettings:Host
```

---

# Environment Selection

The active environment is controlled by the `ASPNETCORE_ENVIRONMENT` (or `DOTNET_ENVIRONMENT`) environment variable.

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

If the environment is set to `Production`,

ASP.NET Core loads:

```
appsettings.json
```

and

```
appsettings.Production.json
```

---

# Internal Working

```
Application Starts

↓

Read appsettings.json

↓

Read Environment File

↓

Read User Secrets

↓

Read Environment Variables

↓

Read Command-Line Arguments

↓

Merge Everything

↓

Application Uses Final Values
```

---

# Real-World Example

Suppose

`appsettings.json`

```json
{
  "PaymentApi": {
    "Url": "https://dev-api"
  }
}
```

Production

```json
{
  "PaymentApi": {
    "Url": "https://prod-api"
  }
}
```

Development

```
https://dev-api
```

Production

```
https://prod-api
```

Same code.

Different configuration.

---

# Advantages

- No code changes between environments.
- Environment-specific configuration.
- Supports secure secret management.
- Easy cloud deployment.
- Cleaner configuration management.

---

# Common Mistakes

## Editing appsettings.json for Production

Bad practice.

Keep `appsettings.json` as the base configuration and override values through environment-specific files or other providers.

---

## Storing Passwords in appsettings.json

Bad.

Use:

- User Secrets (development)
- Environment Variables
- Azure Key Vault or another secure secret store

---

## Forgetting Provider Priority

Many developers expect the value from `appsettings.json` to win.

Actually,

later providers override earlier ones.

---

# Best Practices

- Keep common settings in `appsettings.json`.
- Store only environment-specific overrides in `appsettings.{Environment}.json`.
- Never commit production secrets to source control.
- Use environment variables for deployment-specific configuration.
- Use Azure Key Vault (or another secure secret manager) for sensitive values.
- Prefer the Options Pattern for accessing configuration.

---

# Hierarchy Diagram

```
appsettings.json
        │
        ▼
appsettings.Development.json
        │
        ▼
User Secrets
        │
        ▼
Environment Variables
        │
        ▼
Command-Line Arguments
        │
        ▼
Final Configuration
```

---

# Easy Way to Remember

Imagine **painting a wall**.

First,

you apply a **base coat**.

```
Base Coat
```

Then,

you add another coat.

```
Blue Paint
```

Finally,

you add decorations.

```
Wall Art
```

The latest layer is what you see.

Similarly,

ASP.NET Core keeps applying configuration layers,

and the **last applicable value wins**.

---

# Frequently Asked Interview Questions

### What is `appsettings.json`?

It is the default configuration file used to store application settings such as connection strings, logging, API URLs, and feature flags.

---

### What is `appsettings.Development.json`?

It contains configuration values that override `appsettings.json` when the application runs in the Development environment.

---

### Which file has higher priority?

`appsettings.{Environment}.json` overrides `appsettings.json`.

Environment variables override both.

Command-line arguments override them all.

---

### Can multiple providers define the same key?

Yes.

The value from the provider with the highest priority is used.

---

### How is the active environment selected?

By setting the `ASPNETCORE_ENVIRONMENT` (or `DOTNET_ENVIRONMENT`) environment variable to values such as `Development`, `Staging`, or `Production`.

---

# Interview Answer (2-Minute Version)

**Explain the `appsettings.json` hierarchy in ASP.NET Core.**

`appsettings.json` is the base configuration file that contains default application settings. ASP.NET Core then loads an environment-specific configuration file such as `appsettings.Development.json` or `appsettings.Production.json`, depending on the current environment, and values in that file override the defaults. Additional configuration providers, including User Secrets (in development), environment variables, and command-line arguments, are loaded afterward, with later providers taking precedence over earlier ones. This layered configuration approach allows the same application code to run in different environments without modification while keeping environment-specific settings and secrets separate from the base configuration.