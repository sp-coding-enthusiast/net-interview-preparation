# Web API Design — Pagination, Filtering, Sorting (Layman + Practical)

When APIs return large datasets, we must control **how much data is returned and how it is shaped**.

Without this, APIs become slow, expensive, and unusable.

---

# 1. Problem Statement (Why we need this)

Imagine this API:

```http
GET /api/products
```

It returns:

```text
10 million products
```

Problems:

- Very slow response
- High memory usage
- Network overload
- Client crashes (mobile apps)

---

# 2. Layman Analogy

Think of Amazon product search:

You don’t see:

```text
All products in the world
```

You see:

- First page (10–20 items)
- Filtered results (price, brand)
- Sorted results (price low → high)

That is exactly what APIs must do.

---

# 3. Pagination (Breaking data into pages)

## What it means

Instead of returning everything:

```text
Return small chunks of data
```

---

## Example API

```http
GET /api/products?page=1&size=10
```

---

## Response

```json
{
  "page": 1,
  "size": 10,
  "totalRecords": 1000,
  "totalPages": 100,
  "data": [
    { "id": 1, "name": "Laptop" },
    { "id": 2, "name": "Phone" }
  ]
}
```

---

## Pagination Types

### 1. Page-based pagination (most common)

```http
?page=2&size=20
```

---

### 2. Offset-based pagination

```http
?offset=40&limit=20
```

Used in SQL:

```sql
OFFSET 40 ROWS FETCH NEXT 20 ROWS ONLY
```

---

### 3. Cursor-based pagination (advanced)

```http
?cursor=abc123&limit=20
```

Used in:

- Instagram
- Twitter
- LinkedIn feeds

✔ Best for large datasets  
✔ Avoids performance issues

---

# 4. Filtering (Narrowing data)

## What it means

Get only data matching conditions.

---

## Example

```http
GET /api/products?category=electronics
```

---

## Multiple filters

```http
GET /api/products?category=electronics&brand=apple&minPrice=50000
```

---

## Real Example

```json
{
  "category": "electronics",
  "brand": "apple",
  "price": {
    "min": 50000,
    "max": 150000
  }
}
```

---

## SQL equivalent

```sql
SELECT * FROM Products
WHERE category = 'electronics'
AND brand = 'apple'
AND price >= 50000
```

---

# 5. Sorting (Ordering results)

## What it means

Arrange data in a specific order.

---

## Example

```http
GET /api/products?sort=price
```

---

## Ascending / Descending

```http
GET /api/products?sort=price_asc
GET /api/products?sort=price_desc
```

---

## Multiple sorting

```http
GET /api/products?sort=price_desc,rating_desc
```

Meaning:

1. Sort by price (high → low)
2. Then by rating

---

# 6. Combined Example (Real-world API)

```http
GET /api/products?page=2&size=20
&category=electronics
&brand=apple
&minPrice=50000
&sort=price_desc
```

---

## What it does:

- Page 2
- 20 items per page
- Only electronics
- Only Apple products
- Price >= 50,000
- Sorted by highest price first

---

# 7. ASP.NET Core Example

## Controller

```csharp
[HttpGet]
public IActionResult GetProducts(
    int page = 1,
    int size = 10,
    string? category = null,
    string? brand = null,
    decimal? minPrice = null,
    string sort = "price_desc")
{
    var query = _db.Products.AsQueryable();

    // Filtering
    if (!string.IsNullOrEmpty(category))
        query = query.Where(x => x.Category == category);

    if (!string.IsNullOrEmpty(brand))
        query = query.Where(x => x.Brand == brand);

    if (minPrice.HasValue)
        query = query.Where(x => x.Price >= minPrice);

    // Sorting
    query = sort switch
    {
        "price_asc" => query.OrderBy(x => x.Price),
        "price_desc" => query.OrderByDescending(x => x.Price),
        _ => query.OrderByDescending(x => x.Price)
    };

    // Pagination
    var totalRecords = query.Count();

    var data = query
        .Skip((page - 1) * size)
        .Take(size)
        .ToList();

    return Ok(new
    {
        page,
        size,
        totalRecords,
        data
    });
}
```

---

# 8. Performance Importance

Without pagination:

```text
100,000 records → slow API → timeout
```

With pagination:

```text
20 records → fast response
```

---

# 9. Best Practices

## ✔ Always use pagination for large data

Never return full datasets.

---

## ✔ Always combine filtering with pagination

Avoid unnecessary data load.

---

## ✔ Always use indexes in DB

Example:

```sql
INDEX(category, brand, price)
```

---

## ✔ Limit page size

```text
max size = 100
```

Prevent abuse.

---

# 10. Common Mistakes

---

## ❌ 1. Returning all data

```csharp
return _db.Products.ToList();
```

Problem:

- memory crash
- slow API

---

## ❌ 2. No default pagination

API becomes unpredictable.

---

## ❌ 3. Sorting after pagination

Wrong:

```text
Take(10) → then sort
```

Correct:

```text
Sort → then Take(10)
```

---

## ❌ 4. Missing indexes

Filtering becomes slow.

---

# 11. Advanced Concept: Cursor Pagination

Instead of page numbers:

```http
?cursor=abc123&limit=10
```

Used in:

- Facebook feed
- Twitter timeline
- LinkedIn posts

Benefits:

- faster
- scalable
- avoids duplicate records

---

# 12. Real Production Flow

```text
Client Request
    ↓
API Controller
    ↓
Apply Filters
    ↓
Apply Sorting
    ↓
Apply Pagination
    ↓
Database Query
    ↓
Return Response
```

---

# 13. Senior-Level Interview Answer

Pagination, filtering, and sorting are essential API design techniques used to efficiently manage large datasets. Pagination breaks data into smaller chunks to improve performance and reduce memory usage. Filtering allows clients to retrieve only relevant data based on conditions, while sorting ensures results are returned in a meaningful order. These techniques must always be applied at the database query level, not after fetching data, to ensure scalability and optimal performance in production systems.

---

# 14. Easy Way to Remember

```text
Filter → Reduce data
Sort → Order data
Paginate → Limit data
```

Or:

```text
Find → Arrange → Slice
```