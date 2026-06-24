# HttpClient + Dependency Injection (DI) in ASP.NET Core Microservices

## Why This Topic Is Important

In modern microservices architecture, services communicate with each other through:

- HTTP APIs
- gRPC APIs

For example:

```text
Order Service
      |
      v
Payment Service
```

Whenever Order Service needs to charge a customer, it calls Payment Service through HTTP.

To make HTTP calls in .NET, we use:

```csharp
HttpClient
```

A common interview question is:

> Why should we use HttpClientFactory (DI) instead of creating HttpClient manually?

---

# Bad Approach

Many developers write:

```csharp
public async Task ProcessPayment()
{
    var client = new HttpClient();

    await client.GetAsync("https://payment-api/pay");
}
```

Looks harmless.

But this can create serious production issues.

---

# Real-Life Analogy

Imagine every time you need to call someone:

1. Buy a new mobile phone
2. Make one call
3. Throw the phone away

Then repeat.

Sounds wasteful.

Creating a new HttpClient for every request is very similar.

---

# What Happens Internally?

When HttpClient is created:

```text
HttpClient
      |
      v
HttpMessageHandler
      |
      v
TCP Socket
      |
      v
Network Connection
```

Each connection consumes operating system resources.

When thousands of requests arrive:

```text
Request 1 -> New HttpClient
Request 2 -> New HttpClient
Request 3 -> New HttpClient
Request 4 -> New HttpClient
...
```

Thousands of connections get created.

---

# The Socket Exhaustion Problem

After a request completes, sockets are not immediately released.

They remain in:

```text
TIME_WAIT
```

state.

Example:

```text
1000 Requests/Second
```

and every request does:

```csharp
new HttpClient()
```

Result:

```text
1000 New Sockets/Second
```

After some time:

```text
No free sockets available
```

Application starts throwing:

```text
SocketException
Connection Refused
No Buffer Space Available
```

This production issue is called:

# Socket Exhaustion

---

# How Dependency Injection Solves This

Instead of creating HttpClient manually:

```csharp
new HttpClient()
```

Register it once:

```csharp
builder.Services.AddHttpClient();
```

Now ASP.NET Core provides:

```text
IHttpClientFactory
```

Internally:

```text
IHttpClientFactory
        |
        v
Connection Pool
        |
        v
Reuse Existing Connections
```

Connections are reused instead of continuously recreated.

---

# Registration

```csharp
builder.Services.AddHttpClient();
```

---

# Using Constructor Injection

```csharp
public class PaymentService
{
    private readonly HttpClient _httpClient;

    public PaymentService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task MakePayment()
    {
        await _httpClient.GetAsync("/pay");
    }
}
```

---

# Typed Client (Recommended)

Most companies use Typed Clients.

Registration:

```csharp
builder.Services.AddHttpClient<
    IPaymentClient,
    PaymentClient>();
```

Implementation:

```csharp
public class PaymentClient : IPaymentClient
{
    private readonly HttpClient _httpClient;

    public PaymentClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<string> Pay()
    {
        return await _httpClient.GetStringAsync("/pay");
    }
}
```

Usage:

```csharp
public class OrderService
{
    private readonly IPaymentClient _paymentClient;

    public OrderService(IPaymentClient paymentClient)
    {
        _paymentClient = paymentClient;
    }
}
```

---

# Internal Flow

```text
Application
      |
      v
DI Container
      |
      v
IHttpClientFactory
      |
      v
Connection Pool
      |
      v
HTTP Call
```

---

# Benefit 1: Connection Reuse

Without Factory:

```text
Request 1 -> New Connection
Request 2 -> New Connection
Request 3 -> New Connection
Request 4 -> New Connection
```

With Factory:

```text
Request 1 -> Reuse Connection
Request 2 -> Reuse Connection
Request 3 -> Reuse Connection
Request 4 -> Reuse Connection
```

Less memory usage.

Better performance.

Fewer network resources consumed.

---

# Benefit 2: DNS Refresh

Suppose:

```text
payment.company.com
```

Initially resolves to:

```text
10.1.1.10
```

Kubernetes restarts pod.

New IP:

```text
10.1.1.20
```

If DNS is cached forever:

```text
Application still calls old IP
```

Result:

```text
Request failures
```

IHttpClientFactory periodically refreshes handlers and DNS information.

This is extremely important in:

- Kubernetes
- Docker
- Azure AKS
- AWS EKS
- Cloud-native systems

---

# Benefit 3: Centralized Configuration

Instead of configuring everywhere:

```csharp
new HttpClient()
```

Configure once:

```csharp
builder.Services.AddHttpClient<IPaymentClient, PaymentClient>(client =>
{
    client.BaseAddress =
        new Uri("https://payment-api");

    client.Timeout =
        TimeSpan.FromSeconds(30);
});
```

Now every request automatically uses:

- Base URL
- Timeout
- Headers
- Authentication

---

# Benefit 4: Easier Unit Testing

Without DI:

```csharp
var client = new HttpClient();
```

Hard to mock.

Hard to test.

With DI:

```csharp
IPaymentClient
```

can easily be mocked.

Example:

```csharp
Mock<IPaymentClient>
```

This is one reason why DI is heavily used.

---

# Benefit 5: Retry Policy

Microservices fail temporarily all the time.

Examples:

- Network glitch
- Temporary timeout
- Service restart
- Pod restart
- Load balancer issue

Without retry:

```text
Call Failed
      |
      v
User Gets Error
```

With retry:

```text
Call Failed
      |
Retry
      |
Retry
      |
Success
```

User never notices.

---

# Retry Example

```csharp
builder.Services
       .AddHttpClient<IPaymentClient, PaymentClient>()
       .AddStandardResilienceHandler();
```

The framework automatically performs retries for transient failures.

---

# Circuit Breaker (Most Important Production Concept)

Senior and Lead interviews often ask:

> How does Circuit Breaker work with HttpClientFactory?

---

# Problem Scenario

Payment Service goes down.

```text
Order Service
      |
      v
Payment Service (DOWN)
```

Without Circuit Breaker:

```text
Request 1 -> Fail
Request 2 -> Fail
Request 3 -> Fail
Request 4 -> Fail
Request 5 -> Fail
...
```

Thousands of requests continue hitting the dead service.

Result:

- CPU waste
- Thread waste
- Network waste
- Cascading failures

Entire system slows down.

---

# Real-Life Analogy

Think about an electrical circuit breaker.

```text
Too much current
      |
      v
Circuit trips
      |
      v
Electricity stops
```

This protects your house.

Software Circuit Breaker does exactly the same thing.

---

# Circuit Breaker States

There are three states:

## 1. Closed State

Everything is healthy.

```text
Order Service
      |
      v
Payment Service
```

Requests flow normally.

```text
Success
Success
Success
Success
```

---

## 2. Open State

Too many failures detected.

```text
Fail
Fail
Fail
Fail
Fail
```

Circuit breaker opens.

```text
Order Service
      X
      X
      X
Payment Service
```

No requests are sent.

Requests fail immediately.

This protects resources.

---

## 3. Half-Open State

After waiting for some time:

```text
30 Seconds Later
```

Circuit breaker allows a few test requests.

```text
Test Request
```

If successful:

```text
Circuit Closed Again
```

Normal traffic resumes.

If failed:

```text
Circuit Opens Again
```

Protection continues.

---

# State Transition Diagram

```text
          Failures
Closed -----------------> Open
  ^                        |
  |                        |
  |                        |
  | Success                | Wait Period
  |                        |
  +------ Half Open <------+
```

---

# Circuit Breaker Flow

Normal:

```text
Request
   |
   v
Payment Service
   |
 Success
```

Service Down:

```text
Request
   |
   v
Payment Service
   |
 Failure
```

Many failures:

```text
Failure
Failure
Failure
Failure
Failure
```

Circuit opens:

```text
Request
   |
   X
   |
Immediate Failure
```

No network call is made.

---

# Circuit Breaker with DI

Registration:

```csharp
builder.Services
    .AddHttpClient<IPaymentClient, PaymentClient>()
    .AddStandardResilienceHandler(options =>
    {
    });
```

Internally:

```text
DI Container
      |
      v
HttpClientFactory
      |
      v
Resilience Pipeline
      |
      +---- Retry
      |
      +---- Timeout
      |
      +---- Circuit Breaker
      |
      +---- Rate Limiter
      |
      +---- Fallback
```

Every HTTP request automatically passes through this pipeline.

---

# Production Scenario

Imagine:

```text
Order Service
```

receives:

```text
10,000 Requests/Minute
```

Payment Service crashes.

Without Circuit Breaker:

```text
10,000 failing requests
```

every minute.

System becomes overloaded.

With Circuit Breaker:

```text
First Few Requests -> Fail

Circuit Opens

Remaining Requests
      |
      v
Fail Immediately
```

Application remains responsive.

---

# Why Retry and Circuit Breaker Are Used Together

Retry handles:

```text
Temporary Failures
```

Examples:

- Network blip
- Short timeout
- Pod restart

Circuit Breaker handles:

```text
Long-Term Failures
```

Examples:

- Service completely down
- Database unavailable
- Dependency outage

Together:

```text
Request
   |
Retry
   |
Retry
   |
Still Failing?
   |
Circuit Breaker Opens
```

This is the standard microservices resilience pattern.

---

# Complete Production Architecture

```text
Order Service
      |
      v
Typed Client
(IPaymentClient)
      |
      v
HttpClientFactory
      |
      v
Connection Pool
      |
      v
Retry Policy
      |
      v
Timeout Policy
      |
      v
Circuit Breaker
      |
      v
Payment Service
```

---

# Lead-Level Interview Answer

> In ASP.NET Core microservices, HttpClient should be created through IHttpClientFactory using Dependency Injection rather than using new HttpClient(). IHttpClientFactory manages connection pooling, prevents socket exhaustion, refreshes DNS entries, centralizes configuration, and improves testability. It also enables resilience patterns such as retries, timeouts, and circuit breakers. A circuit breaker monitors failures and, after a threshold is reached, opens the circuit to stop sending requests to an unhealthy service. After a cooldown period it enters half-open state, sends test requests, and closes again if the dependency becomes healthy. This prevents cascading failures and improves overall system reliability and scalability.