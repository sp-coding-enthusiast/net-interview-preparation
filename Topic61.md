# 60. Azure Key Vault in ASP.NET Core

**Azure Key Vault** is a cloud service provided by Microsoft Azure to **securely store and manage sensitive information**, such as:

- Passwords
- API Keys
- Connection Strings
- Certificates
- Encryption Keys
- Secrets used by applications

Instead of storing secrets in:

- `appsettings.json`
- Source code
- Environment Variables (for highly sensitive values)

production applications retrieve them securely from **Azure Key Vault**.

> **Interview Tip:** Azure Key Vault is the **recommended production solution** for managing secrets in Azure-hosted ASP.NET Core applications.

---

# What is Azure Key Vault?

## Definition

Azure Key Vault is a **managed Azure service** that securely stores secrets and allows authorized applications to retrieve them at runtime.

Think of it as a **highly secure digital vault**.

---

# Layman Example: Bank Locker

Imagine you own valuable jewelry.

Would you keep it:

```
On the Dining Table?
```

No.

You keep it in a:

```
Bank Locker
```

Only authorized people with proper access can open it.

Azure Key Vault works exactly like that.

Instead of storing passwords in your project,

you store them in a secure vault.

---

# Another Example: Office Safe

Imagine an office.

Employees don't keep company cash on their desks.

Instead,

everything goes into a secure safe.

```
Office Safe

↓

Authorized Employees

↓

Cash
```

Azure Key Vault is the secure safe for your application secrets.

---

# Why Do We Need Azure Key Vault?

Suppose your project contains:

```json
{
  "ConnectionStrings": {
    "DefaultConnection":
      "Server=SQL01;Password=Admin123"
  }
}
```

Problems

- Password visible in source code repository.
- Difficult to rotate secrets.
- Security risk.
- Compliance issues.

Instead,

store the password in Azure Key Vault.

---

# What Can Azure Key Vault Store?

## 1. Secrets

Examples

- Database passwords
- API Keys
- OAuth client secrets
- JWT signing secrets

---

## 2. Keys

Examples

- Encryption keys
- RSA keys
- AES keys

Used for cryptographic operations.

---

## 3. Certificates

Examples

- SSL/TLS certificates
- Client authentication certificates

---

# Architecture

```
ASP.NET Core App

↓

Managed Identity

↓

Azure Key Vault

↓

Secrets
```

The application never stores the password.

It retrieves it securely when needed.

---

# How Azure Key Vault Works

```
Application Starts

↓

Authenticate

↓

Access Key Vault

↓

Retrieve Secret

↓

Use Secret
```

---

# Authentication

Applications must authenticate before accessing Key Vault.

Common options:

- **Managed Identity** (recommended for Azure-hosted applications)
- Service Principal
- Developer credentials (during local development)

---

# Managed Identity

Instead of storing credentials inside your application,

Azure provides the application with an identity.

```
Application

↓

Managed Identity

↓

Azure Key Vault
```

No username.

No password.

No client secret inside your code.

---

# Configuration Flow

```
Azure Key Vault

↓

Configuration Provider

↓

IConfiguration

↓

Options Pattern

↓

Application
```

The application reads secrets just like any other configuration value.

---

# Example

Suppose Key Vault contains

```
ConnectionStrings--DefaultConnection
```

```
ApiKey
```

```
JwtSecret
```

Your application retrieves these values securely during startup.

---

# Reading Secrets

After Key Vault is added as a configuration provider,

you can use:

```csharp
var apiKey =
    builder.Configuration["ApiKey"];
```

Or, preferably,

bind them using the Options Pattern.

The application does not need to know whether the value came from:

- `appsettings.json`
- Environment Variables
- Azure Key Vault

---

# Options Pattern

```csharp
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));
```

If Key Vault provides

```
EmailSettings:Password
```

the options object receives it automatically.

---

# Real-World Example

E-commerce Application

```
Payment API Key

↓

Azure Key Vault
```

---

```
Database Password

↓

Azure Key Vault
```

---

```
SMTP Password

↓

Azure Key Vault
```

Application

↓

Reads secrets securely.

---

# Development vs Production

Development

```
User Secrets
```

↓

Application

---

Production

```
Azure Key Vault
```

↓

Application

---

# Azure Key Vault vs appsettings.json

| Azure Key Vault | appsettings.json |
|-----------------|------------------|
| Secure secret storage | General application configuration |
| Production-ready | Base configuration |
| Access controlled | Usually stored in source control |
| Supports auditing and rotation | No built-in secret management |

---

# Azure Key Vault vs Secret Manager

| Secret Manager | Azure Key Vault |
|----------------|-----------------|
| Development | Production |
| Local machine | Cloud |
| Individual developer | Enterprise applications |
| Not centralized | Centralized secret management |

---

# Azure Key Vault vs Environment Variables

| Azure Key Vault | Environment Variables |
|-----------------|----------------------|
| Centralized | Local to each host/container |
| Fine-grained access control | Managed by the OS or hosting platform |
| Secret rotation | Manual updates typically required |
| Auditing support | Limited |

---

# Internal Flow

```
Application Starts

↓

Authenticate

↓

Azure Key Vault

↓

Retrieve Secrets

↓

Configuration

↓

Application
```

---

# Advantages

- Secure secret storage.
- Centralized management.
- Fine-grained access control using Azure Identity and Azure RBAC/access policies.
- Secret versioning.
- Secret rotation support.
- Audit logging.
- Integrates with ASP.NET Core configuration.
- No secrets stored in source code.

---

# Disadvantages

- Azure dependency.
- Network access required.
- Additional setup and permissions.
- Potential latency if secrets are fetched repeatedly (applications typically cache configuration after startup).

---

# Common Mistakes

## Storing Passwords in appsettings.json

Bad

```json
{
    "Password": "Admin123"
}
```

Use Azure Key Vault for production secrets.

---

## Hardcoding API Keys

Bad

```csharp
string apiKey = "ABC123";
```

Retrieve the key from Azure Key Vault instead.

---

## Using Secret Manager in Production

Secret Manager is for development.

Production should use a secure secret management solution such as Azure Key Vault.

---

# Best Practices

- Store only sensitive values in Azure Key Vault.
- Keep general application settings in `appsettings.json`.
- Use **Managed Identity** whenever possible.
- Use the Options Pattern for strongly typed configuration.
- Rotate secrets regularly.
- Grant applications only the minimum required permissions (principle of least privilege).

---

# Visual Representation

```
ASP.NET Core App

↓

Managed Identity

↓

Azure Key Vault

↓

Secrets

↓

Application
```

---

# Easy Way to Remember

Imagine **a bank locker**.

Your valuables stay inside the bank.

You don't carry all your jewelry every day.

When you need it,

you unlock the locker,

use it,

and secure it again.

Azure Key Vault is that secure locker for your application's secrets.

---

# Frequently Asked Interview Questions

### What is Azure Key Vault?

Azure Key Vault is a managed Azure service used to securely store and manage secrets, encryption keys, and certificates.

---

### Why use Azure Key Vault?

It protects sensitive information, centralizes secret management, supports auditing and rotation, and prevents secrets from being stored in source code.

---

### What authentication method is recommended for Azure-hosted applications?

**Managed Identity** is the recommended approach because it eliminates the need to store credentials in the application.

---

### Can Azure Key Vault integrate with ASP.NET Core configuration?

Yes.

Azure Key Vault can be added as a configuration provider, allowing secrets to be accessed through `IConfiguration` or the Options Pattern.

---

### What should be stored in Azure Key Vault?

Sensitive values such as API keys, passwords, certificates, encryption keys, OAuth client secrets, and database credentials.

---

# Interview Answer (2-Minute Version)

**What is Azure Key Vault, and why is it used in ASP.NET Core?**

Azure Key Vault is a managed Azure service that securely stores and manages sensitive information such as API keys, database passwords, certificates, and encryption keys. Instead of storing secrets in `appsettings.json` or source code, ASP.NET Core applications retrieve them securely from Azure Key Vault at runtime. The recommended authentication mechanism for Azure-hosted applications is **Managed Identity**, which removes the need to store credentials in the application itself. Azure Key Vault integrates directly with the ASP.NET Core configuration system, allowing secrets to be accessed through `IConfiguration` or the Options Pattern while supporting centralized management, auditing, access control, and secret rotation.