# Composite Primary Keys

## Table of Contents
1. [What are Composite Primary Keys?](#what-are-composite-primary-keys)
2. [Creating an Entity that References a Table via Composite Primary Key](#creating-an-entity-that-references-a-table-via-composite-primary-key)
3. [Performing JOIN Queries with Composite Keys](#performing-join-queries-with-composite-keys)

---

## What are Composite Primary Keys?

Composite primary keys are formed by combining two or more columns in a table, ensuring that each row in the table is unique through a combination of columns rather than a single unique ID key.

This is useful, for example, when saving a new item like a car in a dealership system. If all data is identical except for the current ID and locking data, it doesn't make sense to save it in the database, creating unnecessary duplication. To prevent this, composite keys formed by combining multiple values can be used.

In the table below, if a new car is saved with the same `name`, `model`, and `quantity` data, PostgreSQL will throw an error:

```sql
CREATE TABLE IF NOT EXISTS car (
    name VARCHAR(30) NOT NULL,
    model VARCHAR(30) NOT NULL,
    quantity VARCHAR(30) NOT NULL,
    PRIMARY KEY(name, model, quantity)
);
```

---

## Creating an Entity that References a Table via Composite Primary Key

In the code below, three columns identical to the columns of the referenced table are created, and all three columns are used to create the foreign key.

```sql
CREATE TABLE IF NOT EXISTS car_motor (
    car_motor_id SERIAL PRIMARY KEY NOT NULL,
    name VARCHAR(30),
    model VARCHAR(30),
    quantity VARCHAR(30),
    FOREIGN KEY (name, model, quantity)
        REFERENCES car(name, model, quantity)
);
```

---

## Performing JOIN Queries with Composite Keys

In the case below, the JOIN query is performed using all the fields necessary to create the composite key.

```sql
SELECT 
    motor.car_motor_id, 
    car.model, 
    car.quantity, 
    car.name 
FROM car_motor AS motor
JOIN car ON 
    car.model = motor.model AND
    car.name = motor.name AND
    car.quantity = motor.quantity;
```