# ALTER, DROP, TRUNCATE and Constraints

## Table of Contents

1. [Table Alterations](#table-alterations)
   - [Column Operations](#column-operations)
     - [Adding Columns](#adding-columns)
       - [Simple Column Addition](#simple-column-addition)
       - [Column with Constraints and Defaults](#column-with-constraints-and-defaults)
       - [Multiple Columns at Once](#multiple-columns-at-once)
     - [Removing Columns](#removing-columns)
       - [Single Column Removal](#single-column-removal)
       - [Multiple Column Removal](#multiple-column-removal)
     - [Renaming Columns](#renaming-columns)
     - [Changing Column Data Types](#changing-column-data-types)
       - [Simple Type Change](#simple-type-change)
       - [Type Change with Conversion](#type-change-with-conversion)
   - [Constraints](#constraints)
     - [Adding Constraints](#adding-constraints)
       - [Unique Constraints](#unique-constraints)
       - [Check Constraints](#check-constraints)
       - [Foreign Key Constraints](#foreign-key-constraints)
       - [Primary Key Constraints](#primary-key-constraints)
     - [Removing Constraints](#removing-constraints)
   - [NULL Constraints](#null-constraints)
     - [Adding NOT NULL](#adding-not-null)
     - [Removing NOT NULL](#removing-not-null)
   - [Table Operations](#table-operations)
     - [Renaming Tables](#renaming-tables)
2. [Data Deletion](#data-deletion)
   - [TRUNCATE Operations](#truncate-operations)
     - [Basic Truncate](#basic-truncate)
     - [Truncate with Identity Reset](#truncate-with-identity-reset)
     - [Multiple Table Truncate](#multiple-table-truncate)
     - [Truncate with CASCADE](#truncate-with-cascade)
   - [DROP Operations](#drop-operations)
     - [Basic Table Drop](#basic-table-drop)
     - [Drop with CASCADE](#drop-with-cascade)
     - [Multiple Table Drop](#multiple-table-drop)

---

## Table Alterations

### Column Operations

#### Adding Columns

##### Simple Column Addition

Add a basic column without constraints:

```sql
ALTER TABLE "user" ADD COLUMN cpf CHAR(11);
```

##### Column with Constraints and Defaults

Add a column with NOT NULL constraint and default value:

```sql
ALTER TABLE "user" ADD COLUMN phone VARCHAR(20) NOT NULL DEFAULT '';
```

##### Multiple Columns at Once

Add several columns in a single statement:

```sql
ALTER TABLE "user" 
ADD COLUMN created_at TIMESTAMP DEFAULT NOW(),
ADD COLUMN updated_at TIMESTAMP;
```

#### Removing Columns

##### Single Column Removal

Remove a column with safety check to avoid errors if column doesn't exist:

```sql
ALTER TABLE "user" DROP COLUMN IF EXISTS old_column;
```

##### Multiple Column Removal

Remove several columns in one operation:

```sql
ALTER TABLE "user" 
DROP COLUMN old_column1,
DROP COLUMN old_column2;
```

#### Renaming Columns

Change the name of an existing column:

```sql
ALTER TABLE "user" RENAME COLUMN name TO full_name;
```

#### Changing Column Data Types

##### Simple Type Change

Change column data type when no conversion is needed:

```sql
ALTER TABLE "user" ALTER COLUMN height TYPE NUMERIC(6,2);
```

##### Type Change with Conversion

Change data type with explicit conversion using USING clause:

```sql
ALTER TABLE "user"
ALTER COLUMN birth_date TYPE DATE
USING birth_date::DATE;
```

### Constraints

#### Adding Constraints

##### Unique Constraints

Ensure unique values in a column:

```sql
ALTER TABLE "user" ADD CONSTRAINT unique_cpf UNIQUE(cpf);
```

##### Check Constraints

**Age Validation**

Validate that age values are within reasonable range:

```sql
ALTER TABLE "user" ADD CONSTRAINT check_age 
CHECK (age >= 0 AND age <= 150);
```

**Non-empty String Validation**

Ensure strings have at least one character:

```sql
ALTER TABLE "user" ADD CONSTRAINT name_not_empty 
CHECK (char_length(name) > 0);
```

**Non-blank String Validation**

Ensure strings are not just whitespace:

```sql
ALTER TABLE "user" ADD CONSTRAINT name_not_blank 
CHECK (char_length(trim(name)) > 0);
```

**Category Validation**

Restrict values to specific options (enum substitute):

```sql
ALTER TABLE "user" ADD CONSTRAINT check_user_category 
CHECK (user_category IN ('admin', 'user', 'manager'));
```

##### Foreign Key Constraints

**Basic Foreign Key**

Create a reference to another table:

```sql
ALTER TABLE orders ADD CONSTRAINT fk_user_id
FOREIGN KEY (user_id) REFERENCES "user"(id);
```

**Foreign Key with CASCADE**

Create foreign key that automatically deletes related records:

```sql
ALTER TABLE orders ADD CONSTRAINT fk_user_id
FOREIGN KEY (user_id) REFERENCES "user"(id) 
ON DELETE CASCADE;
```

##### Primary Key Constraints

**Single Column Primary Key**

Add primary key to existing table:

```sql
ALTER TABLE "user" ADD CONSTRAINT pk_user_id PRIMARY KEY (id);
```

**Composite Primary Key**

Create primary key using multiple columns:

```sql
ALTER TABLE order_items ADD CONSTRAINT pk_order_items 
PRIMARY KEY (order_id, product_id);
```

#### Removing Constraints

Remove specific constraints with safety check:

```sql
ALTER TABLE "user" 
DROP CONSTRAINT IF EXISTS constraint_name;
```

### NULL Constraints

#### Adding NOT NULL

Before adding NOT NULL constraint, update any existing NULL values:

```sql
UPDATE "user" SET cpf = '' WHERE cpf IS NULL;

ALTER TABLE "user" ALTER COLUMN cpf SET NOT NULL;
```

#### Removing NOT NULL

Allow NULL values in a column:

```sql
ALTER TABLE "user" ALTER COLUMN cpf DROP NOT NULL;
```

### Table Operations

#### Renaming Tables

Change the name of an existing table:

```sql
ALTER TABLE old_table_name RENAME TO new_table_name;
```

---

## Data Deletion

### TRUNCATE Operations

TRUNCATE removes all rows from a table but preserves the table structure (including columns, constraints, indexes, etc.).

#### Basic Truncate
```sql
TRUNCATE TABLE table_name;
```

#### Truncate with Identity Reset

Remove all data and restart auto-increment sequences from 1:

```sql
TRUNCATE TABLE table_name RESTART IDENTITY;
```

#### Multiple Table Truncate

Clear multiple tables in one operation:

```sql
TRUNCATE TABLE table1, table2 RESTART IDENTITY;
```

#### Truncate with CASCADE

Handle foreign key dependencies by cascading the truncate operation:

```sql
TRUNCATE TABLE parent_table RESTART IDENTITY CASCADE;
```

### DROP Operations

DROP removes the entire table structure and all its data permanently.

#### Basic Table Drop

Remove a table with safety check to avoid errors:

```sql
DROP TABLE IF EXISTS table_name;
```

#### Drop with CASCADE

Remove table and all dependent objects (views, foreign keys, etc.):

```sql
DROP TABLE IF EXISTS table_name CASCADE;
```

#### Multiple Table Drop

Remove several tables in one operation:

```sql
DROP TABLE IF EXISTS table1, table2, table3;
```