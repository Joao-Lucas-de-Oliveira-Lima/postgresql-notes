# PL/pgSQL, Transactions, Stored Procedures and Triggers

## Table of Contents

1. [Description](#description)
2. [Transactions](#transactions)

   * [Without explicit rollback](#without-explicit-rollback)
   * [Custom error handling](#custom-error-handling)
   * [Savepoints](#savepoints)
3. [Storing variables and rows](#storing-variables-and-rows)

   * [INTO](#into)
   * [Operator :=](#operator-)
   * [Storing a well-defined row](#storing-a-well-defined-row)
4. [Returning one or more rows via table()](#returning-one-or-more-rows-via-table)

   * [Using multiple return next;](#using-multiple-return-next)
   * [Using query to return multiple rows in select](#using-query-to-return-multiple-rows-in-select)
   * [Using setof to return tables](#using-setof-to-return-tables)
5. [IN, OUT, INOUT](#in-out-inout)
6. [IF, ELSEIF, ELSE](#if-elseif-else)
7. [Case](#case)
8. [Loops](#loops)

   * [Using simple loop](#using-simple-loop)
   * [For Loops](#for-loops)
   * [For loops with inverted values](#for-loops-with-inverted-values)
   * [FOR iterating through each element of a select](#for-iterating-through-each-element-of-a-select)
   * [While](#while)
9. [DO](#do)
10. [Foreach with arrays](#foreach-with-arrays)
11. [Stored Procedures](#stored-procedures)

    * [Functions vs procedures (main difference)](#functions-vs-procedures-main-difference)
    * [Creating a stored procedure](#creating-a-stored-procedure)
12. [Triggers](#triggers)

---

## Description

PL/pgSQL is a way to extend basic SQL by adding procedural features.
It allows using variables, control flow (`IF`, `CASE`, `LOOP`), error handling, and writing more complex database logic.

---

## Transactions

In PostgreSQL, a transaction is made by surrounding a code block with `BEGIN` and `COMMIT/END`.

### Without explicit rollback

When the `COMMIT` command is executed, if some error occurs previously, PostgreSQL will automatically perform a rollback.

```sql
BEGIN;
INSERT INTO authors (name, nationality) VALUES ('João', 'Test'); -- correct
INSERT INTO authors (id, name, nationality) VALUES (2, 'João', 'Test'); -- error
COMMIT;
```

### Custom error handling

Declaring an exception block allows for custom error handling.
`WHEN OTHERS THEN` means any type of exception.
`RAISE` re-throws the exception, and the rollback occurs before reaching the `END` clause.

```sql
CREATE OR REPLACE PROCEDURE p_sample() AS
$$
BEGIN
    INSERT INTO authors (name, nationality) VALUES ('João', 'Test'); -- correct
    INSERT INTO authors (id, name, nationality) VALUES (2, 'João', 'Test'); -- error
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END;
$$
LANGUAGE plpgsql;
```

### Savepoints

Savepoints allow partial rollback inside a transaction.

```sql
BEGIN;
    SAVEPOINT before_insert;

    INSERT INTO authors (name, nationality) VALUES ('Alice', 'USA');

    -- If something fails, rollback only to the savepoint
    ROLLBACK TO SAVEPOINT before_insert;

    INSERT INTO authors (name, nationality) VALUES ('Bob', 'UK');
COMMIT;
```

---

## Storing variables and rows

### INTO

The `INTO` operator stores the result of a `SELECT` into a variable.

```sql
CREATE OR REPLACE FUNCTION fn_sum(a INTEGER, b INTEGER)
RETURNS INTEGER AS
$$
DECLARE
    result INTEGER;
BEGIN
    SELECT a + b INTO result;
    RETURN result;
END;
$$
LANGUAGE plpgsql;

SELECT fn_sum(10, 20); -- output: 30
```

### Operator :=

The `:=` operator assigns values directly to variables.
The `=` operator is used only for comparisons.

```sql
CREATE OR REPLACE FUNCTION fn_sum(a INTEGER, b INTEGER)
RETURNS INTEGER AS
$$
DECLARE
    result INTEGER;
BEGIN
    result := a + b;
    RETURN result;
END;
$$
LANGUAGE plpgsql;
```

### Storing a well-defined row

You can store an entire row from a table into a variable of that table's type.

```sql
CREATE OR REPLACE FUNCTION fn_get_author(p_id INTEGER)
RETURNS authors AS
$$
DECLARE
    author_row authors;
BEGIN
    SELECT * INTO author_row
    FROM authors
    WHERE id = p_id;
    
    RETURN author_row;
END;
$$
LANGUAGE plpgsql;
```

---

## Returning one or more rows via table()

### Using multiple return next;

Each `RETURN NEXT` adds a new row to the result set.

```sql
CREATE OR REPLACE FUNCTION fn_return_numbers()
RETURNS TABLE (number_value INTEGER) AS
$$
BEGIN
    number_value := 1; RETURN NEXT;
    number_value := 2; RETURN NEXT;
    number_value := 3; RETURN NEXT;
END;
$$
LANGUAGE plpgsql;

SELECT * FROM fn_return_numbers();
```

### Using query to return multiple rows in select

```sql
CREATE OR REPLACE FUNCTION fn_return_multiple_sample()
RETURNS TABLE (
    author_id INTEGER,
    name VARCHAR(100),
    books_quantity BIGINT
) AS
$$
BEGIN
   RETURN QUERY
   SELECT a.id, a.name, COUNT(b.book_id) AS books_quantity
   FROM authors a
   LEFT JOIN books b ON a.id = b.author_id
   GROUP BY a.id, a.name;
END;
$$
LANGUAGE plpgsql;
```

### Using setof to return tables

```sql
CREATE OR REPLACE FUNCTION fn_return_all_authors()
RETURNS SETOF authors AS
$$
BEGIN
   RETURN QUERY
   SELECT * FROM authors;
END;
$$
LANGUAGE plpgsql;
```

---

## IN, OUT, INOUT

Parameters can be declared as `IN`, `OUT`, or `INOUT`:

```sql
CREATE OR REPLACE FUNCTION fn_test_params(
    IN  a INT,
    IN  b INT,
    OUT sum_result INT,
    OUT product_result INT,
    INOUT counter INT
) AS
$$
BEGIN
    sum_result := a + b;
    product_result := a * b;
    counter := counter + 1;
END;
$$
LANGUAGE plpgsql;

-- Usage:
SELECT * FROM fn_test_params(3, 4, 0);
```

---

## IF, ELSEIF, ELSE

```sql
CREATE OR REPLACE FUNCTION fn_integer_category(p_id INTEGER)
RETURNS TEXT AS
$$
BEGIN
    IF p_id > 0 THEN
        RETURN 'Positive number';
    ELSIF p_id < 0 THEN
        RETURN 'Negative number';
    ELSE
        RETURN 'Zero';
    END IF;
END;
$$
LANGUAGE plpgsql;
```

---

## Case

```sql
CREATE OR REPLACE FUNCTION fn_test_case()
RETURNS TEXT AS
$$
DECLARE
    subject DECIMAL := random();
    r_result TEXT;
BEGIN
    r_result := CASE
        WHEN subject < 0.5 THEN 'Below 50%'
        WHEN subject > 0.5 THEN 'Above 50%'
        ELSE 'Exactly 50%'
    END CASE;

    RETURN r_result;
END;
$$
LANGUAGE plpgsql;
```

---

## Loops

### Using simple loop

```sql
CREATE OR REPLACE FUNCTION fn_loop_sample()
RETURNS VOID AS
$$
DECLARE
    i INT := 1;
BEGIN
    LOOP
        RAISE NOTICE 'Iteration: %', i;
        i := i + 1;
        EXIT WHEN i > 5;
    END LOOP;
END;
$$
LANGUAGE plpgsql;
```

### For Loops

```sql
CREATE OR REPLACE FUNCTION fn_for_loop_sample(start_value INT, max_value INT)
RETURNS TABLE (current_value INT) AS
$$
BEGIN
    FOR i IN start_value .. max_value BY 1
	LOOP
        current_value := i;
        RETURN NEXT;
    END LOOP;
END;
$$
LANGUAGE plpgsql;
```

### For loops with inverted values

```sql
CREATE OR REPLACE FUNCTION fn_reverse_loop(min_value INT, max_value INT)
RETURNS TABLE(i_value INT) AS
$$
BEGIN
    FOR i IN REVERSE max_value .. min_value LOOP
        i_value := i;
        RETURN NEXT;
    END LOOP;
END;
$$
LANGUAGE plpgsql;
```

### FOR iterating through each element of a select

```sql
CREATE OR REPLACE FUNCTION fn_foreach()
RETURNS VOID AS
$$
DECLARE
    rec RECORD;
BEGIN
    FOR rec IN SELECT * FROM authors BY 1 
	LOOP
        RAISE NOTICE 'Author: % (% - %)', rec.id, rec.name, rec.nationality;
    END LOOP;
END;
$$
LANGUAGE plpgsql;
```

### While

```sql
CREATE OR REPLACE FUNCTION fn_while()
RETURNS VOID AS
$$
DECLARE
    i INT := 1;
BEGIN
    WHILE i <= 5 LOOP
        RAISE NOTICE 'Value is %', i;
        i := i + 1;
    END LOOP;
END;
$$
LANGUAGE plpgsql;
```

---

## DO

```sql
DO
$$
BEGIN
    RAISE NOTICE 'Hello world!';
END;
$$
LANGUAGE plpgsql;
```

---

## Foreach with arrays

```sql
DO
$$
DECLARE
    elements INT[] := ARRAY[1, 2, 3, 4, 5];
    i INT;
BEGIN
    FOREACH i IN ARRAY elements LOOP
        RAISE NOTICE 'Value: %', i;
    END LOOP;
END;
$$
LANGUAGE plpgsql;
```

---

## Stored Procedures

### Functions vs procedures (main difference)

* **Functions**: always return a value and cannot use transaction control (`COMMIT`, `ROLLBACK`).
* **Procedures**: can execute transaction control and don’t need to return a value. They are called with `CALL` instead of `SELECT`.

### Creating a stored procedure

```sql
CREATE OR REPLACE PROCEDURE p_test()
AS
$$
BEGIN
    INSERT INTO authors (name, nationality)
        VALUES ('João', 'Brazilian');
    RAISE EXCEPTION 'Error';
END;
$$
LANGUAGE plpgsql;

CALL p_test(); -- Rolls back automatically
```

---

## Triggers

```sql
CREATE OR REPLACE FUNCTION fn_trigger()
RETURNS TRIGGER AS
$$
BEGIN
    IF OLD.name <> NEW.name THEN
        INSERT INTO author_audit(message)
        VALUES ('Author updated from ' || OLD.name || ' to ' || NEW.name);
    END IF;
    RETURN NEW;
END;
$$
LANGUAGE plpgsql;

CREATE OR REPLACE TRIGGER trigger_sample
    BEFORE UPDATE
    ON authors
    FOR EACH ROW
    EXECUTE FUNCTION fn_trigger();

CREATE TABLE IF NOT EXISTS author_audit(
    message TEXT
);
```
