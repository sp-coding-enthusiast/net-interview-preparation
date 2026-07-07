# 43. Singleton Pitfalls in ASP.NET Core Dependency Injection (DI)

Singleton is the **most misunderstood** service lifetime in ASP.NET Core.

Many developers think:

> **"Singleton is the fastest, so I'll use it everywhere."**

This is **incorrect**.

Using Singleton incorrectly can lead to:

- Data corruption
- Race conditions
- Thread safety issues
- Memory leaks
- Stale data
- User data leakage
- Application crashes

> **Interview Tip:** Senior and Lead-level interviews often focus on Singleton pitfalls because they reveal your understanding of concurrency and object lifetimes.

---

# What is a Singleton?

A Singleton service is created **only once** during the application's lifetime.

```csharp
builder.Services.AddSingleton<ICacheService, CacheService>();
```

Flow:

```
Application Starts
        │
        ▼
Create One Object
        │
        ▼
Reuse Same Object
        │
        ▼
Application Stops
```

Every request shares the **same instance**.

---

# Layman Example: Hotel Reception Desk

Imagine a hotel.

There is only **one reception desk**.

```
Guest 1

↓

Reception
```

```
Guest 2

↓

Same Reception
```

```
Guest 3

↓

Same Reception
```

The reception desk is shared.

If the receptionist writes Guest 1's room number on a whiteboard and forgets to erase it,

Guest 2 may see the wrong information.

This is similar to storing request-specific state in a Singleton.

---

# Pitfall 1: Shared Mutable State

The biggest mistake.

Bad Example

```csharp
public class UserService
{
    public string CurrentUser { get; set; }
}
```

Registration

```csharp
builder.Services.AddSingleton<UserService>();
```

---

Request 1

```
CurrentUser = Alice
```

---

Request 2

```
CurrentUser = Bob
```

Both requests share the same object.

Result

```
Alice

↓

Overwritten

↓

Bob
```

Users can interfere with each other's data.

---

# Correct Approach

Store request-specific data in:

- Local variables
- Scoped services
- `HttpContext`

Not in Singleton fields.

---

# Pitfall 2: Thread Safety Issues

Singletons are accessed by multiple requests simultaneously.

Imagine:

```
100 Users

↓

Same Singleton

↓

Same Variable
```

Two threads update the same value at the same time.

Example

```csharp
public class CounterService
{
    public int Count;
}
```

Multiple requests

```
Count++

Count++

Count++
```

Expected

```
3
```

Actual

```
1

or

2

or

3
```

depending on timing.

This is called a **race condition**.

---

# Solution

Use thread-safe techniques.

Example

```csharp
Interlocked.Increment(ref Count);
```

or appropriate synchronization mechanisms such as `lock` when necessary.

---

# Pitfall 3: Injecting Scoped Services into Singleton

Bad

```csharp
builder.Services.AddSingleton<OrderService>();

builder.Services.AddScoped<ProductRepository>();
```

Constructor

```csharp
public OrderService(ProductRepository repository)
{
}
```

This is invalid.

Why?

```
Singleton

↓

Lives Forever
```

```
Scoped

↓

Lives Per Request
```

A long-lived service cannot safely hold a reference to a shorter-lived service.

ASP.NET Core typically throws:

```
InvalidOperationException
```

---

# Correct Solution

Instead of injecting a Scoped service directly:

- Change the lifetime if appropriate.
- Or resolve the Scoped service within a created scope using `IServiceScopeFactory`.

---

# Pitfall 4: Memory Leaks

Singletons stay alive until the application stops.

Bad

```csharp
public List<string> Logs = new();
```

Every request

```
Logs.Add(...)
```

After millions of requests

```
Huge Memory Usage
```

The list never gets released.

---

# Better

Use bounded caches or persistent storage when appropriate.

Avoid keeping unbounded collections in Singleton services.

---

# Pitfall 5: Stale Data

Bad Example

```csharp
public class ProductService
{
    public List<Product> Products;
}
```

Application starts.

Products are loaded once.

Hours later,

the database changes.

Singleton still contains old data.

---

Solution

Refresh cached data appropriately or query the data source when freshness is required.

---

# Pitfall 6: User-Specific Data

Bad

```csharp
public class LoginService
{
    public string Username;
}
```

Requests

```
User A

↓

Username = Alice
```

```
User B

↓

Username = Bob
```

User A now sees Bob's data.

This is a serious security issue.

---

Correct

Store user-specific data in:

- Scoped services
- `HttpContext.User`
- Claims
- Session (when appropriate)

---

# Pitfall 7: Not Being Thread-Safe

Bad

```csharp
Dictionary<int, string>
```

Multiple requests update it simultaneously.

Possible issues:

- Exceptions
- Corrupted state
- Lost updates

---

Better

Use thread-safe collections when concurrent access is required.

Example

```csharp
ConcurrentDictionary<int, string>
```

---

# Pitfall 8: Holding Expensive Resources Forever

Bad

```
Singleton

↓

100 MB Image

↓

Never Released
```

The resource stays in memory for the application's lifetime.

Use Singleton only when keeping the resource alive is intentional and beneficial.

---

# Pitfall 9: Using Singleton for Entity Framework DbContext

Bad

```csharp
builder.Services.AddSingleton<AppDbContext>();
```

Problems

- Not thread-safe
- Tracks entities across requests
- Causes stale tracking information
- Memory growth
- Concurrency issues

---

Correct

```csharp
builder.Services.AddDbContext<AppDbContext>();
```

`AddDbContext()` registers `DbContext` as **Scoped** by default.

---

# Pitfall 10: Assuming Singleton Improves Everything

Many developers think

```
Singleton

↓

Faster

↓

Use Everywhere
```

Wrong.

Choosing the wrong lifetime can introduce correctness and security problems that outweigh any performance gain.

Always choose the lifetime based on the service's behavior.

---

# Real-World Example

Suppose an e-commerce application has

```
ShoppingCartService

↓

Singleton
```

User A

```
Adds Laptop
```

User B

```
Adds Phone
```

Now both users may see the same shared cart.

Correct

```
ShoppingCartService

↓

Scoped
```

Each request (or user session, depending on the design) has isolated state.

---

# When Singleton Is a Good Choice

Singleton is appropriate for services that are:

- Stateless
- Thread-safe
- Shared across the application

Examples:

- Configuration service
- Memory cache
- Logging provider
- Feature flag service
- Some HTTP client factories (the factory itself)

---

# When Singleton Is a Bad Choice

Avoid Singleton for:

- Entity Framework `DbContext`
- Repository classes using `DbContext`
- User-specific state
- Shopping carts
- Request-specific data
- Services with mutable shared state unless carefully synchronized

---

# Common Interview Mistakes

### "Singleton is always faster."

False.

Performance is only one factor.

Correctness and thread safety are more important.

---

### "Singleton can store current user."

False.

Current user information is request-specific.

---

### "DbContext should be Singleton."

False.

`DbContext` is designed to be Scoped.

---

### "Singleton can inject Scoped."

False.

A Singleton should not directly depend on a Scoped service.

---

# Best Practices

- Keep Singleton services stateless whenever possible.
- Make Singleton services thread-safe.
- Never store request-specific or user-specific data in a Singleton.
- Avoid direct dependencies from Singleton to Scoped services.
- Use immutable data where practical.
- Choose Singleton only when shared state is intentional and safe.

---

# Good vs Bad Examples

| Good Singleton | Bad Singleton |
|----------------|---------------|
| Configuration Service | `DbContext` |
| Logging Provider | Shopping Cart |
| Memory Cache | Current User |
| Feature Flags | Repository using `DbContext` |
| Stateless Utility | Mutable request state |

---

# Easy Way to Remember

Think of a **Whiteboard in an Office**.

One whiteboard.

Everyone uses it.

If one employee writes:

```
Meeting at 10 AM
```

and another writes:

```
Lunch at 1 PM
```

Everyone sees the same board.

That's exactly how a Singleton behaves.

Never write **private information** on a shared whiteboard.

---

# Frequently Asked Interview Questions

### What is the biggest risk of Singleton?

Shared mutable state leading to concurrency issues and data leakage.

---

### Can Singleton store user information?

No.

User-specific data should be stored in request-scoped components such as `HttpContext` or Scoped services.

---

### Is Singleton thread-safe by default?

No.

If multiple threads access mutable state, you must implement thread safety yourself.

---

### Can Singleton inject Scoped services?

No.

A Singleton should not directly depend on a Scoped service because their lifetimes are incompatible.

---

### Why is `DbContext` not Singleton?

Because `DbContext` is not thread-safe and is designed to represent a single unit of work per request.

---

# Interview Answer (2-Minute Version)

**What are the common pitfalls of using Singleton services in ASP.NET Core?**

Singleton services are created once and shared across all requests for the lifetime of the application. The most common pitfalls are storing mutable shared state, which can lead to race conditions and user data leakage, and failing to make the service thread-safe. Another common mistake is injecting Scoped services, such as Entity Framework `DbContext`, into a Singleton, which results in lifetime mismatches and runtime exceptions. Singleton services can also cause stale data and memory growth if they cache information indefinitely. Singleton should therefore be reserved for shared, thread-safe, typically stateless services such as configuration, caching, and logging, while request-specific data should use Scoped services.