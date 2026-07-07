# 56. Configuration Providers in ASP.NET Core

Configuration Providers are the components that **load configuration values into an ASP.NET Core application**.

Instead of hardcoding values like:

- Database connection strings
- API Keys
- SMTP settings
- Feature flags
- Application settings

ASP.NET Core reads them from different **configuration sources**, called **Configuration Providers**.

> **Interview Tip:** ASP.NET Core supports multiple configuration providers out of the box and combines them into a single configuration system. Providers added later generally have higher priority and can override values from earlier providers.

---

# What is a Configuration Provider?

## Definition

A Configuration Provider is a source from which ASP.NET Core reads configuration.

Examples:

- JSON files
- Environment Variables
- User Secrets
- Command-line Arguments
- Azure App Configuration
- Azure Key Vault
- In-memory Collections

---

# Layman Example: Personal Information

Imagine someone asks for your contact details.

You can provide them from different sources.

```
Phone Contact
```

or

```
Passport
```

or

```
Driving License
```

Different sources,

same information.

Configuration Providers work the same way.

---

# Another Example: Weather Information

You can check weather using:

```
TV
```

```
Mobile App
```

```
Website
```

```
Newspaper
```

Different sources,

same weather.

---

# Why Do We Need Configuration Providers?

Imagine this:

```csharp
string connection =
    "Server=localhost;Database=ShopDB;";
```

Hardcoded values create problems.

If the database changes,

you must rebuild and redeploy the application.

Instead,

move configuration outside the code.

---

# Configuration Flow

```
Configuration Sources

↓

Configuration Providers

↓

Configuration

↓

Application
```

---

# Default Configuration Providers

ASP.NET Core automatically loads several providers.

Typical order includes:

```
appsettings.json
```

↓

```
appsettings.{Environment}.json
```

↓

```
User Secrets (Development)
```

↓

```
Environment Variables
```

↓

```
Command-Line Arguments
```

Providers added later can override earlier values.

---

# 1. JSON Configuration Provider

Most common provider.

`appsettings.json`

```json
{
  "DatabaseSettings": {
    "Server": "localhost",
    "Database": "ShopDB"
  }
}
```

Read

```csharp
builder.Configuration["DatabaseSettings:Server"];
```

Or bind using the Options Pattern.

---

# 2. Environment Variables

Used heavily in:

- Docker
- Kubernetes
- Azure App Service
- CI/CD pipelines

Example

```
ConnectionStrings__DefaultConnection
```

ASP.NET Core maps:

```
__
```

to

```
:
```

So:

```
ConnectionStrings__DefaultConnection
```

becomes

```
ConnectionStrings:DefaultConnection
```

---

# Why Environment Variables?

Different environments need different values.

Development

```
localhost
```

Production

```
Azure SQL
```

No code changes required.

---

# 3. User Secrets

Development only.

Never store API keys in Git.

Instead,

store them locally.

Example

```
dotnet user-secrets
```

Stores

```
API Keys

Passwords

Tokens
```

Outside your project.

---

# 4. Command-Line Provider

Run application.

```
dotnet run

--Environment=Production
```

Command-line values override earlier providers.

Useful for automation and testing.

---

# 5. In-Memory Provider

Stores configuration in memory.

Example

```csharp
builder.Configuration.AddInMemoryCollection(
    new Dictionary<string, string?>
    {
        ["FeatureX"] = "true"
    });
```

Useful for:

- Unit testing
- Temporary settings
- Dynamic scenarios

---

# 6. Azure App Configuration

Large enterprises often store configuration centrally.

```
Application

↓

Azure App Configuration

↓

Settings
```

Benefits

- Centralized management
- Feature flags
- Dynamic refresh
- Multiple applications share configuration

---

# 7. Azure Key Vault

Never store secrets inside:

```
appsettings.json
```

Instead,

store them securely.

Examples

```
Passwords
```

```
API Keys
```

```
Certificates
```

```
Connection Strings
```

Azure Key Vault is commonly used for sensitive values.

---

# Configuration Hierarchy

```
JSON

↓

Environment JSON

↓

User Secrets

↓

Environment Variables

↓

Command-Line
```

Lower providers in the list have higher priority.

---

# Example

Suppose

`appsettings.json`

```json
{
    "Port": 5000
}
```

Environment Variable

```
Port = 8080
```

Result

```
8080
```

Environment Variable overrides JSON.

---

# Reading Configuration

Using `IConfiguration`

```csharp
public class EmailService
{
    private readonly IConfiguration _configuration;

    public EmailService(
        IConfiguration configuration)
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

---

# Better Approach

Use the Options Pattern.

```csharp
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));
```

Then inject:

```csharp
IOptions<EmailSettings>
```

Strongly typed.

Cleaner.

Safer.

---

# Real-World Example

E-commerce Application

```
Database

↓

Connection String
```

↓

Environment Variable

---

```
Payment API Key

↓

Azure Key Vault
```

---

```
Feature Flags

↓

Azure App Configuration
```

---

```
Application Settings

↓

appsettings.json
```

Each setting comes from the most appropriate provider.

---

# Advantages

- No hardcoded configuration
- Easy environment-specific settings
- Better security
- Supports cloud deployments
- Centralized configuration management
- Easy testing

---

# Disadvantages

- Too many providers can complicate debugging.
- Provider precedence must be understood.
- Secrets should never be stored in source control.

---

# Common Mistakes

## Hardcoding Secrets

Bad

```csharp
string apiKey = "ABC123";
```

Store secrets in a secure secret store instead.

---

## Storing Production Passwords in appsettings.json

Bad practice.

Use:

- Azure Key Vault
- Environment Variables
- Other secure secret management solutions

---

## Using IConfiguration Everywhere

Prefer the Options Pattern for related configuration settings.

---

# Best Practices

- Keep default settings in `appsettings.json`.
- Use environment-specific JSON files when needed.
- Store secrets outside source control.
- Use Environment Variables in containers and cloud environments.
- Use Azure Key Vault (or an equivalent secret manager) for sensitive values.
- Use Azure App Configuration (or an equivalent centralized service) for shared configuration and feature flags.
- Prefer strongly typed options over string lookups.

---

# Configuration Flow

```
Configuration Providers

↓

Configuration Builder

↓

IConfiguration

↓

Options Pattern

↓

Application
```

---

# Easy Way to Remember

Imagine **water coming from different sources**.

```
Rain

River

Lake

Well
```

Different sources,

same water supply.

Similarly,

```
JSON

Environment Variables

Key Vault

Azure App Configuration
```

Different providers,

one configuration system.

---

# Frequently Asked Interview Questions

### What is a Configuration Provider?

A Configuration Provider is a source from which ASP.NET Core reads configuration values, such as JSON files, environment variables, Azure Key Vault, or command-line arguments.

---

### Which configuration provider has higher priority?

Providers added later generally have higher priority and override values from earlier providers. In the default configuration, command-line arguments override environment variables, which override JSON configuration.

---

### Why are Environment Variables important?

They allow configuration to change between environments without modifying code, making them ideal for containers, cloud deployments, and CI/CD pipelines.

---

### Why use Azure Key Vault?

Azure Key Vault securely stores secrets such as passwords, API keys, certificates, and connection strings instead of keeping them in configuration files.

---

### Should you inject `IConfiguration` or use the Options Pattern?

For related application settings, the **Options Pattern** is preferred because it provides strongly typed configuration, better maintainability, and compile-time safety.

---

# Interview Answer (2-Minute Version)

**What are Configuration Providers in ASP.NET Core?**

Configuration Providers are the components that load configuration values into an ASP.NET Core application from different sources such as `appsettings.json`, environment-specific JSON files, environment variables, user secrets, command-line arguments, Azure App Configuration, Azure Key Vault, and in-memory collections. ASP.NET Core combines these providers into a unified configuration system, where providers added later can override values from earlier providers. In production applications, JSON files are commonly used for default settings, environment variables for deployment-specific configuration, Azure Key Vault for secrets, and the Options Pattern for accessing configuration through strongly typed classes.