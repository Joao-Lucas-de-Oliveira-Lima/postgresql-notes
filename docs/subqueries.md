# Subqueries, CTEs, and EXISTS

## Table of Contents
1. [Scalar Subqueries in SELECT](#scalar-subqueries-in-select)
2. [Derived Tables (Subqueries in FROM)](#derived-tables-subqueries-in-from)
3. [Common Table Expressions (CTEs)](#common-table-expressions-ctes)
   - [Single CTE](#single-cte)
4. [EXISTS and NOT EXISTS](#exists-and-not-exists)
   - [EXISTS Usage](#exists-usage)
   - [NOT EXISTS Usage](#not-exists-usage)
   - [EXISTS vs IN](#exists-vs-in)

---

## Scalar Subqueries in SELECT

Scalar subqueries return a single value and can be used in the SELECT clause. Each subquery must return exactly one row and one column.

**Basic example with customer order statistics:**
```sql
SELECT
    customer_id,
    first_name,
    (SELECT COUNT(*) FROM orders WHERE customer_id = c.id) AS total_orders,
    (SELECT MAX(order_date) FROM orders WHERE customer_id = c.id) AS last_order
FROM customers c;
```

---

## Derived Tables (Subqueries in FROM)

Derived tables allow you to use the result of a subquery as a temporary table in your FROM clause.

**Simple example grouping by calculated values:**
```sql
SELECT
    order_count,
    COUNT(*) AS customers_with_this_many_orders
FROM (
    SELECT
        customer_id,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY customer_id
) AS customer_summary
GROUP BY order_count
ORDER BY order_count;
```
---

## Common Table Expressions (CTEs)

CTEs improve query readability by naming subqueries and making complex queries easier to understand.

### Single CTE

**Basic CTE example:**
```sql
WITH customer_stats AS (
    SELECT
        customer_id,
        COUNT(*) AS order_count,
        SUM(total) AS total_spent
    FROM orders
    GROUP BY customer_id
)
SELECT
    c.name,
    cs.order_count,
    cs.total_spent
FROM customers c
JOIN customer_stats cs ON c.id = cs.customer_id
WHERE cs.total_spent > 1000;
```
---

## EXISTS and NOT EXISTS

EXISTS checks for the existence of rows in a subquery and is often more efficient than IN for large datasets.

### EXISTS Usage

**Find customers who have placed orders:**
```sql
SELECT
    customer_id,
    name,
    email
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

**Find products that have been ordered:**
```sql
SELECT
    product_id,
    name,
    price
FROM products p
WHERE EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.id
);
```

### NOT EXISTS Usage

**Find customers who have never placed an order:**
```sql
SELECT
    customer_id,
    name,
    email
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

**Find products that have never been ordered:**
```sql
SELECT
    product_id,
    name,
    category
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.id
);
```

### EXISTS vs IN

**EXISTS is often preferred for performance:**

Using EXISTS (recommended):
```sql
SELECT name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
    AND o.order_date > '2024-01-01'
);
```

Using IN (can be slower with large datasets):
```sql
SELECT name
FROM customers
WHERE id IN (
    SELECT customer_id
    FROM orders
    WHERE order_date > '2024-01-01'
);
```