# 64. launchSettings.json in ASP.NET Core

`launchSettings.json` is a **development-only configuration file** used to define **how an ASP.NET Core application starts when running locally** from Visual Studio, Visual Studio Code, or the .NET CLI.

It is commonly used to configure:

- Application URLs
- Environment (`Development`, `Production`, etc.)
- Launch Browser
- Environment Variables
- IIS Express settings
- Startup profiles

> **Interview Tip:** `launchSettings.json` is **not used in production**. It is only used during local development and debugging.

---

# What is launchSettings.json?

## Definition

`launchSettings.json` is a file located inside the **Properties** folder of an ASP.NET Core project.

Typical location

```
Project

│

├── Controllers

├── Models

├── Program.cs

├── appsettings.json

└── Properties
      └── launchSettings.json
```

---

# Layman Example: Car Seat Settings

Imagine two people drive the same car.

Driver A likes:

- Seat forward
- Mirrors adjusted
- AC at 22°C

Driver B likes:

- Seat backward
- Mirrors different
- AC at 18°C

The **car remains the same**.

Only the startup preferences change.

`launchSettings.json` works exactly like that.

It defines **how the application starts during development**.

---

# Why Do We Need launchSettings.json?

Suppose every time you run your application you manually specify:

- Port number
- Browser
- Environment
- HTTPS URL

That would be tedious.

Instead,

store these settings once in `launchSettings.json`.

---

# Example launchSettings.json

```json
{
  "profiles": {
    "MyWebApi": {
      "commandName": "Project",
      "launchBrowser": true,
      "applicationUrl": "https://localhost:7001;http://localhost:5000",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

---

# Main Sections

```
launchSettings.json

↓

Profiles

↓

Environment Variables

↓

Application URLs

↓

Browser Settings
```

---

# Profiles

A profile defines **how the application starts**.

Example

```json
"profiles": {
  "MyWebApi": {
  }
}
```

You can have multiple profiles.

Example

```
Development
```

```
Docker
```

```
IIS Express
```

```
HTTPS
```

Each profile can use different settings.

---

# commandName

Example

```json
"commandName": "Project"
```

Common values include:

| Value | Meaning |
|--------|----------|
| `Project` | Launches the ASP.NET Core application using Kestrel |
| `IISExpress` | Launches using IIS Express (Visual Studio on Windows) |
| `Executable` | Starts an external executable |

---

# applicationUrl

Example

```json
"applicationUrl":
"https://localhost:7001;http://localhost:5000"
```

This tells Kestrel to listen on:

```
https://localhost:7001
```

and

```
http://localhost:5000
```

during development.

---

# launchBrowser

Example

```json
"launchBrowser": true
```

When the application starts,

the default browser opens automatically.

If

```json
false
```

the browser does not open.

---

# Environment Variables

Example

```json
"environmentVariables": {
    "ASPNETCORE_ENVIRONMENT": "Development"
}
```

This sets

```
Development
```

when running locally.

Now ASP.NET Core automatically loads

```
appsettings.Development.json
```

---

# Multiple Profiles

Example

```json
{
  "profiles": {
    "Development": {
      "applicationUrl": "https://localhost:7001"
    },

    "Testing": {
      "applicationUrl": "https://localhost:8001"
    }
  }
}
```

Each profile starts the application differently.

---

# Launch Flow

```
Developer

↓

Run Application

↓

launchSettings.json

↓

Choose Profile

↓

Set URLs

↓

Set Environment

↓

Start Kestrel

↓

Application Running
```

---

# Relationship with appsettings.json

Many developers confuse these files.

| launchSettings.json | appsettings.json |
|---------------------|------------------|
| Controls how the app starts | Stores application configuration |
| Development only | Used in all environments |
| Not deployed | Usually deployed with the application |
| Contains launch profiles | Contains application settings |

---

# Relationship with Environment Variables

Suppose

```
launchSettings.json

↓

ASPNETCORE_ENVIRONMENT

↓

Development
```

Then ASP.NET Core automatically loads

```
appsettings.Development.json
```

This is why your development configuration works automatically.

---

# Real-World Example

Development Machine

```
Browser Opens

↓

https://localhost:7001

↓

Environment

↓

Development
```

Production Server

```
launchSettings.json

↓

Ignored
```

The hosting platform determines the URLs and environment.

---

# Does Production Use launchSettings.json?

**No.**

Production hosting environments such as:

- Azure App Service
- IIS
- Docker
- Kubernetes
- Linux Servers

ignore `launchSettings.json`.

Instead,

they use:

- Environment Variables
- Hosting configuration
- Container configuration
- Web server configuration

---

# Advantages

- Easy local development.
- Multiple launch profiles.
- Automatic browser launch.
- Local environment variable configuration.
- No need to remember startup commands.

---

# Disadvantages

- Development only.
- Not deployed to production.
- Changes affect only local startup behavior.

---

# Common Mistakes

## Storing Secrets

Bad

```json
"Password":"Admin123"
```

Never store passwords or API keys in `launchSettings.json`.

Use:

- Secret Manager
- Azure Key Vault
- Environment Variables

---

## Assuming Production Uses launchSettings.json

It does not.

Production ignores this file.

---

## Editing Ports Without Updating References

If you change

```
5000

↓

8000
```

make sure clients and documentation use the new port.

---

# Best Practices

- Use `launchSettings.json` only for local development.
- Create separate launch profiles when needed.
- Set `ASPNETCORE_ENVIRONMENT` to `Development` for local debugging.
- Do not store secrets.
- Let production hosting configure ports and environment.

---

# Visual Representation

```
Visual Studio / dotnet run

↓

launchSettings.json

↓

Profile

↓

Environment

↓

Kestrel

↓

Application
```

---

# Easy Way to Remember

Imagine **a TV remote**.

The TV is the application.

The remote controls:

- Volume
- Channel
- Brightness

It doesn't change the TV itself.

Similarly,

`launchSettings.json` controls **how the application starts**, not how the application works.

---

# Frequently Asked Interview Questions

### What is `launchSettings.json`?

`launchSettings.json` is a development-only configuration file that defines how an ASP.NET Core application starts locally, including launch profiles, URLs, browser settings, and environment variables.

---

### Where is `launchSettings.json` located?

It is typically located inside the **Properties** folder of an ASP.NET Core project.

---

### Is `launchSettings.json` deployed to production?

No.

It is ignored in production environments.

---

### What is the purpose of `applicationUrl`?

It specifies the HTTP and HTTPS URLs that Kestrel listens on during local development.

---

### Why is `ASPNETCORE_ENVIRONMENT` defined in `launchSettings.json`?

It sets the runtime environment (such as `Development`), allowing ASP.NET Core to load the appropriate environment-specific configuration file like `appsettings.Development.json`.

---

# Interview Answer (2-Minute Version)

**What is `launchSettings.json` in ASP.NET Core?**

`launchSettings.json` is a development-only configuration file located in the project's **Properties** folder. It defines how the application starts during local development, including launch profiles, application URLs, browser launch behavior, and environment variables such as `ASPNETCORE_ENVIRONMENT`. When the application is started from Visual Studio or by using `dotnet run`, these settings determine how Kestrel is configured for local execution. The file is not deployed or used in production, where startup behavior is controlled by the hosting environment, environment variables, and server configuration.