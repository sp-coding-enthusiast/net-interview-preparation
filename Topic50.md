# 50. IOptions vs IOptionsSnapshot vs IOptionsMonitor in ASP.NET Core

One of the most common ASP.NET Core interview questions is:

> **"What is the difference between `IOptions`, `IOptionsSnapshot`, and `IOptionsMonitor`?"**

All three are used with the **Options Pattern**, but they differ in **how and when they read configuration values**.

Choosing the right one depends on whether your configuration:

- Never changes
- Changes per request
- Can change while the application is running

---

# Layman Example: Weather App

Imagine you check today's weather.

There are three ways.

## Option 1 - Printed Newspaper

```
Morning Newspaper

↓

Read Weather

↓

Weather Never Changes
```

Even if it rains later,

your newspaper doesn't update.

This is **IOptions**.

---

## Option 2 - Check Website Every Time

```
Open Weather Website

↓

Latest Weather
```

Every time you check,

you get updated information.

This is **IOptionsSnapshot**.

---

## Option 3 - Live Weather App

```
Weather App Open

↓

Rain Starts

↓

Notification Appears

↓

Screen Updates Automatically
```

This is **IOptionsMonitor**.

---

# What Are These?

ASP.NET Core provides three interfaces:

```
IOptions<T>

↓

Read Once
```

```
IOptionsSnapshot<T>

↓

Read Once Per HTTP Request
```

```
IOptionsMonitor<T>

↓

Automatically Updates While App Is Running
```

---

# Sample Configuration

`appsettings.json`

```json
{
  "EmailSettings": {
    "Host": "smtp.gmail.com",
    "Port": 587
  }
}
```

---

Configuration class

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

---

# 1. IOptions<T>

## Definition

`IOptions<T>` reads configuration **once** when the application starts (or when the options instance is first created).

It **does not automatically reflect configuration changes** while the application is running.

---

Example

```csharp
public class EmailService
{
    private readonly EmailSettings _settings;

    public EmailService(IOptions<EmailSettings> options)
    {
        _settings = options.Value;
    }
}
```

---

How it works

```
Application Starts

↓

Read Configuration

↓

Store Values

↓

Always Use Same Values
```

---

Example

Application starts.

```
Host

↓

smtp.gmail.com
```

You change `appsettings.json`

```
Host

↓

smtp.office365.com
```

Application still uses

```
smtp.gmail.com
```

until it is restarted (or recreated).

---

Use When

- Configuration rarely changes
- API keys
- SMTP settings
- Connection strings
- Application constants

---

Advantages

- Simple
- Fast
- Works with Singleton services

---

Disadvantages

- Doesn't automatically reflect configuration changes

---

# 2. IOptionsSnapshot<T>

## Definition

`IOptionsSnapshot<T>` creates a **new snapshot of configuration for each HTTP request**.

If configuration changes,

the next request receives the updated values.

---

Example

```csharp
public class EmailService
{
    private readonly EmailSettings _settings;

    public EmailService(
        IOptionsSnapshot<EmailSettings> options)
    {
        _settings = options.Value;
    }
}
```

---

How it works

```
Request 1

↓

Read Configuration

↓

Snapshot A
```

---

```
Configuration Changes
```

---

```
Request 2

↓

Read Configuration Again

↓

Snapshot B
```

---

Example

Request 1

```
Host

↓

smtp.gmail.com
```

Configuration changes.

Request 2

```
Host

↓

smtp.office365.com
```

New request receives new values.

---

Use When

- ASP.NET Core web applications
- Per-request configuration
- Feature flags refreshed between requests
- Request-based settings

---

Advantages

- Updated values for every request
- Easy to use

---

Disadvantages

- Cannot be injected into Singleton services because it has a Scoped lifetime

---

# 3. IOptionsMonitor<T>

## Definition

`IOptionsMonitor<T>` continuously watches configuration and automatically provides updated values while the application is running.

No restart required.

---

Example

```csharp
public class EmailService
{
    private readonly IOptionsMonitor<EmailSettings> _options;

    public EmailService(
        IOptionsMonitor<EmailSettings> options)
    {
        _options = options;
    }

    public void Send()
    {
        var settings = _options.CurrentValue;

        Console.WriteLine(settings.Host);
    }
}
```

---

How it works

```
Application Running

↓

Configuration Changes

↓

Monitor Detects Change

↓

CurrentValue Updated
```

---

Example

Initial value

```
smtp.gmail.com
```

Configuration changes

```
smtp.office365.com
```

Without restarting,

```
CurrentValue

↓

smtp.office365.com
```

---

# Change Notifications

`IOptionsMonitor<T>` also supports callbacks.

```csharp
public EmailService(
    IOptionsMonitor<EmailSettings> options)
{
    options.OnChange(settings =>
    {
        Console.WriteLine($"New Host: {settings.Host}");
    });
}
```

Whenever the configuration changes,

the callback executes automatically.

---

Use When

- Long-running background services
- Singleton services
- Runtime configuration updates
- Cache refresh
- Feature flags
- Background workers

---

Advantages

- Automatically updates values
- Supports change notifications
- Singleton-friendly

---

Disadvantages

- Slightly more complex than `IOptions<T>`

---

# Internal Working

## IOptions

```
Application Starts

↓

Read Config Once

↓

Store

↓

Always Same Value
```

---

## IOptionsSnapshot

```
Request Starts

↓

Read Config

↓

Create Snapshot

↓

Use Snapshot

↓

Request Ends
```

---

## IOptionsMonitor

```
Application Running

↓

Watch Configuration

↓

Detect Change

↓

Update CurrentValue
```

---

# Lifetime Comparison

| Interface | Lifetime | Reads Configuration | Updates Automatically |
|------------|----------|---------------------|-----------------------|
| `IOptions<T>` | Singleton-friendly | Once | No |
| `IOptionsSnapshot<T>` | Scoped | Once per HTTP request | Yes, on the next request |
| `IOptionsMonitor<T>` | Singleton-friendly | Continuously | Yes, immediately after change detection |

---

# Feature Comparison

| Feature | IOptions | IOptionsSnapshot | IOptionsMonitor |
|----------|----------|------------------|-----------------|
| Singleton Compatible | ✅ Yes | ❌ No | ✅ Yes |
| Scoped | ❌ No | ✅ Yes | ❌ No |
| Reads Updated Values | ❌ No | ✅ Next request | ✅ Immediately after change detection |
| Supports OnChange | ❌ No | ❌ No | ✅ Yes |
| Best For | Stable configuration | Web requests | Runtime-changing configuration |

---

# Real-World Example

Suppose an e-commerce application has:

## Email Settings

Rarely change.

Use

```
IOptions
```

---

## Discount Percentage

Admin updates discounts during the day.

Use

```
IOptionsSnapshot
```

Each new request gets the latest value.

---

## Feature Flags

Operations team enables or disables features while the application is running.

Use

```
IOptionsMonitor
```

No application restart required.

---

# Common Mistakes

## Using IOptionsSnapshot in Singleton

Bad

```csharp
public class CacheService
{
    public CacheService(
        IOptionsSnapshot<AppSettings> options)
    {
    }
}
```

`IOptionsSnapshot` is Scoped.

Singleton services cannot depend on Scoped services.

---

## Using IOptions for Frequently Changing Settings

If configuration changes regularly,

`IOptions` won't automatically reflect the updates.

Use `IOptionsMonitor` or `IOptionsSnapshot` instead.

---

## Using IOptionsMonitor Everywhere

If configuration never changes,

`IOptions` is simpler and sufficient.

---

# Best Practices

- Use **`IOptions<T>`** for stable configuration values.
- Use **`IOptionsSnapshot<T>`** for request-based configuration updates in ASP.NET Core web applications.
- Use **`IOptionsMonitor<T>`** for Singleton services or when configuration must update while the application is running.
- Group related settings into strongly typed options classes.
- Validate options during startup using `.Validate()` or `.ValidateOnStart()`.

---

# Easy Way to Remember

Imagine **reading news**.

### IOptions

```
Printed Newspaper

↓

Never Changes
```

---

### IOptionsSnapshot

```
Refresh Website

↓

New News
```

---

### IOptionsMonitor

```
Live News TV

↓

Updates Automatically
```

---

# Frequently Asked Interview Questions

### What is the difference between `IOptions` and `IOptionsSnapshot`?

`IOptions<T>` reads configuration once and is suitable for stable settings. `IOptionsSnapshot<T>` creates a new snapshot for every HTTP request, so new requests see updated configuration.

---

### What is `IOptionsMonitor` used for?

`IOptionsMonitor<T>` is used when configuration can change while the application is running. It provides updated values through `CurrentValue` and supports change notifications.

---

### Which options interface can be injected into a Singleton service?

- ✅ `IOptions<T>`
- ✅ `IOptionsMonitor<T>`
- ❌ `IOptionsSnapshot<T>`

---

### Which interface supports the `OnChange()` callback?

Only **`IOptionsMonitor<T>`**.

---

# Memory Trick

| Think Of | Interface |
|-----------|-----------|
| 📄 Printed Newspaper | `IOptions<T>` |
| 🌐 Refreshing a Web Page | `IOptionsSnapshot<T>` |
| 📺 Live News Channel | `IOptionsMonitor<T>` |

---

# Interview Answer (2-Minute Version)

**What is the difference between `IOptions`, `IOptionsSnapshot`, and `IOptionsMonitor` in ASP.NET Core?**

All three are used to access strongly typed configuration, but they differ in when configuration values are refreshed. **`IOptions<T>`** reads configuration once and is ideal for stable settings such as connection strings or API keys. It is Singleton-friendly but does not automatically reflect configuration changes. **`IOptionsSnapshot<T>`** is Scoped and creates a fresh snapshot for every HTTP request, allowing new requests to see updated configuration values. **`IOptionsMonitor<T>`** continuously monitors configuration changes, exposes the latest values through `CurrentValue`, supports `OnChange()` callbacks, and can be safely used in Singleton services and long-running background processes. The choice depends on how frequently configuration changes and where the service is used.