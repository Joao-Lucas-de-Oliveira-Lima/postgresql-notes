# Basic SQL Functions

## Table of Contents

1. [Functions Overview](#functions-overview)
2. [Functions Returning Array of Tuples](#functions-returning-array-of-tuples)
   - [Creating Set-Returning Functions](#creating-set-returning-functions)
   - [Calling Set-Returning Functions](#calling-set-returning-functions)
3. [Functions Returning Single Row](#functions-returning-single-row)
4. [Parameter and Column Name Conflicts](#parameter-and-column-name-conflicts)
   - [Using Positional Parameters](#using-positional-parameters)

---

## Functions Overview

Once a function is created, if it needs to be modified and the modification involves the return type, parameters, or names, you can first delete it using `DROP FUNCTION <function_name>(parameter_types_list...)`.

**Example:**
```sql
DROP FUNCTION fn_function(bigint);
```

## Functions Returning Array of Tuples

### Creating Set-Returning Functions

Use the `SETOF` keyword to return multiple rows:

```sql
CREATE OR REPLACE FUNCTION fn_find_customer_by_country_or_city(
    p_city VARCHAR(30), 
    p_country VARCHAR(30)
)
RETURNS SETOF custom_customers AS
$body$
    SELECT * FROM custom_customers AS cc
    WHERE
        (p_city IS NULL OR cc.city ILIKE CONCAT('%', p_city, '%')) AND
        (p_country IS NULL OR cc.country ILIKE CONCAT('%', p_country, '%'))
$body$
LANGUAGE sql;
```

### Calling Set-Returning Functions

There are two ways to retrieve values from set-returning functions:

#### 1. Returning after SELECT clause
If the function return is not enclosed in parentheses `()` followed by `.*` or `.<column_name>`, the entire result will be brought within a single column.

```sql
SELECT (fn_find_customer_by_country_or_city(NULL, 'Portugal')).*;
SELECT (fn_find_customer_by_country_or_city(NULL, 'Portugal')).city;
```

#### 2. Returning after FROM clause
In this case, one or more returned rows will be processed normally:

```sql
SELECT * FROM fn_find_customer_by_country_or_city(NULL, 'Portugal');
SELECT country, city FROM fn_find_customer_by_country_or_city(NULL, 'Portugal');
```

## Functions Returning Single Row

Without the `SETOF` keyword, only the first occurrence of the row will be returned:

```sql
CREATE OR REPLACE FUNCTION fn_find_customer_by_country_or_city(
    p_city VARCHAR(30), 
    p_country VARCHAR(30)
)
RETURNS custom_customers AS
$body$
    SELECT * FROM custom_customers AS cc
    WHERE
        (p_city IS NULL OR cc.city ILIKE CONCAT('%', p_city, '%')) AND
        (p_country IS NULL OR cc.country ILIKE CONCAT('%', p_country, '%'))
$body$
LANGUAGE sql;
```

## Parameter and Column Name Conflicts

If parameter names are identical to column names, even when using aliases with `AS`, PostgreSQL may interpret them as the same. It's preferable to always have distinct names or reference them by their order using the `$` operator.

### Using Positional Parameters

When using `$`, you can omit parameter names as well:

```sql
CREATE OR REPLACE FUNCTION fn_find_customer_by_country_or_city(
    VARCHAR(30), 
    VARCHAR(30)
)
RETURNS SETOF custom_customers AS
$body$
    SELECT * FROM custom_customers AS cc
    WHERE
        ($1 IS NULL OR cc.city ILIKE CONCAT('%', $1, '%')) AND
        ($2 IS NULL OR cc.country ILIKE CONCAT('%', $2, '%'))
$body$
LANGUAGE sql;
```

> **Note:** `$1`, `$2`, etc. refer to the function parameters by their positional order.