# INSERT, UPDATE, DELETE, CONFLICT, and RETURNING Clauses

## Table of Contents

1. [Inserting Data](#inserting-data)

   * [Single Insert](#single-insert)
   * [Multiple Inserts](#multiple-inserts)
   * [Handling Conflicts (Upsert)](#handling-conflicts-upsert)
   * [Ignoring Conflicts](#ignoring-conflicts)
2. [Updating Data](#updating-data)

   * [Basic Update](#basic-update)
   * [Conditional Updates with CASE](#conditional-updates-with-case)
3. [Deleting Data](#deleting-data)

   * [Basic Delete](#basic-delete)
4. [RETURNING Clause](#returning-clause)

---

## Inserting Data

### Single Insert

Use `INSERT INTO` to add a single record to a table:

```sql
INSERT INTO product (value, quantity, description)
VALUES (10.20, 100, 'Product Description');
```

### Multiple Inserts

Insert multiple records in a single statement for better performance:

```sql
INSERT INTO product (value, quantity, description)
VALUES
    (10.20, 100, 'First Product'),
    (20.50, 200, 'Second Product'),
    (15.75, 150, 'Third Product');
```

### Handling Conflicts (Upsert)

The `ON CONFLICT` clause handles constraint violations.
The `EXCLUDED` keyword refers to the values that were attempted to be inserted and can be used to turn an insert into an update:

```sql
INSERT INTO product (id, value, quantity, description)
VALUES (1, 25.00, 300, 'Updated Product')
ON CONFLICT (id)
DO UPDATE SET
    value = EXCLUDED.value,
    quantity = EXCLUDED.quantity,
    description = EXCLUDED.description,
    updated_at = CURRENT_TIMESTAMP;
```

### Ignoring Conflicts

`DO NOTHING` can be used to ignore inserts that would cause conflicts:

```sql
INSERT INTO product (id, value, quantity, description)
VALUES (1, 25.00, 300, 'Updated Product')
ON CONFLICT (id)
DO NOTHING;
```

---

## Updating Data

### Basic Update

Update a single record:

```sql
UPDATE users
SET username = 'new_username',
    updated_at = CURRENT_TIMESTAMP
WHERE id = 10;
```

### Conditional Updates with CASE

The `CASE` expression can be used for conditional updates.
It must end with `END`, and `ELSE` is recommended to handle default values.

Example: update status based on current value:

```sql
UPDATE orders
SET status = CASE
    WHEN status = 'pending' THEN 'processing'
    WHEN status = 'processing' THEN 'shipped'
    WHEN status = 'shipped' THEN 'delivered'
    ELSE status
END,
updated_at = CURRENT_TIMESTAMP
WHERE order_date >= '2024-01-01';
```

---

## Deleting Data

### Basic Delete

Remove records using the `DELETE` statement with appropriate `WHERE` clauses.

Delete a specific record:

```sql
DELETE FROM author WHERE id = 1;
```

⚠️ **Warning:** Deleting all rows should be done with extreme caution to avoid data loss:

```sql
DELETE FROM temp_table;
```

---

## RETURNING Clause

The `RETURNING` clause retrieves data from affected rows without requiring an additional `SELECT`.
This is especially useful for auto-generated IDs, timestamps, or verifying changes.

### With INSERT

Return generated IDs and timestamps:

```sql
INSERT INTO authors (name, birth_date, country)
VALUES ('Jane Doe', '1990-05-15', 'USA')
RETURNING author_id, created_at;
```

### With UPDATE

Return updated values:

```sql
UPDATE authors
SET 
    name = 'João Lucas',
    birth_date = birth_date + INTERVAL '10 years',
    country = 'Brazil',
    updated_at = CURRENT_TIMESTAMP
WHERE author_id = 2
RETURNING author_id, name, updated_at;
```

### With DELETE

Return data before deletion (useful for logging):

```sql
DELETE FROM authors 
WHERE author_id % 2 = 0 
RETURNING author_id, name, 'DELETED' AS status;
```
