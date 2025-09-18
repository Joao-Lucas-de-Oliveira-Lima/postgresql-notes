# Indexes

## Table of Contents

1. [Introduction to Indexes](#introduction-to-indexes)
2. [Creating Indexes](#creating-indexes)
3. [Unique Indexes](#unique-indexes)
4. [Managing Indexes](#managing-indexes)

   * [Dropping Indexes](#dropping-indexes)
   * [Reindexing](#reindexing)
5. [Checking Index Space Usage](#checking-index-space-usage)

6. [Composite Index Column Ordering](#composite-index-column-ordering)

---

## Introduction to Indexes

Indexes are database objects that improve read performance and can enforce uniqueness in a table.

---

## Creating Indexes


```sql
-- Single-column index
CREATE INDEX idx_customer_email ON customer(email);

-- Multi-column index (order matters!)
CREATE INDEX idx_customer_name ON customer(first_name, last_name);

-- Partial index (with condition)
CREATE INDEX idx_active_users ON "user"(email) WHERE is_active = true;
```

---

## Unique Indexes

A unique index enforces data consistency in a table, ensuring that no duplicate values exist on the indexed column(s).

```sql
-- Ensure uniqueness on a single column
CREATE UNIQUE INDEX unique_idx_customer_email ON customer(email);

-- Composite unique index
CREATE UNIQUE INDEX unique_idx_customer_name
ON customer(first_name, last_name);
```

---

## Managing Indexes

### Dropping Indexes

Remove an index that is no longer needed:

```sql
DROP INDEX IF EXISTS idx_customer_email;
```

### Reindexing

Reindexing is useful to:

* Recover corrupted indexes due to system problems
* Optimize existing indexes by removing “dead tuples” that waste space

```sql
REINDEX INDEX idx_customer_email;
REINDEX TABLE customer;
REINDEX DATABASE mydb;
```

---

## Checking Index Space Usage

Check the size of indexes for a given table:

```sql
SELECT
    indexname AS index,
    pg_size_pretty(pg_relation_size(indexname::text)) AS size
FROM pg_indexes
WHERE tablename = 'table_name';
```

---

## Composite Index Column Ordering

When creating composite indexes, **column order is crucial** for optimal performance.
Place the most selective columns first in the index definition.

```sql
-- Example: If queries often filter by name, or by name + email together:
CREATE INDEX idx_name_email ON users(name, email) WHERE is_active = true;
```

**Key principles for column ordering:**

1. **Left-prefix rule**: Databases can efficiently use a composite index only if the query includes the leftmost column(s).

   * Queries filtering by `name` → ✅ uses the index
   * Queries filtering by `email` alone → ❌ does not use the index
   * Queries filtering by `name` and `email` → ✅ fully uses the index
