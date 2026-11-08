# SQL Complete Reference & Practical Cookbook

## Table of Contents
1. [Core Sample Database Schema & Data](#1-core-sample-database-schema--data)
2. [Query Execution Pipeline & Syntax Anatomy](#2-query-execution-pipeline--syntax-anatomy)
3. [DDL: Data Definition Language & Integrity Constraints](#3-ddl-data-definition-language--integrity-constraints)
4. [DML: Data Manipulation & Upsert Mechanics](#4-dml-data-manipulation--upsert-mechanics)
5. [Filtering, Predicates & NULL Mechanics](#5-filtering-predicates--null-mechanics)
6. [Joins: All Variants, Venn Logic & Outputs](#6-joins-all-variants-venn-logic--outputs)
7. [Aggregations & Grouping Extensions (ROLLUP, CUBE)](#7-aggregations--grouping-extensions-rollup-cube)
8. [Window Functions (Ranking, Value, Frames)](#8-window-functions-ranking-value-frames)
9. [Subqueries & CTEs (Correlated, Non-Correlated, Recursive)](#9-subqueries--ctes-correlated-non-correlated-recursive)
10. [Set Operators: UNION, INTERSECT, EXCEPT](#10-set-operators-union-intersect-except)
11. [Views, Materialized Views & Stored Routines](#11-views-materialized-views--stored-routines)
12. [Indexing Strategies & Query Performance Optimization](#12-indexing-strategies--query-performance-optimization)
13. [Transactions, Isolation Levels & ACID Mechanics](#13-transactions-isolation-levels--acid-mechanics)

---

## 1. Core Sample Database Schema & Data

The examples throughout this guide execute against the following standardized schema:

```sql
-- Teardown
DROP TABLE IF EXISTS audit_logs CASCADE;
DROP TABLE IF EXISTS order_items CASCADE;
DROP TABLE IF EXISTS orders CASCADE;
DROP TABLE IF EXISTS products CASCADE;
DROP TABLE IF EXISTS employees CASCADE;
DROP TABLE IF EXISTS departments CASCADE;

-- 1. Departments
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50) NOT NULL,
    location VARCHAR(50) NOT NULL
);

-- 2. Employees (Self-referencing foreign key for org hierarchies)
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    dept_id INT REFERENCES departments(dept_id),
    manager_id INT REFERENCES employees(emp_id),
    salary NUMERIC(10, 2) NOT NULL,
    hire_date DATE NOT NULL,
    status VARCHAR(20) DEFAULT 'ACTIVE'
);

-- 3. Products
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    stock_qty INT DEFAULT 0
);

-- 4. Orders
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount NUMERIC(10, 2) NOT NULL,
    order_status VARCHAR(20) DEFAULT 'COMPLETED'
);

-- 5. Order Items (Bridge table)
CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INT REFERENCES products(product_id),
    quantity INT NOT NULL,
    unit_price NUMERIC(10, 2) NOT NULL
);
```

### Reference Seed Data

```sql
INSERT INTO departments VALUES
(1, 'Engineering', 'Building A'),
(2, 'Sales', 'Building B'),
(3, 'Marketing', 'Building B'),
(4, 'Legal', 'Building C');

INSERT INTO employees VALUES
(101, 'Ada', 'Lovelace', 1, NULL, 150000.00, '2021-01-15', 'ACTIVE'),
(102, 'Alan', 'Turing', 1, 101, 130000.00, '2021-03-01', 'ACTIVE'),
(103, 'Grace', 'Hopper', 1, 101, 135000.00, '2021-06-20', 'ACTIVE'),
(104, 'John', 'von Neumann', 2, NULL, 120000.00, '2022-01-10', 'ACTIVE'),
(105, 'Claude', 'Shannon', 2, 104, 95000.00, '2022-05-18', 'ACTIVE'),
(106, 'Tim', 'Berners-Lee', 3, NULL, 110000.00, '2022-08-01', 'ACTIVE'),
(107, 'Linus', 'Torvalds', NULL, NULL, 140000.00, '2023-01-01', 'PROBATION');

INSERT INTO products VALUES
(1, 'ThinkPad X1', 'Electronics', 1400.00, 25),
(2, 'Dell XPS 15', 'Electronics', 1800.00, 10),
(3, 'Mechanical Keyboard', 'Accessories', 120.00, 150),
(4, 'Ergonomic Mouse', 'Accessories', 80.00, 200),
(5, '4K Monitor', 'Displays', 450.00, 40);

INSERT INTO orders VALUES
(1001, 501, '2026-01-10', 1520.00, 'COMPLETED'),
(1002, 502, '2026-01-15', 1800.00, 'COMPLETED'),
(1003, 501, '2026-02-01', 200.00, 'CANCELLED'),
(1004, 503, '2026-02-14', 2370.00, 'COMPLETED');

INSERT INTO order_items VALUES
(1, 1001, 1, 1, 1400.00),
(2, 1001, 3, 1, 120.00),
(3, 1002, 2, 1, 1800.00),
(4, 1003, 4, 1, 80.00),
(5, 1003, 3, 1, 120.00),
(6, 1004, 2, 1, 1800.00),
(7, 1004, 5, 1, 450.00),
(8, 1004, 3, 1, 120.00);
```

---

## 2. Query Execution Pipeline & Syntax Anatomy

### Logical Phase Sequence

An engine does **not** process clauses top-down. The SQL query pipeline runs in this strict sequence:

$$
\text{FROM / JOIN} \xrightarrow{1} \text{WHERE} \xrightarrow{2} \text{GROUP BY} \xrightarrow{3} \text{HAVING} \xrightarrow{4} \text{SELECT} \xrightarrow{5} \text{DISTINCT} \xrightarrow{6} \text{ORDER BY} \xrightarrow{7} \text{LIMIT / OFFSET}
$$

### Execution Pipeline Rules

1. **`FROM & JOIN`**: Base tables are accessed and cross/joined into an interim working table.
2. **`WHERE`**: Filters records *prior* to any aggregation. Column aliases declared in `SELECT` **cannot** be used here.
3. **`GROUP BY`**: Collapses raw rows into distinct summary buckets.
4. **`HAVING`**: Filters grouped rows *after* aggregate functions ($COUNT, SUM, AVG$) evaluate.
5. **`SELECT`**: Evaluates expressions, assigns column aliases, and builds output tuples.
6. **`DISTINCT`**: Eliminates duplicate projected tuples.
7. **`ORDER BY`**: Sorts final result sets. Column aliases from `SELECT` are accessible here.
8. **`LIMIT / OFFSET`**: Discards leading rows and slices the final row count.

---

## 3. DDL: Data Definition Language & Integrity Constraints

### Creating Constraints & Referential Actions

```sql
CREATE TABLE accounts (
    account_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    iban VARCHAR(34) NOT NULL UNIQUE,
    balance NUMERIC(15, 2) NOT NULL DEFAULT 0.00,
    currency VARCHAR(3) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,

    -- Table Constraints
    CONSTRAINT chk_positive_balance CHECK (balance >= 0.00),
    CONSTRAINT chk_valid_currency CHECK (currency IN ('USD', 'EUR', 'INR', 'GBP'))
);

CREATE TABLE account_cards (
    card_id BIGINT PRIMARY KEY,
    account_id BIGINT NOT NULL,
    card_number VARCHAR(16) UNIQUE NOT NULL,
    expiry_date DATE NOT NULL,

    -- Foreign Key Actions: CASCADE, SET NULL, RESTRICT, NO ACTION
    CONSTRAINT fk_account
        FOREIGN KEY (account_id)
        REFERENCES accounts(account_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

### Table Alters

```sql
-- Add column
ALTER TABLE employees ADD COLUMN middle_name VARCHAR(50);

-- Modify type/nullability
ALTER TABLE employees ALTER COLUMN middle_name SET DEFAULT 'N/A';

-- Add check constraint post-creation
ALTER TABLE employees ADD CONSTRAINT chk_salary_min CHECK (salary >= 25000);

-- Drop constraint
ALTER TABLE employees DROP CONSTRAINT chk_salary_min;
```

---

## 4. DML: Data Manipulation & Upsert Mechanics

### Basic Modifications

```sql
-- Multi-row insert
INSERT INTO products (product_id, product_name, category, price, stock_qty)
VALUES 
    (6, 'Webcam 1080p', 'Accessories', 65.00, 80),
    (7, 'USB-C Dock', 'Accessories', 180.00, 45);

-- Conditional update using joined data
UPDATE employees e
SET salary = e.salary * 1.10
FROM departments d
WHERE e.dept_id = d.dept_id
  AND d.dept_name = 'Engineering';

-- Sliced deletion
DELETE FROM products WHERE stock_qty = 0;
```

### Idiomatic UPSERT (Insert or Update on Key Conflict)

```sql
-- PostgreSQL / SQLite / modern engines
INSERT INTO products (product_id, product_name, category, price, stock_qty)
VALUES (1, 'ThinkPad X1 Carbon Gen 12', 'Electronics', 1450.00, 30)
ON CONFLICT (product_id) 
DO UPDATE SET 
    product_name = EXCLUDED.product_name,
    price = EXCLUDED.price,
    stock_qty = products.stock_qty + EXCLUDED.stock_qty;

-- MySQL Equivalent
INSERT INTO products (product_id, product_name, category, price, stock_qty)
VALUES (1, 'ThinkPad X1 Carbon Gen 12', 'Electronics', 1450.00, 30)
ON DUPLICATE KEY UPDATE
    product_name = VALUES(product_name),
    price = VALUES(price),
    stock_qty = stock_qty + VALUES(stock_qty);
```

---

## 5. Filtering, Predicates & NULL Mechanics

### Three-Valued Logic (3VL)

In SQL, boolean expressions evaluate to `TRUE`, `FALSE`, or `UNKNOWN`. `NULL` signifies an absent or unknown value.

| Expression | Evaluates To | Note |
| :--- | :--- | :--- |
| `NULL = NULL` | `UNKNOWN` | Cannot equate unknown states |
| `NULL != NULL` | `UNKNOWN` | Inequality remains unknown |
| `NULL IS NULL` | `TRUE` | Proper check |
| `TRUE AND UNKNOWN` | `UNKNOWN` | Needs all conditions true |
| `FALSE AND UNKNOWN` | `FALSE` | Short-circuited by false |
| `TRUE OR UNKNOWN` | `TRUE` | Short-circuited by true |

```sql
-- SAFE: Checking NULL status
SELECT emp_id, first_name 
FROM employees 
WHERE manager_id IS NULL;

-- DANGEROUS: NOT IN with NULL elements
-- Returns ZERO rows if ANY record returned by the subquery contains a NULL value!
SELECT emp_id, first_name 
FROM employees 
WHERE emp_id NOT IN (SELECT manager_id FROM employees); -- Fails if manager_id has NULLs

-- SAFE REPLACEMENT: NOT EXISTS
SELECT e.emp_id, e.first_name 
FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM employees m WHERE m.manager_id = e.emp_id
);
```

### Comprehensive Predicates

```sql
SELECT 
    product_name, 
    price,
    COALESCE(stock_qty, 0) AS safe_stock,
    NULLIF(price, 0) AS non_zero_price,
    CASE 
        WHEN price >= 1000 THEN 'Premium'
        WHEN price BETWEEN 200 AND 999.99 THEN 'Standard'
        ELSE 'Budget'
    END AS tier
FROM products
WHERE category ILIKE '%access%'     -- Case-insensitive regex/pattern match
  AND price BETWEEN 50 AND 300      -- Inclusive range
  AND product_id IN (3, 4, 7);
```

---

## 6. Joins: All Variants, Venn Logic & Outputs

```
+---------------+---------------+--------------------+--------------------+
|  INNER JOIN   |   LEFT JOIN   |     RIGHT JOIN     |  FULL OUTER JOIN   |
|   [ A ∩ B ]   |   [ A + ∩ ]   |     [ ∩ + B ]      |     [ A ∪ B ]      |
+---------------+---------------+--------------------+--------------------+
```

### INNER JOIN

Retrieves strictly rows where keys exist in both sets.

```sql
SELECT e.emp_id, e.first_name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

**Output:**
```
 emp_id | first_name |  dept_name  
--------+------------+-------------
    101 | Ada        | Engineering
    102 | Alan       | Engineering
    103 | Grace      | Engineering
    104 | John       | Sales
    105 | Claude     | Sales
    106 | Tim        | Marketing
(6 rows, Linus Torvalds omitted because dept_id IS NULL)
```

### LEFT OUTER JOIN

Returns all left rows; unmatched right attributes populate as `NULL`.

```sql
SELECT e.emp_id, e.first_name, COALESCE(d.dept_name, 'Unassigned') AS dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

**Output:**
```
 emp_id | first_name |  dept_name  
--------+------------+-------------
    101 | Ada        | Engineering
    102 | Alan       | Engineering
    103 | Grace      | Engineering
    104 | John       | Sales
    105 | Claude     | Sales
    106 | Tim        | Marketing
    107 | Linus      | Unassigned
```

### FULL OUTER JOIN

Preserves rows from both sides; unmatched keys on either side produce `NULL` columns.

```sql
SELECT e.first_name, d.dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.dept_id;
```

**Output:**
```
 first_name |  dept_name  
------------+-------------
 Ada        | Engineering
 Alan       | Engineering
 Grace      | Engineering
 John       | Sales
 Claude     | Sales
 Tim        | Marketing
 Linus      | NULL
 NULL       | Legal        <-- Department with no assigned employees
```

### SELF JOIN

Joins a table to itself to resolve graph hierarchies.

```sql
SELECT 
    e.first_name AS employee,
    COALESCE(m.first_name, 'No Boss (Root)') AS direct_manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;
```

**Output:**
```
  employee   | direct_manager 
-------------+----------------
 Ada         | No Boss (Root)
 Alan        | Ada
 Grace       | Ada
 John        | No Boss (Root)
 Claude      | John
 Tim         | No Boss (Root)
 Linus       | No Boss (Root)
```

### CROSS JOIN (Cartesian Product)

Every row of table $A$ paired with every row of table $B$ ($M \times N$).

```sql
SELECT d.dept_name, p.category
FROM departments d
CROSS JOIN (SELECT DISTINCT category FROM products) p
WHERE d.dept_id = 4;
```

**Output:**
```
 dept_name |  category   
-----------+-------------
 Legal     | Electronics
 Legal     | Accessories
 Legal     | Displays
```

---

## 7. Aggregations & Grouping Extensions (ROLLUP, CUBE)

### Standard Grouping & Filtering with HAVING

```sql
SELECT 
    d.dept_name,
    COUNT(e.emp_id) AS head_count,
    ROUND(AVG(e.salary), 2) AS avg_comp,
    MAX(e.salary) AS peak_salary
FROM departments d
INNER JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_name
HAVING COUNT(e.emp_id) >= 2;
```

**Output:**
```
  dept_name  | head_count | avg_comp  | peak_salary 
-------------+------------+-----------+-------------
 Engineering |          3 | 138333.33 |   150000.00
 Sales       |          2 | 107500.00 |   120000.00
```

### Hierarchical Aggregation: ROLLUP

Produces subtotals and grand totals along an ordered hierarchy.

```sql
SELECT 
    COALESCE(d.dept_name, 'All Depts') AS department,
    COALESCE(e.status, 'Dept Total') AS status,
    SUM(e.salary) AS total_payroll
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
GROUP BY ROLLUP (d.dept_name, e.status);
```

**Output:**
```
 department  |   status   | total_payroll 
-------------+------------+---------------
 Engineering | ACTIVE     |     415000.00
 Engineering | Dept Total |     415000.00
 Sales       | ACTIVE     |     215000.00
 Sales       | Dept Total |     215000.00
 Marketing   | ACTIVE     |     110000.00
 Marketing   | Dept Total |     110000.00
 All Depts   | PROBATION  |     140000.00
 All Depts   | Dept Total |     140000.00
 All Depts   | Dept Total |     880000.00 <-- Grand Total
```

### Multi-Dimensional Aggregation: CUBE

Calculates aggregations across all possible permutations of the specified columns ($2^N$ combinations).

```sql
SELECT 
    category,
    CASE WHEN price > 500 THEN 'High' ELSE 'Low' END AS price_tier,
    COUNT(*) AS total_items
FROM products
GROUP BY CUBE (category, (CASE WHEN price > 500 THEN 'High' ELSE 'Low' END));
```

---

## 8. Window Functions (Ranking, Value, Frames)

Unlike `GROUP BY`, window functions **do not collapse** rows. Every individual row retains its identity while appending the calculated aggregate value.

$$
\text{FUNCTION}(\dots) \ \text{OVER} \ \Big( [\text{PARTITION BY } c_1, c_2] \ [\text{ORDER BY } o_1] \ [\text{FRAME\_CLAUSE}] \Big)
$$

### 1. Ranking Functions: ROW_NUMBER, RANK, DENSE_RANK

```sql
SELECT 
    first_name, 
    dept_id, 
    salary,
    ROW_NUMBER() OVER w AS row_num,
    RANK()       OVER w AS rnk,
    DENSE_RANK() OVER w AS dense_rnk
FROM employees
WINDOW w AS (PARTITION BY dept_id ORDER BY salary DESC);
```

**Output:**
```
 first_name | dept_id |  salary   | row_num | rnk | dense_rnk 
------------+---------+-----------+---------+-----+-----------
 Ada        |       1 | 150000.00 |       1 |   1 |         1
 Grace      |       1 | 135000.00 |       2 |   2 |         2
 Alan       |       1 | 130000.00 |       3 |   3 |         3
 John       |       2 | 120000.00 |       1 |   1 |         1
 Claude     |       2 |  95000.00 |       2 |   2 |         2
 Tim        |       3 | 110000.00 |       1 |   1 |         1
 Linus      |    NULL | 140000.00 |       1 |   1 |         1
```

* `ROW_NUMBER()`: Monotonically increasing sequential integer ($1, 2, 3, 4$).
* `RANK()`: Ties share equal ranks; skips subsequent ranks ($1, 2, 2, 4$).
* `DENSE_RANK()`: Ties share equal ranks; never skips numbers ($1, 2, 2, 3$).

### 2. Value Offsets: LAG, LEAD, FIRST_VALUE

```sql
SELECT 
    order_id, 
    order_date, 
    total_amount,
    LAG(total_amount, 1, 0.00) OVER (ORDER BY order_date) AS prev_order_val,
    total_amount - LAG(total_amount, 1, total_amount) OVER (ORDER BY order_date) AS revenue_delta,
    LEAD(order_date, 1) OVER (ORDER BY order_date) AS next_order_date
FROM orders;
```

**Output:**
```
 order_id | order_date | total_amount | prev_order_val | revenue_delta | next_order_date 
----------+------------+--------------+----------------+---------------+-----------------
     1001 | 2026-01-10 |      1520.00 |           0.00 |          0.00 | 2026-01-15
     1002 | 2026-01-15 |      1800.00 |        1520.00 |        280.00 | 2026-02-01
     1003 | 2026-02-01 |       200.00 |        1800.00 |      -1600.00 | 2026-02-14
     1004 | 2026-02-14 |      2370.00 |         200.00 |       2170.00 | NULL
```

### 3. Window Frame Definitions (Running vs Moving Aggregations)

```sql
SELECT 
    order_id,
    order_date,
    total_amount,
    -- Running Cumulative Total (From start of history to current row)
    SUM(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total,
    -- Moving 3-Period Average (Previous row + Current row + Next row)
    AVG(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS rolling_3_day_avg
FROM orders;
```

---

## 9. Subqueries & CTEs (Correlated, Non-Correlated, Recursive)

### Correlated Subquery

Runs once for every outer query candidate row.

```sql
-- Find employees whose salary exceeds their department's average
SELECT e1.emp_id, e1.first_name, e1.salary, e1.dept_id
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.dept_id = e1.dept_id
);
```

**Output:**
```
 emp_id | first_name |  salary   | dept_id 
--------+------------+-----------+---------
    101 | Ada        | 150000.00 |       1
    104 | John       | 120000.00 |       2
```

### Common Table Expression (CTE)

Provides clean, reusable modular pipeline blocks:

```sql
WITH DeptSummaries AS (
    SELECT 
        dept_id, 
        AVG(salary) AS avg_dept_sal,
        COUNT(*) AS team_size
    FROM employees
    GROUP BY dept_id
),
TopDepartments AS (
    SELECT dept_id 
    FROM DeptSummaries 
    WHERE avg_dept_sal > 115000
)
SELECT e.emp_id, e.first_name, e.salary, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE e.dept_id IN (SELECT dept_id FROM TopDepartments);
```

### Recursive CTE (Graph Traversal & Hierarchies)

Traverses unbounded hierarchical structures (such as manager $\rightarrow$ report lines).

```sql
WITH RECURSIVE OrgTree AS (
    -- 1. Anchor Member: Find company roots (managers with no manager)
    SELECT 
        emp_id, 
        first_name, 
        manager_id, 
        1 AS depth, 
        CAST(first_name AS VARCHAR(255)) AS management_chain
    FROM employees
    WHERE manager_id IS NULL AND dept_id IS NOT NULL

    UNION ALL

    -- 2. Recursive Member: Join next level child rows to the previous step
    SELECT 
        child.emp_id, 
        child.first_name, 
        child.manager_id, 
        parent.depth + 1,
        CAST(parent.management_chain || ' -> ' || child.first_name AS VARCHAR(255))
    FROM employees child
    INNER JOIN OrgTree parent ON child.manager_id = parent.emp_id
)
SELECT depth, emp_id, management_chain FROM OrgTree ORDER BY depth, emp_id;
```

**Output:**
```
 depth | emp_id |      management_chain       
-------+--------+-----------------------------
     1 |    101 | Ada
     1 |    104 | John
     1 |    106 | Tim
     2 |    102 | Ada -> Alan
     2 |    103 | Ada -> Grace
     2 |    105 | John -> Claude
```

---

## 10. Set Operators: UNION, INTERSECT, EXCEPT

Set operations combine outputs of multiple `SELECT` statements with identical column schemas.

```sql
-- Query A: Engineers
SELECT first_name FROM employees WHERE dept_id = 1
UNION ALL
-- Query B: High Earners (> 125,000)
SELECT first_name FROM employees WHERE salary > 125000;
```

* `UNION`: Combines sets and performs a deduplication sort pass (removes duplicate rows).
* `UNION ALL`: Concatenates datasets directly without deduplication (much faster, lower memory).
* `INTERSECT`: Returns only rows present in both result sets.
* `EXCEPT` (or `MINUS` in Oracle): Returns records present in the first set, excluding any records appearing in the second set.

---

## 11. Views, Materialized Views & Stored Routines

### Standard View vs Materialized View

```sql
-- Standard View: Logical macro recomputed on every execution
CREATE VIEW v_active_engineering_staff AS
SELECT e.emp_id, e.first_name, e.last_name, e.salary
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_name = 'Engineering' AND e.status = 'ACTIVE';

-- Materialized View: Persists physical data snapshot to disk
CREATE MATERIALIZED VIEW mv_department_payroll_snapshot AS
SELECT 
    d.dept_name,
    COUNT(e.emp_id) AS total_staff,
    SUM(e.salary) AS total_spend
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_name;

-- Periodic refresh command
REFRESH MATERIALIZED VIEW mv_department_payroll_snapshot;
```

### User Defined Functions & Triggers

```sql
-- Audit Log Target Table
CREATE TABLE audit_logs (
    log_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    table_name VARCHAR(50),
    action VARCHAR(10),
    row_key INT,
    executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Trigger Function (PostgreSQL Syntax)
CREATE OR REPLACE FUNCTION fn_audit_product_modifications()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_logs (table_name, action, row_key)
    VALUES ('products', TG_OP, NEW.product_id);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger Attachment
CREATE TRIGGER trg_audit_products
AFTER INSERT OR UPDATE ON products
FOR EACH ROW
EXECUTE FUNCTION fn_audit_product_modifications();
```

---

## 12. Indexing Strategies & Query Performance Optimization

### Index Types

```sql
-- 1. Standard B-Tree Index (Default, handles =, <, <=, >, >=, BETWEEN)
CREATE INDEX idx_emp_salary ON employees(salary);

-- 2. Composite Index (Strict Left-Prefix Rule)
CREATE INDEX idx_orders_cust_date ON orders(customer_id, order_date);
-- EFFICIENT:   WHERE customer_id = 501 AND order_date = '2026-01-10'
-- EFFICIENT:   WHERE customer_id = 501
-- INEFFICIENT: WHERE order_date = '2026-01-10' (Skipped the leading indexed column!)

-- 3. Unique Index
CREATE UNIQUE INDEX idx_products_sku ON products(product_name);

-- 4. Partial Index (Index a subset to minimize RAM footprint)
CREATE INDEX idx_active_orders ON orders(order_id) WHERE order_status = 'COMPLETED';

-- 5. Expression / Functional Index
CREATE INDEX idx_lower_lastname ON employees(LOWER(last_name));
-- Optimizes: WHERE LOWER(last_name) = 'lovelace'
```

### Reading EXPLAIN & EXPLAIN ANALYZE

Executing `EXPLAIN ANALYZE` executes the query and reports runtime statistics:

```sql
EXPLAIN ANALYZE 
SELECT * FROM employees WHERE salary > 130000;
```

```
                                                  QUERY PLAN                                                  
--------------------------------------------------------------------------------------------------------------
 Seq Scan on employees  (cost=0.00..1.09 rows=3 width=76) (actual time=0.012..0.015 rows=3 loops=1)
   Filter: (salary > 130000)
   Rows Removed by Filter: 4
 Planning Time: 0.058 ms
 Execution Time: 0.027 ms
```

### Optimization Checklist

1. **SARGability (Search Argument Able):** Avoid applying functions to indexed columns in `WHERE` predicates:
   * **Bad:** `WHERE DATE_PART('year', order_date) = 2026` (invalidates index scan).
   * **Good:** `WHERE order_date >= '2026-01-01' AND order_date < '2027-01-01'`.
2. **Eliminate `SELECT *`:** Only project required columns to maximize **Index-Only Scans** and avoid pulling unused data pages into the buffer pool.
3. **Avoid Cartesian Accidents:** Always verify that $N$ tables in a query have at least $N-1$ join conditions.
4. **Prefer `EXISTS` over `COUNT(*) > 0`:** `EXISTS` short-circuits on the first matching record; `COUNT(*)` scans every candidate row.

---

## 13. Transactions, Isolation Levels & ACID Mechanics

### The ACID Model
* **Atomicity:** All statements inside the transaction block succeed, or all modifications roll back.
* **Consistency:** Database transitions between valid states, never violating schemas, types, or foreign key constraints.
* **Isolation:** Determines the visibility of concurrent uncommitted transactional mutations.
* **Durability:** Committed transactions persist on disk (via Write-Ahead Logging/WAL) through crashes or reboots.

### Transaction Syntax

```sql
BEGIN TRANSACTION; -- or simply BEGIN;

-- Debit Alice
UPDATE accounts 
SET balance = balance - 500.00 
WHERE account_id = 1;

-- Savepoint checkpoint
SAVEPOINT pre_credit;

-- Credit Bob
UPDATE accounts 
SET balance = balance + 500.00 
WHERE account_id = 2;

-- If an error occurred with Bob:
-- ROLLBACK TO SAVEPOINT pre_credit;

COMMIT;
```

### Isolation Levels & Concurrency Phenomena

| Isolation Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads |
| :--- | :--- | :--- | :--- |
| **`READ UNCOMMITTED`** | **Allowed** | **Allowed** | **Allowed** |
| **`READ COMMITTED`** *(Default)* | Prevented | **Allowed** | **Allowed** |
| **`REPEATABLE READ`** | Prevented | Prevented | **Allowed** *(Prevented in Postgres via MVCC)* |
| **`SERIALIZABLE`** | Prevented | Prevented | Prevented |

### Phenomenon Definitions

1. **Dirty Read:** Transaction reads uncommitted, in-flight data written by another concurrent transaction (which might subsequently roll back).
2. **Non-Repeatable Read:** Transaction reads a row, another transaction modifies and commits that row, and the original transaction re-reads it, seeing mutated data.
3. **Phantom Read:** Transaction runs a range query (`WHERE price > 100`), a concurrent transaction inserts and commits a new matching row, and the original transaction re-runs the range query, seeing newly appeared "phantom" rows.

### Setting Isolation Levels

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN;
SELECT * FROM products WHERE stock_qty < 10;
-- Concurrent inserts will not distort repeat evaluations of this query
COMMIT;
```