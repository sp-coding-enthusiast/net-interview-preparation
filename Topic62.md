# 61. Configuration Precedence in ASP.NET Core

One of the most important concepts in ASP.NET Core configuration is **Configuration Precedence**.

It answers the question:

> **"If the same configuration key exists in multiple places, which value does ASP.NET Core use?"**

The answer is:

> **The value from the configuration provider with the highest precedence (loaded last) wins.**

---

# What is Configuration Precedence?

## Definition

Configuration Precedence is the **order in which ASP.NET Core loads configuration providers**.

If multiple providers contain the same configuration key,

the provider loaded **later** overrides the earlier one.

---

# Layman Example: Teacher's Instructions

Imagine your teacher gives homework.

Morning

```
Solve Page 20
```

---

Later,

the teacher says

```
Instead,

Solve Page 25
```

Which instruction do you follow?

The latest one.

ASP.NET Core behaves the same way.

The **last provider loaded** wins.

---

# Another Example: GPS Navigation

Google Maps says

```
Turn Left
```

A traffic update arrives.

Now it says

```
Turn Right
```

Which one do you follow?

The latest instruction.

Configuration Precedence works the same way.

---

# Why Do We Need Configuration Precedence?

Suppose

Development

```
localhost
```

Production

```
Azure SQL
```

Should we edit code every time?

No.

Instead,

different configuration providers supply different values,

and ASP.NET Core chooses the one with the highest priority.

---

# Default Configuration Order

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
3. User Secrets (Development only)
```

↓

```
4. Environment Variables
```

↓

```
5. Command-Line Arguments
```

The **last loaded provider has the highest precedence**.

---

# Visual Representation

```
Lowest Priority

↓

appsettings.json

↓

Environment JSON

↓

User Secrets

↓

Environment Variables

↓

Command-Line Arguments

↓

Highest Priority
```

---

# Example 1

## appsettings.json

```json
{
    "Port": 5000
}
```

Environment Variable

```
Port=7000
```

Application uses

```
7000
```

Why?

Environment Variables override JSON.

---

# Example 2

## appsettings.json

```json
{
    "Port": 5000
}
```

Environment Variable

```
Port=7000
```

Command Line

```
--Port=9000
```

Application uses

```
9000
```

Because Command-Line Arguments have higher precedence.

---

# Example 3

## appsettings.json

```json
{
  "EmailSettings": {
    "Host": "smtp.gmail.com"
  }
}
```

Environment Variable

```
EmailSettings__Host = smtp.office365.com
```

Application result

```
smtp.office365.com
```

The Environment Variable overrides the JSON value.

---

# Configuration Flow

```
appsettings.json

↓

appsettings.Production.json

↓

User Secrets

↓

Environment Variables

↓

Command-Line Arguments

↓

Final Configuration
```

---

# How ASP.NET Core Merges Configuration

Imagine

Provider 1

```json
{
  "ApiUrl": "https://dev-api"
}
```

Provider 2

```
ApiUrl=https://test-api
```

Provider 3

```
--ApiUrl=https://prod-api
```

Final Value

```
https://prod-api
```

Only one value survives.

The highest-precedence provider wins.

---

# Real-World Example

Development

```
Connection String

↓

localhost
```

Production

```
Environment Variable

↓

Azure SQL
```

Emergency Deployment

```
Command Line

↓

Temporary Database
```

No code changes required.

---

# Configuration Providers

| Provider | Typical Use |
|----------|-------------|
| `appsettings.json` | Default application settings |
| `appsettings.{Environment}.json` | Environment-specific overrides |
| User Secrets | Development secrets |
| Environment Variables | Deployment and production configuration |
| Command-Line Arguments | Temporary runtime overrides |

---

# Reading Configuration

Regardless of where the value comes from,

you read it the same way.

```csharp
var value =
    builder.Configuration["Database:Server"];
```

ASP.NET Core determines which provider supplied the final value.

---

# Options Pattern

```csharp
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));
```

If an Environment Variable overrides

```
EmailSettings__Host
```

the `EmailSettings` object automatically contains the overridden value.

---

# Real Enterprise Example

Suppose

`appsettings.json`

```
Logging Level

↓

Information
```

Development

```
Logging Level

↓

Debug
```

Production

```
Environment Variable

↓

Warning
```

Application

↓

Uses

```
Warning
```

Because Environment Variables have higher precedence.

---

# Advantages

- Flexible configuration.
- No code changes between environments.
- Easy cloud deployment.
- Secure separation of secrets.
- Supports temporary runtime overrides.

---

# Common Mistakes

## Assuming `appsettings.json` Always Wins

Incorrect.

Higher-precedence providers can override it.

---

## Forgetting Environment Variables Override JSON

Many debugging issues happen because an Environment Variable silently overrides a JSON value.

---

## Hardcoding Production Values

Avoid embedding environment-specific values in code.

Use configuration providers instead.

---

# Best Practices

- Keep default values in `appsettings.json`.
- Use environment-specific JSON files for environment overrides.
- Store development secrets in User Secrets.
- Use Environment Variables for deployment-specific settings.
- Use Command-Line Arguments only for temporary runtime overrides.
- Use Azure Key Vault (or another secure secret store) for highly sensitive production secrets.

---

# Internal Working

```
Application Starts

↓

Load appsettings.json

↓

Load Environment JSON

↓

Load User Secrets

↓

Load Environment Variables

↓

Load Command-Line Arguments

↓

Merge Configuration

↓

Application Uses Final Values
```

---

# Easy Way to Remember

Imagine **writing on a whiteboard**.

First,

you write:

```
Port = 5000
```

Later,

you erase it and write:

```
Port = 7000
```

Finally,

you erase it again and write:

```
Port = 9000
```

The last value is the one everyone sees.

That's Configuration Precedence.

---

# Frequently Asked Interview Questions

### What is Configuration Precedence?

Configuration Precedence is the order in which ASP.NET Core loads configuration providers. If multiple providers define the same key, the provider loaded later overrides earlier values.

---

### Which configuration provider has the highest priority?

In the default configuration pipeline, **Command-Line Arguments** have the highest precedence.

---

### Do Environment Variables override `appsettings.json`?

Yes.

Environment Variables are loaded after JSON configuration and therefore override matching values.

---

### Do User Secrets override `appsettings.json`?

Yes.

During development, User Secrets are loaded after JSON configuration and override matching settings.

---

### Why is Configuration Precedence important?

It enables the same application to run across Development, Testing, Staging, and Production without changing code, while allowing environment-specific settings and secure secret management.

---

# Configuration Precedence Summary

| Priority (Low → High) | Configuration Provider |
|------------------------|------------------------|
| 1 | `appsettings.json` |
| 2 | `appsettings.{Environment}.json` |
| 3 | User Secrets (Development only) |
| 4 | Environment Variables |
| 5 | Command-Line Arguments |

**Rule to Remember:** **The last loaded provider wins.**

---

# Interview Answer (2-Minute Version)

**What is Configuration Precedence in ASP.NET Core?**

Configuration Precedence is the order in which ASP.NET Core loads configuration providers and resolves conflicts when the same configuration key exists in multiple sources. By default, the framework loads `appsettings.json`, followed by the environment-specific JSON file, User Secrets (during development), Environment Variables, and finally Command-Line Arguments. If the same key appears in more than one provider, the value from the provider loaded later overrides the earlier one. This layered approach allows applications to have sensible defaults while supporting environment-specific configuration, secure secret management, and runtime overrides without modifying application code.