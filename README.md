# PostgreSQL

Annotations from PostgreSQL commands for study and practice.

## Setup

### Requirements

* [Docker Desktop](https://www.docker.com/products/docker-desktop) installed and running

### Getting Started

PostgreSQL and **Adminer** services can be started with **Docker Compose**. Alternatively, you can also run these services with **pgAdmin** or **DBeaver**.

**Start PostgreSQL and Adminer:**

```bash
docker compose up -d
```

**Default connection URI inside Docker:**

```
postgresql://user:password@postgres:5432/database
```

**Adminer connection:**

* URL: [http://localhost:8080](http://localhost:8080)

## Other Services

1. **Start with pgAdmin**

   * URL: [http://localhost:8888](http://localhost:8888)
   * User: `admin@gmail.com`
   * Password: `admin`

   ```bash
   docker compose --profile pgadmin up -d
   ```

2. **Start with DBeaver**

   * URL: [http://localhost:8978](http://localhost:8978)

   ```bash
   docker compose --profile dbeaver up -d
   ```

---

## Table of Contents

* [SELECT Clause](/docs/select.md)
* [Basic SQL Functions](/docs/basic-functions.md)
* [INSERT, UPDATE, DELETE, CONFLICT, and RETURNING](/docs/insert-update-delete.md)
* [Operators and Functions](/docs/operators.md)
* [ALTER, DROP, TRUNCATE, and Constraints](/docs/alter-table.md)
* [Enums](/docs/enums.md)
* [GROUP BY, HAVING, and Aggregate Functions](/docs/group-by.md)
* [Relationships](/docs/relationships.md)
* [Indexes](/docs/indexes.md)
* [Subqueries, CTEs, EXISTS, and NOT EXISTS](/docs/subqueries.md)
* [Composite Primary Keys](/docs/composite-primary-keys.md)
* [JSON and JSONB](/docs/json.md)
* [PL/pgSQL: Transactions, Stored Procedures, and Triggers](/docs/plpgsql.md)

---
