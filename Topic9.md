# ASP.NET Core Middleware Pipeline — Order of Execution

The **order of middleware execution is one of the most important concepts in ASP.NET Core**.  
A wrong order can break authentication, security, routing, or even make endpoints unreachable.

---

# 1. Core Idea

Middleware runs in two phases:

## Request Phase (INBOUND)

```text
Client Request
   ↓
Middleware 1
   ↓
Middleware 2
   ↓
Middleware 3
   ↓
Controller
```

## Response Phase (OUTBOUND)

```text
Controller
   ↑
Middleware 3
   ↑
Middleware 2
   ↑
Middleware 1
   ↑
Client Response
```

This is called the **onion model**.

---

# 2. Standard Execution Order in ASP.NET Core

A typical correct pipeline looks like this:

```csharp
app.UseExceptionHandler();

app.UseHttpsRedirection();

app.UseStaticFiles();

app.UseRouting();

app.UseCors();

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

---

# 3. Step-by-Step Execution Flow

## Step 1: Exception Handling (Outer Layer)

```csharp
app.UseExceptionHandler();
```

### Purpose:
- Catches unhandled exceptions globally

### Why first?
Because it must catch errors from everything inside.

---

## Step 2: HTTPS Redirection

```csharp
app.UseHttpsRedirection();
```

### Purpose:
- Redirect HTTP → HTTPS

### Why early?
To ensure all requests are secure before processing.

---

## Step 3: Static Files

```csharp
app.UseStaticFiles();
```

### Purpose:
- Serves files like:
  - images
  - CSS
  - JS

### Why here?
So static files are served early and don’t hit controllers.

---

## Step 4: Routing

```csharp
app.UseRouting();
```

### Purpose:
- Determines which endpoint will handle the request

### Important:
This **only selects route**, it does NOT execute controller yet.

---

## Step 5: CORS (Cross-Origin Requests)

```csharp
app.UseCors();
```

### Purpose:
- Allows or blocks cross-origin requests

### Why here?
After routing is known but before authentication.

---

## Step 6: Authentication

```csharp
app.UseAuthentication();
```

### Purpose:
- Identifies the user (who are you?)

Examples:
- JWT validation
- Cookies
- OAuth tokens

---

## Step 7: Authorization

```csharp
app.UseAuthorization();
```

### Purpose:
- Checks permissions (are you allowed?)

Example:

```text
Admin role required
```

---

## Step 8: Endpoint Execution

```csharp
app.MapControllers();
```

### Purpose:
- Executes controller action

Example:

```csharp
[HttpGet("/orders")]
public IActionResult GetOrders()
```

---

# 4. Full Request Flow (Real Execution)

## Request comes in:

```text
GET /api/orders
```

## Execution sequence:

```text
1. Exception Handler Middleware
2. HTTPS Redirection Middleware
3. Static Files Middleware
4. Routing Middleware
5. CORS Middleware
6. Authentication Middleware
7. Authorization Middleware
8. Controller Execution
```

---

## Response flows back in reverse:

```text
Controller
   ↑
Authorization
   ↑
Authentication
   ↑
CORS
   ↑
Routing
   ↑
Static Files
   ↑
HTTPS
   ↑
Exception Handler
   ↑
Client
```

---

# 5. Why Order Matters (Very Important)

---

## ❌ Wrong Order Example

```csharp
app.UseAuthorization();
app.UseAuthentication();
```

### Problem:
- Authorization runs before user identity is established

### Result:
```text
User is always unauthorized
```

---

## ❌ Wrong Routing Order

```csharp
app.MapControllers();
app.UseRouting();
```

### Problem:
- No route matching happens

### Result:
```text
404 Not Found
```

---

## ❌ CORS after Authorization

```csharp
app.UseAuthorization();
app.UseCors();
```

### Problem:
- Browser blocks request before it reaches auth

---

# 6. Short-Circuiting Middleware

Some middleware can stop execution early:

```csharp
app.Use(async (context, next) =>
{
    await context.Response.WriteAsync("Blocked");
    return; // stops pipeline
});
```

### Result:
- Controller will NOT execute
- Remaining middleware skipped

---

# 7. Visual Model (Interview-Friendly)

```text
REQUEST FLOW
------------------------------------------------
Exception Handler
        ↓
HTTPS Redirection
        ↓
Static Files
        ↓
Routing
        ↓
CORS
        ↓
Authentication
        ↓
Authorization
        ↓
Controllers
------------------------------------------------
RESPONSE FLOW (Reverse)
------------------------------------------------
Controllers
        ↑
Authorization
        ↑
Authentication
        ↑
CORS
        ↑
Routing
        ↑
Static Files
        ↑
HTTPS
        ↑
Exception Handler
------------------------------------------------
```

---

# 8. Key Interview Points

## 1. Order is critical
Middleware order directly affects behavior.

---

## 2. Authentication before Authorization
Because:
- Authentication identifies user
- Authorization checks permissions

---

## 3. Routing must come before endpoint execution
Because:
- It decides which controller runs

---

## 4. Exception handler must be first
Because:
- It must catch all exceptions in pipeline

---

# 9. Common Interview Questions

## Q1: What happens if UseRouting is missing?

```text
No endpoint selection → 404 errors
```

---

## Q2: What happens if UseAuthentication is after UseAuthorization?

```text
User is not authenticated → authorization fails
```

---

## Q3: Can middleware change order dynamically?

```text
No, order is fixed at startup
```

---

# 10. Senior-Level Answer

The ASP.NET Core middleware pipeline executes in a strictly defined order where each middleware processes the request in sequence and the response flows back in reverse order. Correct ordering is critical: exception handling must be first to catch errors globally, routing must precede endpoint execution, authentication must occur before authorization, and endpoint mapping must be last. Incorrect ordering can result in broken authentication, failed routing, or blocked requests.

---

# 11. Easy Memory Trick

Think:

```text
Security → Setup → Routing → Identity → Permission → Action
```

Or simply:

```text
Exception → HTTPS → Static → Routing → Auth → AuthZ → Controller
```