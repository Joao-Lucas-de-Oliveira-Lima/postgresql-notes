# JSON

## Table of Contents
1. [Converting a string to JSON object](#converting-a-string-to-json-object)
2. [Accessing fields from JSON](#accessing-fields-from-json)
   - [Access a field from JSON](#access-a-field-from-json)
   - [Access and receive as text](#access-and-receive-as-text)
   - [Accessing an element from an array using index value](#accessing-an-element-from-an-array-using-index-value)
3. [Working with JSON tables](#working-with-json-tables)
   - [Insert values into a JSON table](#insert-values-into-a-json-table)
   - [Verifying if JSON has a property using the ? operator](#verifying-if-json-has-a-property-using-the--operator)
   - [Converting JSON key values](#converting-json-key-values)
   - [Checking if a key contains a specific value](#checking-if-a-key-contains-a-specific-value)
   - [Checking if a key contains all occurrences using the ?& operator](#checking-if-a-key-contains-all-occurrences-using-the--operator)
   - [Checking if a key contains at least one occurrence from a list using the ?| operator](#checking-if-a-key-contains-at-least-one-occurrence-from-a-list-using-the--operator)
   - [Checking if one JSON is contained within another using the @> operator](#checking-if-one-json-is-contained-within-another-using-the--operator)
4. [Indexing JSONB](#indexing-jsonb)

PostgreSQL offers support for two JSON types: `json` and `jsonb`.

**JSON** is raw data in text form and can be more efficient for saving, but is not optimized for reading and indexing.

**JSONB** is a binary tree structure that is much more optimized for data reading and querying.

## Converting a string to JSON object

```sql
SELECT '{}'::json;

-- Example 2
SELECT '{
    "name": "João",
    "address": {
        "Country": "Brazil"
    },
    "phones": ["+55 (11) 99999-9999", "+55 (22) 92222-2222"]
}'::jsonb;
```

## Accessing fields from JSON

### Access a field from JSON
Using a single `->` indicates that we want to access the value of a specific field from the JSON.

```sql
SELECT '{
    "name": "João",
    "address": {
        "Country": "Brazil"
    },
    "phones": ["+55 (11) 99999-9999", "+55 (22) 92222-2222"]
}'::jsonb -> 'address';
```

**Output:** `{"Country": "Brazil"}`

### Access and receive as text
Using double `->>` indicates that in addition to accessing the field, we want to receive the value as text.

```sql
SELECT '{
    "name": "João",
    "address": {
        "Country": "Brazil"
    },
    "phones": ["+55 (11) 99999-9999", "+55 (22) 92222-2222"]
}'::jsonb ->> 'name';
```

**Output:** `João`

### Accessing an element from an array using index value
Using `->0`, we are specifying that we want the value at index 0 from the array.

```sql
SELECT '{
    "name": "João",
    "address": {
        "Country": "Brazil"
    },
    "phones": ["+55 (11) 99999-9999", "+55 (22) 92222-2222"]
}'::jsonb -> 'phones' -> 0;
```

**Output:** `"+55 (11) 99999-9999"`

## Working with JSON tables

### Insert values into a JSON table

```sql
CREATE TABLE IF NOT EXISTS posts(
    id SERIAL PRIMARY KEY NOT NULL,
    metadata JSONB
);

INSERT INTO posts (metadata)
VALUES
('{
    "name": "test"
}'::jsonb),
('{
    "phones": ["Number1", "Number2"]
}'::jsonb);
```

### Verifying if JSON has a property using the `?` operator
The query below will return all rows from posts where the 'phones' property exists.

```sql
SELECT * FROM posts WHERE metadata ? 'phones';
```

### Converting JSON key values
You can extract data as text and then cast it using the `::` operator to the chosen type for verification.

```sql
SELECT * FROM posts WHERE (metadata ->> 'id')::int > 5;
```

### Checking if a key contains a specific value

```sql
SELECT * FROM posts WHERE metadata -> 'phones' ? 'Number1';
```

### Checking if a key contains all occurrences using the `?&` operator

```sql
SELECT * FROM posts WHERE metadata -> 'phones' ?& '{"Number1", "Number2"}';
```

### Checking if a key contains at least one occurrence from a list using the `?|` operator

```sql
SELECT * FROM posts WHERE metadata -> 'phones' ?| '{"Number1", "Number2"}';
```

### Checking if one JSON is contained within another using the `@>` operator

```sql
SELECT * FROM posts WHERE metadata @> '{"phones": ["Number1", "Number2"]}';
```

## Indexing JSONB
To index a JSONB field as a whole, you can use the following command:

```sql
CREATE INDEX IF NOT EXISTS json_index ON posts USING GIN (metadata);
```