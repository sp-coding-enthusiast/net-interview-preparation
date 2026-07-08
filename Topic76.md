# 82. File Upload in ASP.NET Core

---

# What is File Upload?

File Upload is the process of sending one or more files from a client (browser, mobile app, Postman, etc.) to a server.

The server receives the file and can:

- Save it to disk
- Store it in a database
- Upload it to cloud storage (Azure Blob Storage, AWS S3)
- Process it (resize images, scan for viruses, extract data)

---

# Real-Life Analogy

Imagine applying for a job online.

You upload:

- Resume (PDF)
- Passport photo (JPG)
- Certificates (PDF)

The website stores these files.

That's exactly what a File Upload API does.

```
You

↓

Upload Resume

↓

Server

↓

Save Resume
```

---

# How File Upload Works

```
Client

↓

Select File

↓

HTTP POST Request

↓

Multipart/Form-Data

↓

ASP.NET Core

↓

Save File

↓

Response
```

---

# Why Use `multipart/form-data`?

Files are binary data.

JSON cannot efficiently send binary files.

Instead, browsers send files using:

```
Content-Type:

multipart/form-data
```

This format allows sending:

- Files
- Text fields
- Images
- Videos

In a single request.

---

# ASP.NET Core File Upload

ASP.NET Core uses the `IFormFile` interface.

Example

```csharp
[HttpPost]
public async Task<IActionResult> Upload(IFormFile file)
{
    if (file == null || file.Length == 0)
        return BadRequest("No file selected.");

    var path = Path.Combine("Uploads", file.FileName);

    using var stream = new FileStream(path, FileMode.Create);

    await file.CopyToAsync(stream);

    return Ok("File uploaded successfully.");
}
```

---

# Upload Multiple Files

```csharp
[HttpPost]
public async Task<IActionResult> Upload(List<IFormFile> files)
{
    foreach (var file in files)
    {
        var path = Path.Combine("Uploads", file.FileName);

        using var stream = new FileStream(path, FileMode.Create);

        await file.CopyToAsync(stream);
    }

    return Ok();
}
```

---

# File Upload Flow

```
Choose File

↓

Browser

↓

Multipart/Form-Data

↓

ASP.NET Core

↓

IFormFile

↓

Save File

↓

Response
```

---

# Common Properties of IFormFile

```csharp
file.FileName
```

Original file name.

---

```csharp
file.Length
```

File size.

---

```csharp
file.ContentType
```

MIME type.

Example

```
image/png

application/pdf
```

---

```csharp
file.OpenReadStream()
```

Returns a readable stream.

---

# Security Best Practices

Never trust uploaded files.

Always:

- Validate file type.
- Validate file size.
- Generate your own file names (avoid trusting `FileName` directly).
- Scan for malware if required.
- Store uploads outside the web root when appropriate.
- Restrict executable file uploads.

---

# Interview Questions

## What is `IFormFile`?

`IFormFile` represents a file sent with an HTTP request using `multipart/form-data`.

---

## Why is `multipart/form-data` used?

Because it efficiently transfers binary files along with optional form fields.

---

# Quick Revision

- File uploads use `multipart/form-data`.
- ASP.NET Core represents uploaded files with `IFormFile`.
- Use `CopyToAsync()` to save files.
- Always validate size, type, and file name for security.

---

# 83. Streaming

---

# What is Streaming?

Streaming means:

> **Sending or receiving data continuously instead of loading everything into memory first.**

Instead of waiting for the complete file,

data is processed piece by piece.

---

# Real-Life Analogy

Imagine watching a movie on Netflix.

Without streaming:

```
Download entire movie

↓

Watch movie
```

This could take several minutes.

With streaming:

```
Download first few seconds

↓

Start watching

↓

Continue downloading in the background
```

Much faster and uses less memory.

---

# Why Use Streaming?

Suppose you want to download a **5 GB file**.

Loading the whole file into memory would consume huge amounts of RAM.

Streaming avoids this by reading and sending small chunks.

---

# Streaming Flow

```
Large File

↓

Read Small Chunk

↓

Send

↓

Read Next Chunk

↓

Send

↓

Continue...
```

---

# ASP.NET Core Example

```csharp
[HttpGet]
public IActionResult Download()
{
    var stream = System.IO.File.OpenRead("Files/report.pdf");

    return File(stream, "application/pdf", "report.pdf");
}
```

ASP.NET Core streams the file instead of loading the entire file into memory.

---

# Reading Large Files

Instead of

```
Read Entire File

↓

Memory
```

Streaming does

```
Read 4 KB

↓

Process

↓

Read Next 4 KB

↓

Process
```

This reduces memory usage significantly.

---

# Where is Streaming Used?

- Video streaming
- Audio streaming
- File downloads
- File uploads
- Large CSV exports
- Large database results
- Cloud storage access

---

# Benefits

- Lower memory usage
- Faster response start time
- Better scalability
- Supports very large files
- Improved user experience

---

# Streaming vs Loading Everything

| Loading Entire File | Streaming |
|----------------------|-----------|
| High memory usage | Low memory usage |
| Wait until complete | Start immediately |
| Slower for large files | Faster for large files |
| Not ideal for huge files | Ideal for huge files |

---

# Interview Questions

## What is streaming?

Streaming is the process of transferring data in small chunks instead of loading the complete content into memory.

---

## Why is streaming useful?

It reduces memory usage, improves scalability, and allows processing of very large files efficiently.

---

# Quick Revision

- Streaming processes data in chunks.
- Ideal for large uploads and downloads.
- Minimizes memory consumption.
- Commonly used for videos, audio, and large files.

---

# 84. Pagination

---

# What is Pagination?

Pagination is the process of dividing a large dataset into **smaller pages**.

Instead of returning every record,

the API returns only a subset.

---

# Real-Life Analogy

Imagine reading a 500-page book.

You don't read all 500 pages at once.

You read:

```
Page 1

↓

Page 2

↓

Page 3
```

Similarly, APIs return data page by page.

---

# Why Do We Need Pagination?

Suppose a database contains:

```
10,000,000 Customers
```

Returning all records in one request would:

- Be slow
- Consume lots of memory
- Increase network traffic
- Slow down the client

Pagination solves this.

---

# Example

Instead of returning

```
10,000 records
```

Return

```
20 records
```

per request.

---

# API Example

```
GET /products?page=1&pageSize=10
```

Returns

```
Products 1–10
```

---

Next request

```
GET /products?page=2&pageSize=10
```

Returns

```
Products 11–20
```

---

# Pagination Flow

```
Database

↓

1...1000

↓

Page Size = 10

↓

Page 1

↓

Records 1-10

---------------

Page 2

↓

Records 11-20

---------------

Page 3

↓

Records 21-30
```

---

# ASP.NET Core Example

```csharp
[HttpGet]
public IActionResult GetUsers(int page = 1, int pageSize = 10)
{
    var users = db.Users
                  .Skip((page - 1) * pageSize)
                  .Take(pageSize)
                  .ToList();

    return Ok(users);
}
```

---

# How Skip and Take Work

Suppose

```
Page = 3

Page Size = 10
```

Calculation

```
Skip

(3-1) × 10

=

20
```

Then

```
Take

10
```

Result

```
21–30
```

---

# Response Example

```json
{
  "page": 2,
  "pageSize": 5,
  "totalRecords": 100,
  "totalPages": 20,
  "data": [
    ...
  ]
}
```

Providing metadata helps clients build pagination controls.

---

# Types of Pagination

## 1. Offset Pagination

Uses:

```
Skip

Take
```

Example

```
?page=3&pageSize=20
```

Simple but can become slower for very large offsets.

---

## 2. Cursor (Keyset) Pagination

Uses a unique value (such as an ID or timestamp) instead of page numbers.

Example

```
GET /products?afterId=100
```

Returns records after ID 100.

Better for very large datasets and frequently changing data.

---

# Offset vs Cursor Pagination

| Offset Pagination | Cursor Pagination |
|-------------------|-------------------|
| Uses page numbers | Uses cursor/key |
| Easy to implement | More efficient for large datasets |
| Can slow down with large offsets | Scales better |
| Best for small to medium datasets | Best for very large datasets |

---

# Best Practices

- Set a reasonable maximum page size.
- Always sort data before applying pagination.
- Return pagination metadata (`totalRecords`, `page`, `pageSize`, etc.).
- Use cursor pagination for large or real-time datasets.
- Validate page and page size values.

---

# Interview Questions

## What is pagination?

Pagination divides a large result set into smaller pages so clients receive manageable amounts of data.

---

## Why is pagination important?

It improves performance, reduces memory usage, lowers network traffic, and provides a better user experience.

---

## What are the common pagination techniques?

- Offset pagination (`Skip`/`Take`)
- Cursor (Keyset) pagination

---

## Why is cursor pagination preferred for large datasets?

Because it avoids scanning and skipping large numbers of rows, making it more efficient and scalable.

---

# Quick Revision

- Pagination returns data page by page.
- Use `Skip()` and `Take()` for offset pagination.
- Cursor pagination is better for very large datasets.
- Always include pagination metadata in API responses.
- Apply a consistent sort order before paginating.