# ENUM Types


## Table of Contents
1. [Overview](#overview)
2. [Creating ENUM Types](#creating-enum-types)
   - [Basic Syntax](#basic-syntax)
   - [Using ENUM in Tables](#using-enum-in-tables)
3. [Viewing ENUM Types and Values](#viewing-enum-types-and-values)
   - [Show values for a specific ENUM type](#show-values-for-a-specific-enum-type)
   - [List all ENUM types and their values](#list-all-enum-types-and-their-values)
   - [Show all custom types (including ENUMs)](#show-all-custom-types-including-enums)
4. [Modifying ENUM Types](#modifying-enum-types)
   - [Adding New Values](#adding-new-values)
     - [Add before an existing value](#add-before-an-existing-value)
     - [Add after an existing value](#add-after-an-existing-value)
   - [Renaming ENUM Types](#renaming-enum-types)
   - [Renaming ENUM Values](#renaming-enum-values)
5. [Removing ENUM Types and Values](#removing-enum-types-and-values)
   - [Drop an ENUM type](#drop-an-enum-type)
   - [Removing Specific ENUM Values](#removing-specific-enum-values)

## Overview
ENUM types in PostgreSQL allow you to define a data type with a fixed set of predefined values. They are useful for columns that should only contain specific, known values.

## Creating ENUM Types

### Basic Syntax
```sql
CREATE TYPE type_name AS ENUM ('value1', 'value2', 'value3');
```


### Using ENUM in Tables
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    category category_type,
    status status_type DEFAULT 'pending'
);

-- Insert data
INSERT INTO products (name, category, status)
VALUES ('Wireless Headphones', 'electronics', 'active');
```

## Viewing ENUM Types and Values

### Show values for a specific ENUM type
```sql
SELECT enum_range(NULL::status_type);
```

### List all ENUM types and their values
Return the values using STRING_AGG from the enum.
```sql
SELECT
    n.nspname AS enum_schema,
    t.typname AS enum_name,
    STRING_AGG(e.enumlabel, ', ' ORDER BY e.enumsortorder) AS enum_values
FROM pg_type t
    JOIN pg_enum e ON t.oid = e.enumtypid
    JOIN pg_catalog.pg_namespace n ON n.oid = t.typnamespace
GROUP BY enum_schema, enum_name
ORDER BY enum_schema, enum_name;
```

### Show all custom types (including ENUMs)
Return the list of custom types and the type_category (char, varchar, etc...)
```sql
SELECT
    typname AS type_name,
    typtype AS type_category
FROM pg_type
WHERE typtype = 'e'  -- 'e' stands for enum
ORDER BY typname;
```

## Modifying ENUM Types

### Adding New Values

#### Add before an existing value
```sql
ALTER TYPE status_type ADD VALUE IF NOT EXISTS 'reviewing' BEFORE 'active';
```

#### Add after an existing value
```sql
ALTER TYPE status_type ADD VALUE IF NOT EXISTS 'archived' AFTER 'inactive';
```

### Renaming ENUM Types
```sql
ALTER TYPE category_type RENAME TO product_category_type;
```

### Renaming ENUM Values
```sql
ALTER TYPE status_type RENAME VALUE 'pending' TO 'awaiting_approval';
```

## Removing ENUM Types and Values

### Drop an ENUM type
```sql
DROP TYPE IF EXISTS category_type;
```

### Removing Specific ENUM Values
> **Important**: PostgreSQL doesn't allow direct removal of ENUM values. You must recreate the type.