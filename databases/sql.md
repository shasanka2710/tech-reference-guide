# 🗄️ SQL Quick Reference

## Core SQL Commands

### DDL (Data Definition Language)

```sql
-- Create table
CREATE TABLE users (
    id         SERIAL PRIMARY KEY,
    name       VARCHAR(100)  NOT NULL,
    email      VARCHAR(255)  UNIQUE NOT NULL,
    age        INT           CHECK (age >= 0),
    created_at TIMESTAMP     DEFAULT NOW()
);

-- Modify table
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users ALTER COLUMN name TYPE TEXT;
ALTER TABLE users RENAME COLUMN name TO full_name;

-- Delete table
DROP TABLE IF EXISTS users;
TRUNCATE TABLE users;   -- removes all rows, faster than DELETE
```

### DML (Data Manipulation Language)

```sql
-- Insert
INSERT INTO users (name, email, age) VALUES ('Alice', 'alice@example.com', 30);

-- Bulk insert
INSERT INTO users (name, email) VALUES
    ('Bob',   'bob@example.com'),
    ('Carol', 'carol@example.com');

-- Update
UPDATE users SET age = 31 WHERE name = 'Alice';

-- Delete
DELETE FROM users WHERE id = 5;
```

### SELECT & Filtering

```sql
SELECT *          FROM users;
SELECT name, age  FROM users WHERE age > 25;
SELECT DISTINCT city FROM users;

-- Sorting
SELECT * FROM users ORDER BY age DESC, name ASC;

-- Limit & Offset (pagination)
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

-- Pattern matching
SELECT * FROM users WHERE name LIKE 'A%';     -- starts with A
SELECT * FROM users WHERE name LIKE '%bob%';  -- contains bob

-- IN / NOT IN
SELECT * FROM users WHERE age IN (25, 30, 35);

-- BETWEEN
SELECT * FROM users WHERE age BETWEEN 20 AND 40;

-- NULL check
SELECT * FROM users WHERE phone IS NULL;
SELECT * FROM users WHERE phone IS NOT NULL;
```

## Aggregate Functions

```sql
SELECT COUNT(*)           FROM users;
SELECT COUNT(DISTINCT age) FROM users;
SELECT AVG(age)           FROM users;
SELECT SUM(salary)        FROM employees;
SELECT MIN(age), MAX(age) FROM users;

-- GROUP BY
SELECT department, COUNT(*), AVG(salary)
FROM employees
GROUP BY department;

-- HAVING (filter after GROUP BY)
SELECT department, AVG(salary) AS avg_sal
FROM employees
GROUP BY department
HAVING AVG(salary) > 60000;
```

## JOINs

```sql
-- INNER JOIN: matching rows in both tables
SELECT u.name, o.product
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: all rows from left + matching from right
SELECT u.name, o.product
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN: all rows from right + matching from left
SELECT u.name, o.product
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN: all rows from both tables
SELECT u.name, o.product
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- SELF JOIN
SELECT a.name AS employee, b.name AS manager
FROM employees a
JOIN employees b ON a.manager_id = b.id;
```

## Subqueries & CTEs

```sql
-- Subquery in WHERE
SELECT name FROM users
WHERE id IN (SELECT user_id FROM orders WHERE amount > 100);

-- Subquery in FROM
SELECT avg_data.dept, avg_data.avg_sal
FROM (SELECT department AS dept, AVG(salary) AS avg_sal
      FROM employees GROUP BY department) AS avg_data
WHERE avg_data.avg_sal > 50000;

-- CTE (Common Table Expression)
WITH high_earners AS (
    SELECT id, name, salary
    FROM employees
    WHERE salary > 80000
)
SELECT * FROM high_earners ORDER BY salary DESC;

-- Recursive CTE (org chart)
WITH RECURSIVE org AS (
    SELECT id, name, manager_id FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id
    FROM employees e
    JOIN org ON e.manager_id = org.id
)
SELECT * FROM org;
```

## Window Functions

```sql
-- ROW_NUMBER, RANK, DENSE_RANK
SELECT name, salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num,
    RANK()       OVER (PARTITION BY department ORDER BY salary DESC) AS rnk,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rnk
FROM employees;

-- LAG / LEAD
SELECT name, salary,
    LAG(salary,  1, 0) OVER (ORDER BY id) AS prev_salary,
    LEAD(salary, 1, 0) OVER (ORDER BY id) AS next_salary
FROM employees;

-- SUM / AVG running total
SELECT name, salary,
    SUM(salary) OVER (ORDER BY id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM employees;
```

## Indexes

```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_users_email ON users(email);
CREATE INDEX idx_composite ON orders(user_id, created_at DESC);

-- Drop index
DROP INDEX idx_users_email;

-- Partial index (PostgreSQL)
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
```

## Transactions

```sql
BEGIN;                    -- or START TRANSACTION
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Rollback on error
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
ROLLBACK;

-- Savepoints
BEGIN;
    UPDATE t1 SET val = 1;
    SAVEPOINT sp1;
    UPDATE t2 SET val = 2;
    ROLLBACK TO sp1;   -- undo t2 only
COMMIT;
```

## Normalization

| Form | Rule |
|------|------|
| 1NF  | Atomic values, no repeating groups |
| 2NF  | 1NF + no partial dependency (all non-key columns depend on whole PK) |
| 3NF  | 2NF + no transitive dependency |
| BCNF | 3NF + every determinant is a candidate key |

## Common Functions

```sql
-- String
UPPER(name), LOWER(name)
TRIM(name), LTRIM(name), RTRIM(name)
LENGTH(name)
SUBSTRING(name, 1, 3)
CONCAT(first_name, ' ', last_name)
REPLACE(name, 'old', 'new')
COALESCE(col1, col2, 'default')   -- first non-null

-- Date/Time (PostgreSQL)
NOW(), CURRENT_TIMESTAMP
CURRENT_DATE, CURRENT_TIME
DATE_PART('year', created_at)
AGE(end_date, start_date)
created_at + INTERVAL '7 days'

-- Casting
CAST(price AS INTEGER)
price::INTEGER  -- PostgreSQL shorthand

-- CASE expression
SELECT name,
    CASE
        WHEN age < 18  THEN 'minor'
        WHEN age < 65  THEN 'adult'
        ELSE 'senior'
    END AS age_group
FROM users;
```
