# SQL Coding Standards — Superbotics

Applies to all raw `.sql` files, database migrations, and query strings in any language.

---

## Core Rules

- All SQL keywords must be written in **UPPERCASE**.
- All table names, column names, and aliases must be in `snake_case`.
- Every query must be written in a format that is human-readable without a query formatter.
- Never write a query that returns `SELECT *` in production — always name columns explicitly.
- All queries that modify data must be wrapped in a transaction where supported.

---

## File Header (for `.sql` files)

```sql
-- ============================================================
-- File: create_users_table.sql
-- Purpose: Creates the users table with all required fields,
--          constraints, and default values for the user entity.
-- Author: Developer Name
-- Created: YYYY-MM-DD
-- Last Modified: YYYY-MM-DD
-- ============================================================
```

---

## Naming Conventions

| Object           | Convention           | Example                       |
|------------------|----------------------|-------------------------------|
| Tables           | snake_case, plural   | `users`, `order_items`        |
| Columns          | snake_case           | `created_at`, `email_address` |
| Primary keys     | `id`                 | `id`                          |
| Foreign keys     | `[table_singular]_id`| `user_id`, `order_id`         |
| Indexes          | `idx_[table]_[col]`  | `idx_users_email_address`     |
| Unique indexes   | `uq_[table]_[col]`   | `uq_users_email_address`      |
| Foreign key constraints | `fk_[child_table]_[parent_table]` | `fk_orders_users` |
| Check constraints | `chk_[table]_[col]` | `chk_users_status`           |

---

## SELECT Queries

- Always name every column explicitly — no `SELECT *`.
- Use table aliases for multi-table queries — aliases must be descriptive abbreviations.
- One clause per line for readability.
- Column list alignment: one column per line, comma before the column name.

```sql
-- Correct — explicit columns, readable formatting
SELECT
    u.id                  AS user_id
  , u.full_name           AS user_full_name
  , u.email_address       AS user_email_address
  , u.created_at          AS user_created_at
  , r.role_name           AS assigned_role_name
FROM
    users AS u
INNER JOIN
    user_roles AS ur ON ur.user_id = u.id
INNER JOIN
    roles AS r ON r.id = ur.role_id
WHERE
    u.is_active = TRUE
    AND u.deleted_at IS NULL
ORDER BY
    u.created_at DESC
LIMIT 50;

-- Incorrect — never write this
SELECT * FROM users u JOIN user_roles ur on u.id = ur.user_id WHERE u.is_active = 1;
```

---

## INSERT Queries

- Always name every column in the INSERT — never rely on column position.

```sql
INSERT INTO users (
    full_name
  , email_address
  , hashed_password
  , is_active
  , created_at
  , updated_at
) VALUES (
    'John Doe'
  , 'john.doe@example.com'
  , 'hashed_value_here'
  , TRUE
  , NOW()
  , NOW()
);
```

---

## UPDATE Queries

- Always include a `WHERE` clause — never update without a filter.
- Always update `updated_at` in every UPDATE query.

```sql
UPDATE
    users
SET
    full_name   = 'Jane Doe'
  , is_active   = FALSE
  , updated_at  = NOW()
WHERE
    id = 42
    AND deleted_at IS NULL;
```

---

## DELETE Queries

- Prefer soft deletes — set `deleted_at = NOW()` instead of hard deletion.
- If a hard DELETE is required, always wrap in a transaction with a prior SELECT for verification.

```sql
-- Soft delete (preferred)
UPDATE
    users
SET
    deleted_at = NOW()
  , updated_at = NOW()
WHERE
    id = 42;

-- Hard delete — always in a transaction
BEGIN;

    SELECT id, email_address FROM users WHERE id = 42;
    -- Verify the record above before proceeding

    DELETE FROM users WHERE id = 42;

COMMIT;
```

---

## Table Creation (Migrations)

- Every table must have: `id`, `created_at`, `updated_at`, `deleted_at` (for soft deletes).
- Always define foreign key constraints explicitly.
- Always create indexes on foreign key columns and frequently queried columns.

```sql
-- ============================================================
-- File: 2024_01_15_create_orders_table.sql
-- Purpose: Creates the orders table representing customer orders.
-- Author: Developer Name
-- Created: 2024-01-15
-- Last Modified: 2024-01-15
-- ============================================================

CREATE TABLE orders (
    id                BIGINT          NOT NULL AUTO_INCREMENT
  , user_id           BIGINT          NOT NULL
  , order_status      VARCHAR(50)     NOT NULL DEFAULT 'pending'
  , total_amount_paise BIGINT         NOT NULL DEFAULT 0
  , notes             TEXT            NULL
  , created_at        TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP
  , updated_at        TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
  , deleted_at        TIMESTAMP       NULL

  , CONSTRAINT pk_orders
        PRIMARY KEY (id)

  , CONSTRAINT fk_orders_users
        FOREIGN KEY (user_id)
        REFERENCES users (id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE

  , CONSTRAINT chk_orders_status
        CHECK (order_status IN ('pending', 'confirmed', 'shipped', 'delivered', 'cancelled'))
);

-- Index on foreign key for join performance
CREATE INDEX idx_orders_user_id ON orders (user_id);

-- Index for status-based filtering
CREATE INDEX idx_orders_status ON orders (order_status);

-- Index for soft-delete filtering
CREATE INDEX idx_orders_deleted_at ON orders (deleted_at);
```

---

## Stored Procedures and Functions

- One procedure or function per file.
- Always declare `DELIMITER` changes when writing procedures in `.sql` files.
- All procedure names in `snake_case` with a verb prefix: `get_active_users`, `create_order_record`.

---

## Parameterisation

- Never concatenate user input directly into SQL strings in application code.
- Always use named or positional parameterised queries — the query placeholder syntax depends on the driver.

```sql
-- Parameterised query (application layer)
-- Correct representation — actual syntax varies by language driver

SELECT
    id
  , full_name
  , email_address
FROM
    users
WHERE
    id = :user_id
    AND is_active = TRUE;
```

---

## Forbidden Patterns

- No `SELECT *` in any production query.
- No UPDATE or DELETE without a WHERE clause.
- No raw string concatenation to build SQL queries in application code.
- No hardcoded IDs or magic numbers in migrations — use descriptive constants or variables.
- No cross-database queries without explicit documentation and approval.
- No dropping columns or tables without a corresponding data backup step documented in the migration file.
