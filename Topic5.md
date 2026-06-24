# Dependency Injection (DI) in Microservices

DI in microservices is the same core concept as monoliths, but the **scope, boundaries, and design discipline become more important** because each service is independent.

---

# 1. First Principle: What changes in microservices?

In a monolith:

```text
One application → many modules → shared DI container
```

In microservices:

```text
Each microservice = independent application + its own DI container
```

Example:

```text
Order Service → its own DI container
Payment Service → its own DI container
User Service → its own DI container
```

There is **no shared DI across services**.

---

# 2. DI inside a microservice

Inside each microservice, DI works exactly like ASP.NET Core:

```csharp
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddSingleton<ICacheService, CacheService>();
```

Each service:

- builds its own dependency graph
- manages its own object lifetimes
- has its own configuration and registrations

---

# 3. Real Microservices Example

## Order Microservice

```text
Controller
   ↓
OrderService (Scoped)
   ↓
OrderRepository (Scoped)
   ↓
DbContext (Scoped)
```

## Payment Microservice

```text
Controller
   ↓
PaymentService (Scoped)
   ↓
PaymentGatewayClient (Transient)
   ↓
HttpClient (Singleton or managed via IHttpClientFactory)
```

Each service is independent.

---

# 4. Key Role of DI in Microservices

DI is used to manage:

- internal service composition
- external integrations
- cross-cutting concerns (logging, caching, telemetry)

But NOT cross-service orchestration.

---

# 5. Important Pattern: HttpClient + DI

In microservices, communication happens via HTTP/gRPC.

Bad approach:

```csharp
new HttpClient()
```

Good approach:

```csharp
builder.Services.AddHttpClient<IPaymentClient, PaymentClient>();
```

Why DI matters here:

- avoids socket exhaustion
- enables reuse
- supports resilience policies (retry, circuit breaker)

---

# 6. Service Communication + DI Boundary

DI does NOT cross service boundaries.

Example:

```text
Order Service DI container
        |
        | HTTP call
        ↓
Payment Service DI container
```

Each side resolves its own dependencies independently.

---

# 7. Lifetimes in Microservices Context

## Transient

Used for stateless operations:

- formatters
- mappers
- request builders

---

## Scoped (MOST IMPORTANT)

Represents:

```text
One HTTP request = one scope
```

Used for:

- DbContext
- business transaction logic
- request context

---

## Singleton

Used for:

- caching
- configuration
- connection pools
- stateless shared services

⚠️ Must be thread-safe in microservices because concurrency is high.

---

# 8. Common Pitfalls in Microservices DI

---

## Pitfall 1: God Services (too many dependencies)

```csharp
public OrderService(
    IPaymentService payment,
    INotificationService notification,
    IShippingService shipping,
    IInventoryService inventory,
    ILogger logger,
    ICache cache)
```

Problem:
- tight coupling
- hard to test
- violates microservice autonomy

Fix:
- split responsibilities
- use domain services

---

## Pitfall 2: Distributed circular dependency

Bad design:

```text
Order Service → Payment Service → Order Service
```

This creates:

- network dependency cycles
- runtime coupling
- deployment blockage

Fix:

- event-driven architecture
- message queues (Kafka/RabbitMQ)

---

## Pitfall 3: Singleton misuse across async calls

Shared singleton state:

```csharp
public class RequestTracker
{
    public Dictionary<string, string> Data;
}
```

Problem:

- race conditions
- cross-request contamination

---

## Pitfall 4: Overusing synchronous HTTP calls

```text
Order → Payment → Inventory → Shipping
```

Problem:

- latency explosion
- cascading failures

Fix:

- async messaging
- background processing

---

## Pitfall 5: Wrong lifetime for HttpClient

Bad:

```csharp
new HttpClient()
```

Good:

```csharp
AddHttpClient()
```

or DI-managed singleton HttpClientFactory

---

# 9. DI + Resilience in Microservices

DI is often combined with:

- Retry policies
- Circuit breakers
- Timeouts
- Bulkheads

Example:

```csharp
builder.Services.AddHttpClient<PaymentClient>()
    .AddPolicyHandler(GetRetryPolicy())
    .AddPolicyHandler(GetCircuitBreakerPolicy());
```

---

# 10. Testing Advantage in Microservices

DI makes microservices highly testable:

```csharp
var service = new OrderService(
    new FakePaymentService(),
    new FakeRepository());
```

Benefits:

- no real network calls
- isolated unit tests
- fast CI pipelines

---

# 11. Architecture View

```
+-------------------+        +-------------------+
| Order Service     |        | Payment Service   |
| DI Container      |        | DI Container      |
| Scoped Services   |        | Scoped Services   |
+-------------------+        +-------------------+
        |                             |
        | HTTP / Messaging           |
        ↓                             ↓
External Communication Layer (not DI shared)
```

---

# 12. Key Interview Insight

DI in microservices is NOT about sharing dependencies across services.

It is about:

- **clean internal structure per service**
- **decoupled communication between services**
- **proper lifetime management inside each service**

---

# 13. Senior-Level Interview Answer

Dependency Injection in microservices is scoped to each individual service. Every microservice maintains its own DI container, lifetimes, and dependency graph. Transient, Scoped, and Singleton lifetimes are used within each service just like in a monolith, but there is no shared dependency injection across service boundaries.

The main challenges in microservices DI are managing HttpClient properly, avoiding distributed circular dependencies, ensuring thread safety in singleton services, and preventing tight coupling between services through direct service-to-service calls. Best practice is to use DI for internal composition and rely on async messaging or APIs for inter-service communication.