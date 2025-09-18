# Operators and Functions

## Official Documentation
[Link](https://www.postgresql.org/docs/9.0/functions-math.html)

## Table of Contents
1. [Basic Comparison Operators](#basic-comparison-operators)
2. [Logical Operators](#logical-operators)
   - [Between](#between)
   - [Not Between](#not-between)
3. [NULL Checking](#null-checking)
4. [Arithmetic Operators](#arithmetic-operators)
5. [Pattern Matching](#pattern-matching)
   - [LIKE Examples](#like-examples)
6. [List Operations](#list-operations)
7. [JSON Operators](#json-operators)
   - [JSON Examples](#json-examples)
8. [Functions](#functions)
   - [Concat](#concat)
   - [Round](#round)

## Basic Comparison Operators
| Operator     | Description           | Example              |
| ------------ | --------------------- | -------------------- |
| `=`          | Equal                 | `age = 25`           |
| `!=` or `<>` | Not equal             | `status != 'active'` |
| `<`          | Less than             | `price < 100`        |
| `>`          | Greater than          | `quantity > 0`       |
| `<=`         | Less than or equal    | `age <= 65`          |
| `>=`         | Greater than or equal | `score >= 70`        |

## Logical Operators
| Operator | Description         | Example                           |
| -------- | ------------------- | --------------------------------- |
| `AND`    | Logical conjunction | `age > 18 AND status = 'active'` |
| `OR`     | Logical disjunction | `type = 'A' OR type = 'B'`        |
| `NOT`    | Negate an operation | `NOT (age > 30)`                  |

### Between

Between can be used to simplify `AND` and `NOT (AND)` predicates.

**Example:**
```sql
SELECT * FROM users WHERE age BETWEEN 50 AND 60;
```

Is equivalent to:
```sql
SELECT * FROM users WHERE age >= 50 AND age <= 60;
```

#### Not Between 

Not Between is just the predicate negated by the NOT operator.

```sql
SELECT * FROM users WHERE age NOT BETWEEN 18 AND 65;
```

## NULL Checking
| Operator     | Description        | Example              |
| ------------ | ------------------ | -------------------- |
| `IS NULL`    | Check for NULL     | `phone IS NULL`      |
| `IS NOT NULL`| Check for not NULL | `email IS NOT NULL`  |

To check if a value is NULL or not, the `IS` operator must be used. The comparison `email = NULL` will not work as intended.

## Arithmetic Operators
| Operator | Description       | Example            |
| -------- | ----------------- | ------------------ |
| `+`      | Addition          | `price + tax`      |
| `-`      | Subtraction       | `total - discount` |
| `*`      | Multiplication    | `quantity * price` |
| `/`      | Division          | `total / count`    |
| `%`      | Modulo            | `id % 2 = 0`       |
| `^`      | Exponentiation    | `2 ^ 3`            |
| `@`      | Absolute value    | `@ -5`             |
| `\|/`    | Square root       | `\|/ 25`           |

## Pattern Matching
| Operator | Description              | Example                    |
| -------- | ------------------------ | -------------------------- |
| `LIKE`   | Pattern matching         | `name LIKE 'John%'`        |
| `ILIKE`  | Case-insensitive LIKE    | `name ILIKE '%gmail%'`     |
| `~`      | Regex (case-sensitive)   | `name ~ '^[A-Z]'`          |
| `~*`     | Regex (case-insensitive) | `name ~* '^john'`          |

### LIKE Examples
```sql
-- Find names starting with "John"
SELECT * FROM users WHERE name LIKE 'John%';

-- Case-insensitive search
SELECT * FROM stores WHERE name ILIKE '%barnes%';

-- Case-insensitive with UPPER/LOWER
SELECT * FROM stores WHERE UPPER(name) LIKE UPPER('%barnes%');
```

## List Operations
| Operator | Description       | Example                           |
| -------- | ----------------- | --------------------------------- |
| `IN`     | Value in list     | `status IN ('active', 'pending')` |
| `NOT IN` | Value not in list | `id NOT IN (1, 2, 3)`             |

```sql
-- Instead of multiple OR statements
SELECT * FROM books WHERE category IN ('ROMANCE', 'ADVENTURE');
```

## JSON Operators
| Operator | Description             | Example                                    |
| -------- | ----------------------- | ------------------------------------------ |
| `->`     | Get JSON field as JSON  | `data->'name'`                             |
| `->>`    | Get JSON field as text  | `data->>'name'`                            |
| `?`      | Key exists              | `metadata->'phones' ? 'Number1'`          |
| `?&`     | All keys exist          | `metadata ?& '{"key1", "key2"}'`           |
| `?\|`    | Any key exists          | `metadata ?\| '{"key1", "key2"}'`          |
| `@>`     | Contains JSON           | `metadata @> '{"phones": ["Number1"]}'`   |

### JSON Examples
```sql
-- Check if key exists
SELECT * FROM posts WHERE metadata->'phones' ? 'Number1';

-- Check if JSON contains another JSON
SELECT * FROM posts WHERE metadata @> '{"phones": ["Number1", "Number2"]}';
```

## Functions

### Concat

It's also possible to concatenate the result of a query as in the example below:

```sql
SELECT CONCAT(title, ' - ', published_year) FROM books;
```

### Round

Used to control the precision of a decimal number.

```sql
SELECT ROUND(2.123444, 2); -- Output: 2.12
```