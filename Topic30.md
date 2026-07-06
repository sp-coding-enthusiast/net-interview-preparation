# 9. Static Files Middleware in ASP.NET Core

Static Files Middleware is responsible for serving files that **do not require server-side processing**.

These files are sent directly to the browser without executing any controller or business logic.

Examples include:

- HTML
- CSS
- JavaScript
- Images
- Fonts
- Videos
- PDF files

---

# What is Static Files Middleware?

## Definition

Static Files Middleware serves files directly from the application's **wwwroot** folder (or another configured location) to the client.

Instead of passing the request through controllers, ASP.NET Core reads the file and returns it immediately.

---

# Layman Example: Library

Imagine you visit a library.

You ask for a book.

The librarian simply hands you the book.

There is:

- No approval
- No processing
- No calculations

```
Customer

↓

Ask for Book

↓

Librarian

↓

Book Delivered
```

The librarian behaves like **Static Files Middleware**.

---

# Another Example: Restaurant

Suppose you order:

### Water

The waiter immediately serves it.

```
Customer

↓

Water

↓

Served Immediately
```

No cooking required.

Now order:

### Pizza

```
Customer

↓

Kitchen

↓

Chef

↓

Cooking

↓

Serve Pizza
```

Static files are like **water**.

Controllers are like **pizza**.

---

# Why Do We Need Static Files Middleware?

Every website contains static assets.

Example:

```
Home Page

↓

logo.png

↓

style.css

↓

app.js

↓

favicon.ico
```

Without Static Files Middleware, none of these files would be accessible.

---

# Example Folder Structure

```
MyWebApp

│

├── Controllers

├── Models

├── Views

└── wwwroot
      │
      ├── css
      │      style.css
      │
      ├── js
      │      app.js
      │
      ├── images
      │      logo.png
      │
      └── fonts
```

Everything inside **wwwroot** can be served directly.

---

# Enable Static Files Middleware

```csharp
app.UseStaticFiles();
```

That's all.

ASP.NET Core now serves files from

```
wwwroot
```

---

# Example

Suppose

```
wwwroot

↓

images

↓

logo.png
```

Browser requests

```
https://localhost:5001/images/logo.png
```

Pipeline

```
Browser

↓

Static Files Middleware

↓

Find File

↓

Return Image

↓

Browser
```

No controller executes.

---

# What Happens Internally?

```
Browser

↓

GET /images/logo.png

↓

Static Files Middleware

↓

Does File Exist?

↓

Yes

↓

Read File

↓

Return File

↓

Browser
```

---

# If File Does Not Exist

```
GET /images/car.png

↓

Static Middleware

↓

File Found?

↓

No

↓

Next Middleware

↓

404 Not Found
```

The request continues through the pipeline if the file isn't found.

---

# Middleware Pipeline

```
Browser
    │
    ▼
Exception Middleware
    │
    ▼
Static Files Middleware
    │
    ▼
Routing
    │
    ▼
Authentication
    │
    ▼
Authorization
    │
    ▼
Controller
```

Notice

Static files are checked **before** routing.

---

# Why Before Routing?

Suppose the browser requests

```
/css/site.css
```

If Static Files Middleware comes first

```
Browser

↓

Static Files

↓

Return CSS

↓

Done
```

Very fast.

---

If Routing comes first

```
Browser

↓

Routing

↓

Controller Search

↓

No Controller

↓

Then Static Files

↓

CSS Returned
```

Extra work.

Therefore Static Files Middleware is usually registered before routing.

---

# Example

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.UseStaticFiles();

app.MapControllers();

app.Run();
```

---

# Accessing Static Files

Suppose

```
wwwroot

↓

css

↓

style.css
```

Browser

```
https://localhost:5001/css/style.css
```

Loads

```
style.css
```

---

Suppose

```
wwwroot

↓

images

↓

car.jpg
```

Browser

```
https://localhost:5001/images/car.jpg
```

Returns

```
car.jpg
```

---

# Serving HTML

Suppose

```
wwwroot

↓

index.html
```

Browser

```
https://localhost:5001/index.html
```

Returns the HTML file directly.

---

# Real-World Example

A typical website may contain:

```
Home Page

↓

HTML

↓

CSS

↓

JavaScript

↓

Images

↓

Fonts
```

Every one of these is served through Static Files Middleware.

---

# Without Static Files Middleware

Suppose you have

```
wwwroot

↓

logo.png
```

Browser requests

```
/logo.png
```

Pipeline

```
Browser

↓

Routing

↓

Controller

↓

404
```

Image is not found because Static Files Middleware is not enabled.

---

# With Static Files Middleware

```
Browser

↓

Static Middleware

↓

logo.png

↓

Return Image

↓

Browser
```

Works correctly.

---

# Static File Types

Common examples:

| File Type | Example |
|-----------|----------|
| HTML | index.html |
| CSS | site.css |
| JavaScript | app.js |
| Image | logo.png |
| Font | roboto.ttf |
| PDF | guide.pdf |
| Video | intro.mp4 |
| Audio | music.mp3 |

---

# Can Static Files Be Protected?

By default,

```
wwwroot
```

is **publicly accessible**.

Anyone who knows the URL can access those files.

Example

```
https://localhost/images/logo.png
```

returns the image without authentication.

---

# If You Need Secure Files

Store them **outside** `wwwroot`.

Then return them through a controller after authorization.

Example

```
PrivateFiles

↓

salary.pdf
```

Controller

```
Authentication

↓

Authorization

↓

Return PDF
```

Only authorized users can download it.

---

# Best Practices

- Keep only public assets in `wwwroot`.
- Store confidential documents outside `wwwroot`.
- Register `UseStaticFiles()` before routing.
- Enable browser caching for static assets in production.
- Use versioning (cache busting) for CSS and JavaScript files to ensure clients receive updated content after deployments.

---

# Frequently Asked Interview Questions

### What does `UseStaticFiles()` do?

It enables ASP.NET Core to serve static files directly from the `wwwroot` folder without involving controllers or Razor Pages.

---

### Why is Static Files Middleware faster?

Because it bypasses MVC, model binding, filters, and controller execution, reading the file directly from disk and returning it.

---

### Can Static Files Middleware serve files outside `wwwroot`?

Yes, by configuring an additional file provider, though `wwwroot` is the default public web root.

---

### Should sensitive files be placed in `wwwroot`?

No.

Files in `wwwroot` are intended to be publicly accessible. Sensitive files should be stored outside the web root and served only after authentication and authorization checks.

---

# Complete Request Flow

```
Browser
    │
    ▼
Exception Middleware
    │
    ▼
Static Files Middleware
    │
    ├── File Exists?
    │        │
    │        ├── Yes → Return File
    │        │
    │        └── No
    ▼
Routing
    │
    ▼
Authentication
    │
    ▼
Authorization
    │
    ▼
Controller
    │
    ▼
Response
```

---

# Interview Answer (2-Minute Version)

**What is Static Files Middleware in ASP.NET Core?**

Static Files Middleware is responsible for serving static assets such as HTML, CSS, JavaScript, images, fonts, and PDF files directly from the application's `wwwroot` folder. When a request matches a static file, the middleware returns the file immediately without invoking routing or controllers, making it highly efficient. It is typically registered early in the middleware pipeline using `app.UseStaticFiles()`. Public assets should be stored in `wwwroot`, while sensitive files should be stored outside the web root and served through authenticated and authorized endpoints.