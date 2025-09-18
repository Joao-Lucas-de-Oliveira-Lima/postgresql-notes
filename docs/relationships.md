# Relationships

## Table of Contents

1. [Many-to-Many Relationship](#many-to-many-relationship)
2. [One-to-One Relationship](#one-to-one-relationship)
3. [One-to-Many Relationship](#one-to-many-relationship)

---

## Many-to-Many Relationship

You can use a composite primary key as shown below in `student_course`, ensuring that the relationship is unique and won't accept duplicate entries in the `student_course` table with identical foreign keys, preventing database duplication.

```sql
CREATE TABLE IF NOT EXISTS student (
    student_id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE IF NOT EXISTS course (
    course_id BIGSERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL
);

-- Junction table
CREATE TABLE IF NOT EXISTS student_course (
    student_id BIGINT NOT NULL,
    course_id BIGINT NOT NULL,
    CONSTRAINT pk_student_course PRIMARY KEY (student_id, course_id),
    CONSTRAINT fk_student FOREIGN KEY (student_id) REFERENCES student (student_id),
    CONSTRAINT fk_course FOREIGN KEY (course_id) REFERENCES course (course_id)
);
```

**Key Points:**
- Uses a junction table to resolve the many-to-many relationship
- Composite primary key ensures uniqueness of the relationship

---

## One-to-One Relationship

To create a one-to-one relationship, simply add a `UNIQUE` constraint to the foreign key. This ensures that the primary key can only be used as a foreign key once.

```sql
CREATE TABLE IF NOT EXISTS event (
    event_id BIGSERIAL PRIMARY KEY NOT NULL,
    name VARCHAR(30)
);

CREATE TABLE IF NOT EXISTS financial_report (
    financial_report_id BIGSERIAL PRIMARY KEY NOT NULL,
    event_id BIGINT UNIQUE,
    CONSTRAINT fk_event_id FOREIGN KEY (event_id) REFERENCES event (event_id)
);
```

**Key Points:**
- The `UNIQUE` constraint on the foreign key enforces the one-to-one relationship
- Each event can have at most one financial report
- The foreign key can be nullable if the relationship is optional

---

## One-to-Many Relationship

The code below creates a simple one-to-many relationship where one customer can have multiple orders, using a foreign key to establish this relationship.

```sql
CREATE TABLE IF NOT EXISTS customer (
    customer_id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE IF NOT EXISTS orders (
    order_id BIGSERIAL PRIMARY KEY,
    order_date TIMESTAMP NOT NULL DEFAULT NOW(),
    amount NUMERIC(10,2) NOT NULL,
    customer_id BIGINT NOT NULL,
    CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES customer (customer_id)
);
```

**Key Points:**
- One customer can have many orders
- Foreign key in the "many" side (orders table) references the "one" side (customer table)