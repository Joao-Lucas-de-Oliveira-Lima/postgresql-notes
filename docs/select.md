# Reading Data

## Table of Contents
1. [List All Tables in the Database](#list-all-tables-in-the-database)
2. [List All Column Information for a Specific Table](#list-all-column-information-for-a-specific-table)
   - [Example Output](#example-output)
3. [Check if a Table is Not Empty](#check-if-a-table-is-not-empty)

## List All Tables in the Database
This query retrieves all base tables from the public schema in your PostgreSQL database:
```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
  AND table_type = 'BASE TABLE';
```

## List All Column Information for a Specific Table
This query provides detailed information about columns in a specific table, including their data types, nullable status, and default values:
```sql
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'your_table_name'
ORDER BY ordinal_position;
```

### Example Output
```
column_name          data_type            is_nullable  column_default
"author_id"          "integer"           "NO"         "nextval('authors_author_id_seq'::regclass)"
"name"               "character varying" "NO"        
"birth_date"         "date"              "YES"        
"country"            "character varying" "YES"        
```

## Check if a Table is Not Empty
To simply check for existence of data:
```sql
SELECT EXISTS(SELECT 1 FROM table_name);
```

To get the actual count:
```sql
-- Get total number of rows
SELECT COUNT(*) FROM table_name;
```