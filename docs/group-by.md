# Group By, Aggregate Functions and Having

## Table of Contents
1. [Group By and Aggregate Functions](#group-by-and-aggregate-functions)
2. [Having vs Where](#having-vs-where)
3. [Aggregate Functions](#aggregate-functions)
   - [Basic Aggregate Functions](#basic-aggregate-functions)
   - [Array and JSON Aggregate Functions](#array-and-json-aggregate-functions)
     - [Different Aggregation Types](#different-aggregation-types)


---

## Group By and Aggregate Functions

The `GROUP BY` clause groups rows with the same values. All selected columns must either be in an aggregate function (`SUM`, `AVG`, `COUNT`, etc.) or in the `GROUP BY` clause.

**Simple Example:**
```sql
SELECT 
    category,
    COUNT(*) as total_books,
    AVG(price) as avg_price
FROM books
GROUP BY category;
```

**With Joins:**
```sql
SELECT 
    author_name,
    COUNT(*) as book_count,
    SUM(price) as total_value
FROM authors 
JOIN books ON authors.id = books.author_id
GROUP BY author_name;
```

---

## Having vs Where

- **WHERE**: Filters rows before grouping
- **HAVING**: Filters groups after grouping

```sql
SELECT 
    category,
    COUNT(*) as book_count,
    AVG(price) as avg_price
FROM books
WHERE price > 10              -- Filter individual books
GROUP BY category
HAVING COUNT(*) > 5;          -- Filter categories with more than 5 books
```

---

## Aggregate Functions

### Basic Aggregate Functions

```sql
SELECT
    COUNT(*) as total_books,
    SUM(price) as total_value,
    AVG(price) as average_price,
    MIN(price) as cheapest,
    MAX(price) as most_expensive
FROM books;
```

**Common aggregate functions:**
- `COUNT(*)` - Count all rows
- `COUNT(column)` - Count non-null values
- `SUM(column)` - Sum of values
- `AVG(column)` - Average of values
- `MIN(column)` - Minimum value
- `MAX(column)` - Maximum value

### Array and JSON Aggregate Functions

These functions collect values into arrays or JSON formats. You can add `ORDER BY` to sort the results.

**Array Aggregation:**
```sql
SELECT 
    country,
    array_agg(city ORDER BY city) as cities
FROM customers
GROUP BY country;
```

**Result example:**
```
USA | {"Boston","Chicago","New York"}
```

#### Different Aggregation Types

**`array_agg`** - PostgreSQL array format
```sql
SELECT 
    category,
    array_agg(title) as book_titles
FROM books
GROUP BY category;
-- Result: {"Book 1","Book 2","Book 3"}
```

**`json_agg`** - JSON array format
```sql
SELECT 
    category,
    json_agg(title) as book_titles_json
FROM books
GROUP BY category;
-- Result: ["Book 1","Book 2","Book 3"]
```

**`jsonb_agg`** - Binary JSON (more efficient)
```sql
SELECT 
    category,
    jsonb_agg(title) as book_titles_jsonb
FROM books
GROUP BY category;
-- Result: ["Book 1","Book 2","Book 3"]
```