# 59. Secret Manager in ASP.NET Core

The **Secret Manager** (also called **User Secrets**) is a feature in ASP.NET Core that allows developers to **store sensitive information securely during development** without putting it in `appsettings.json` or committing it to source control.

Examples of secrets include:

- API Keys
- Database Passwords
- JWT Signing Keys
- SMTP Passwords
- OAuth Client Secrets

> **Interview Tip:** Secret Manager is **only for local development**. It is **not encrypted** and **should not be used in production**. For production, use a secure secret store such as **Azure Key Vault**, **AWS Secrets Manager**, or **HashiCorp Vault**.

---

# What is Secret Manager?

## Definition

Secret Manager is a development-time configuration provider that stores sensitive information **outside your project folder**.

Instead of storing secrets in:

```
appsettings.json
```

they are stored in a secure location on your local machine and automatically loaded into your application during development.

---

# Layman Example: House Key

Imagine you keep your house key.

Bad idea:

```
Tape the key

↓

On the front door
```

Anyone can find it.

---

Better idea:

```
Keep the key

↓

Inside your wallet
```

Only you have access.

Secret Manager works like your wallet.

It keeps secrets away from your project files.

---

# Another Example: ATM PIN

Your ATM card is public.

```
ATM Card
```

Your PIN is secret.

```
PIN
```

You never write the PIN on the card.

Similarly,

your project can be shared,

but secrets should remain separate.

---

# Why Do We Need Secret Manager?

Suppose your `appsettings.json` contains:

```json
{
  "ConnectionStrings": {
    "DefaultConnection":
      "Server=localhost;Password=Admin123"
  }
}
```

You push your code to GitHub.

Now everyone can see your database password.

This is a serious security risk.

Instead,

store the password using Secret Manager.

---

# How Secret Manager Works

```
Project

↓

User Secrets

↓

Configuration

↓

Application
```

The application reads the secret,

but the secret is not stored in the project.

---

# Enable Secret Manager

Initialize User Secrets for your project.

```bash
dotnet user-secrets init
```

This adds a unique **UserSecretsId** to your project file.

Example

```xml
<UserSecretsId>
3d9d8f84-1234-5678-abcd-123456789abc
</UserSecretsId>
```

This ID links your project to its local secret store.

---

# Add a Secret

Store a secret from the command line.

```bash
dotnet user-secrets set "EmailSettings:Password" "MyPassword123"
```

Another example

```bash
dotnet user-secrets set "ApiKey" "ABC123XYZ"
```

The secret is stored outside the project directory.

---

# List Secrets

```bash
dotnet user-secrets list
```

Example output

```
EmailSettings:Password = MyPassword123

ApiKey = ABC123XYZ
```

---

# Remove a Secret

```bash
dotnet user-secrets remove "ApiKey"
```

---

# Clear All Secrets

```bash
dotnet user-secrets clear
```

---

# Reading Secrets

Secrets are loaded into the normal configuration system.

Example

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
        var password =
            _configuration["EmailSettings:Password"];
    }
}
```

The application does not know whether the value came from:

- `appsettings.json`
- Secret Manager
- Environment Variables

It simply reads configuration.

---

# Better Approach

Use the Options Pattern.

Configuration

```json
{
  "EmailSettings": {
    "Host": "smtp.gmail.com"
  }
}
```

Secret

```
EmailSettings:Password
```

Options class

```csharp
public class EmailSettings
{
    public string Host { get; set; } = string.Empty;

    public string Password { get; set; } = string.Empty;
}
```

Registration

```csharp
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));
```

Now the password is injected through strongly typed options.

---

# Where Are Secrets Stored?

Secrets are **not** stored inside the project.

They are stored in a user-specific location on the development machine.

The exact location depends on the operating system.

```
Project Folder

↓

User Secrets Store

↓

Application
```

---

# Configuration Hierarchy

Typical configuration order is:

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

User Secrets override JSON values during development.

---

# Real-World Example

Development

```
Database Password

↓

Secret Manager
```

Production

```
Database Password

↓

Azure Key Vault
```

Same application.

Different configuration providers.

---

# Secret Manager vs appsettings.json

| Secret Manager | appsettings.json |
|----------------|------------------|
| Stores sensitive values | Stores application configuration |
| Outside project | Inside project |
| Not committed to Git | Usually committed to Git |
| Development only | Used in all environments |

---

# Secret Manager vs Environment Variables

| Secret Manager | Environment Variables |
|----------------|----------------------|
| Development | Development and Production |
| Local machine | Operating system / hosting platform |
| Easy for developers | Common in cloud deployments |

---

# Secret Manager vs Azure Key Vault

| Secret Manager | Azure Key Vault |
|----------------|-----------------|
| Local development | Production |
| Local secret storage | Centralized secure secret storage |
| Single developer machine | Shared enterprise applications |

---

# Internal Flow

```
Application Starts

↓

Load appsettings.json

↓

Load User Secrets

↓

Merge Configuration

↓

Application Uses Secrets
```

---

# Advantages

- Keeps secrets out of source control.
- Easy to use during development.
- Integrates with ASP.NET Core configuration.
- Works seamlessly with the Options Pattern.
- Reduces accidental credential leaks.

---

# Disadvantages

- Development only.
- Not encrypted.
- Not intended for production use.
- Secrets are local to each developer.

---

# Common Mistakes

## Storing Passwords in appsettings.json

Bad

```json
{
  "ApiKey": "ABC123"
}
```

Use Secret Manager instead during development.

---

## Using Secret Manager in Production

Bad practice.

Use a production-grade secret store such as Azure Key Vault.

---

## Committing Secrets to Git

Never commit passwords, API keys, or connection strings containing credentials to source control.

---

# Best Practices

- Use Secret Manager for local development secrets.
- Keep non-sensitive defaults in `appsettings.json`.
- Use Environment Variables or Azure Key Vault in production.
- Access secrets through the Options Pattern where appropriate.
- Rotate secrets periodically.

---

# Visual Representation

```
Developer

↓

Secret Manager

↓

Configuration

↓

Application
```

Production

```
Azure Key Vault

↓

Configuration

↓

Application
```

---

# Easy Way to Remember

Imagine **a locker at the gym**.

Your clothes go in the locker.

The locker key stays with you.

You don't leave valuables lying around.

Similarly,

Secret Manager keeps sensitive values away from your project files.

---

# Frequently Asked Interview Questions

### What is Secret Manager?

Secret Manager (User Secrets) is a development-time configuration provider used to store sensitive information outside the project directory.

---

### Is Secret Manager secure for production?

No.

It is designed only for local development and is not a production secret management solution.

---

### Where are secrets stored?

They are stored outside the project in a user-specific location managed by the .NET SDK.

---

### Does Secret Manager override `appsettings.json`?

Yes.

In the default configuration pipeline, User Secrets are loaded after `appsettings.json`, so they override matching values during development.

---

### What should be used instead of Secret Manager in production?

Use a secure secret management solution such as **Azure Key Vault**, **AWS Secrets Manager**, or **HashiCorp Vault**.

---

# Interview Answer (2-Minute Version)

**What is Secret Manager in ASP.NET Core?**

Secret Manager, also known as User Secrets, is a development-time configuration provider that stores sensitive information such as API keys, database passwords, and OAuth client secrets outside the project directory. This prevents secrets from being committed to source control while allowing applications to access them through the standard ASP.NET Core configuration system. User Secrets are loaded after `appsettings.json`, so they override matching configuration values during development. Secret Manager is intended only for local development and is not a production security solution. For production environments, services such as Azure Key Vault, AWS Secrets Manager, or HashiCorp Vault should be used instead.