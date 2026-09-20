# 📘 PostgreSQL Day 4 — Views, Procedures, Functions, Window Functions, CTEs & Triggers
> **Video**: [PostgreSQL Masterclass by MPrashant](https://youtu.be/cnzka7kF5Zk?t=13505) | Timestamps: `03:45:00 → 04:47:00 (End of Masterclass 🎉)`

---

## 🗂️ Topics Covered

| # | Topic | Timestamp | Key Concepts |
|---|-------|-----------|--------------|
| 1 | **VIEWS & Materialized Views** | [03:45:05](https://youtu.be/cnzka7kF5Zk?t=13505) | Virtual Tables, Security & Abstraction, `REFRESH MATERIALIZED VIEW` |
| 2 | **Advanced HAVING & Filtering** | [03:48:57](https://youtu.be/cnzka7kF5Zk?t=13737) | Multi-condition aggregate filtering, HAVING vs WHERE deep-dive |
| 3 | **Stored Procedures** | [03:54:00](https://youtu.be/cnzka7kF5Zk?t=14040) | `CREATE PROCEDURE`, `CALL`, Transactions (`COMMIT`/`ROLLBACK`) |
| 4 | **User Defined Functions (UDF)** | [04:03:05](https://youtu.be/cnzka7kF5Zk?t=14585) | `PL/pgSQL`, `RETURNS`, `DECLARE`, `BEGIN...END`, Procedures vs Functions |
| 5 | **WINDOW Functions** | [04:13:39](https://youtu.be/cnzka7kF5Zk?t=15219) | `OVER()`, `PARTITION BY`, `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LEAD()`, `LAG()`, Running Totals |
| 6 | **CTEs (Common Table Expressions)** | [04:27:06](https://youtu.be/cnzka7kF5Zk?t=16026) | `WITH` clause, Modular queries, Recursive CTE for Hierarchy |
| 7 | **TRIGGERS & Trigger Functions** | [04:39:34](https://youtu.be/cnzka7kF5Zk?t=16774) | `BEFORE`/`AFTER`, `FOR EACH ROW`, `NEW`/`OLD` records, Audit Logging System |

---

## 🧪 Base Dataset Setup

> Ek practical table banate hain jo saare advanced topics me use hogi!

```sql
DROP TABLE IF EXISTS sales;
DROP TABLE IF EXISTS staff;

CREATE TABLE staff (
    emp_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    dept VARCHAR(50) NOT NULL,
    salary NUMERIC(10,2) NOT NULL,
    manager_id INT REFERENCES staff(emp_id)
);

CREATE TABLE sales (
    sale_id SERIAL PRIMARY KEY,
    emp_id INT REFERENCES staff(emp_id),
    product_name VARCHAR(100),
    amount NUMERIC(10,2) NOT NULL,
    sale_date DATE NOT NULL
);

-- Insert Staff (with hierarchy: CEO -> Managers -> Associates)
INSERT INTO staff (name, dept, salary, manager_id) VALUES
('Vikram Malhotra', 'Management',  180000.00, NULL), -- 1 (CEO)
('Rohit Sharma',    'Engineering', 120000.00, 1),    -- 2
('Priya Verma',     'Engineering',  95000.00, 2),    -- 3
('Amit Patel',      'Engineering',  85000.00, 2),    -- 4
('Neha Gupta',      'Sales',       105000.00, 1),    -- 5 (Sales Head)
('Karan Singh',     'Sales',        70000.00, 5),    -- 6
('Simran Kaur',     'Sales',        75000.00, 5),    -- 7
('Pooja Joshi',     'HR',           80000.00, 1);    -- 8

-- Insert Sales
INSERT INTO sales (emp_id, product_name, amount, sale_date) VALUES
(5, 'Enterprise Cloud Plan', 50000.00, '2026-01-05'),
(6, 'Pro Subscription',      15000.00, '2026-01-10'),
(6, 'Standard License',       8000.00, '2026-01-15'),
(7, 'Enterprise Cloud Plan', 45000.00, '2026-01-20'),
(5, 'Consulting Services',   30000.00, '2026-02-01'),
(7, 'Pro Subscription',      20000.00, '2026-02-12'),
(6, 'Enterprise Cloud Plan', 52000.00, '2026-02-25');
```

---

## 1️⃣ VIEWS & Materialized Views
> **Timestamp**: [03:45:05](https://youtu.be/cnzka7kF5Zk?t=13505)

> 💡 **Analogy**: View ek **"Saved Bookmark / Virtual Window"** hai. Yeh actual data physically store nahi karti — jab bhi call karo, yeh background me predefined query chala kar fresh data return karti hai.

```
                      ┌─────────────────────────────────┐
                      │          POSTGRES VIEW          │
                      │  (Saved SELECT Query Definition)│
                      └────────────────┬────────────────┘
                                       │ Runs underlying SQL on-the-fly
                     ┌─────────────────┴─────────────────┐
                     ▼                                   ▼
             ┌──────────────┐                    ┌──────────────┐
             │ staff table  │                    │ sales table  │
             └──────────────┘                    └──────────────┘
```

---

### 🛡️ Why Use Views?
1. **Security & Data Masking**: Sensitive columns (like `salary`, `password`, `ssn`) hide kar sakte hain.
2. **Code Simplicity**: Complex 5-table JOINs ko single simple table ki tarah query karne dete hain.
3. **Consistency**: Business metrics (like "active high-value customers") har developer ke liye exact same rahenge.

---

### 💻 1. Standard View Create & Query
```sql
-- View for Public Staff Directory (Salary column hidden)
CREATE OR REPLACE VIEW v_public_staff_directory AS
SELECT 
    emp_id,
    name,
    dept
FROM staff;

-- View se query karo bilkul normal table jaise!
SELECT * FROM v_public_staff_directory WHERE dept = 'Sales';
```

---

### 📊 2. Complex Analytical View
```sql
CREATE OR REPLACE VIEW v_monthly_sales_summary AS
SELECT 
    s.name AS sales_person,
    s.dept,
    COUNT(sa.sale_id) AS total_deals,
    COALESCE(SUM(sa.amount), 0.00) AS total_revenue
FROM staff s
LEFT JOIN sales sa ON s.emp_id = sa.emp_id
WHERE s.dept = 'Sales'
GROUP BY s.name, s.dept;

-- Simple SELECT over the view:
SELECT * FROM v_monthly_sales_summary WHERE total_revenue > 30000;
```

---

### ⚡ Standard View vs MATERIALIZED VIEW

| Feature | Standard View (`VIEW`) | Materialized View (`MATERIALIZED VIEW`) |
|---------|------------------------|------------------------------------------|
| **Data Storage** | ❌ No physical storage (Virtual) | ✅ Physically saved on disk (Cached) |
| **Speed** | Recalculates on every query (slower for huge joins) | ⚡ Blazing fast read speed |
| **Data Freshness** | 100% Real-time fresh | Stale until manually/cron refreshed |
| **Use Case** | Data masking, basic simplifications | Heavy analytical reports, dashboards |

```sql
-- Materialized View create karna
CREATE MATERIALIZED VIEW mv_top_performers AS
SELECT 
    emp_id,
    SUM(amount) AS gross_sales
FROM sales
GROUP BY emp_id;

-- Refresh cache when underlying data changes
REFRESH MATERIALIZED VIEW mv_top_performers;
```

---

## 2️⃣ Advanced HAVING Clause & Group Filtering
> **Timestamp**: [03:48:57](https://youtu.be/cnzka7kF5Zk?t=13737)

```
        ┌─────────────────────────────────────────────────────────┐
        │             WHERE vs HAVING Execution Stage             │
        │                                                         │
        │   FROM staff                                            │
        │      ↓                                                  │
        │   WHERE salary > 70000     ◄── [1. Filters Rows Early]  │
        │      ↓                                                  │
        │   GROUP BY dept                                         │
        │      ↓                                                  │
        │   HAVING COUNT(*) > 1      ◄── [2. Filters Aggregates]  │
        │      ↓                                                  │
        │   SELECT dept, AVG(salary)                              │
        └─────────────────────────────────────────────────────────┘
```

```sql
-- Complex Filter: Departments with > 2 employees having average salary > 75000
SELECT 
    dept,
    COUNT(*) AS total_members,
    ROUND(AVG(salary), 2) AS avg_dept_salary,
    MAX(salary) AS highest_salary
FROM staff
WHERE salary >= 70000               -- Step 1: Filter individual rows
GROUP BY dept                       -- Step 2: Form groups
HAVING COUNT(*) >= 2                -- Step 3: Filter groups by count
   AND AVG(salary) > 80000;         -- Step 4: Filter groups by aggregated average
```

---

## 3️⃣ Stored Procedures in PostgreSQL
> **Timestamp**: [03:54:00](https://youtu.be/cnzka7kF5Zk?t=14040)

> 💡 **What is a Stored Procedure?** Database server pe saved SQL statements ka block jise `CALL` command se execute kiya jaata hai.  
> 🔥 **Super Power**: Procedures can manage **Transactions (`COMMIT` / `ROLLBACK`)** internally!

---

### 💻 Creating and Calling a Stored Procedure

```sql
-- Procedure: Give a percentage bonus increment to a specific department
CREATE OR REPLACE PROCEDURE sp_give_dept_bonus(
    p_dept VARCHAR,
    p_percentage NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Update salaries with validation
    UPDATE staff
    SET salary = salary + (salary * (p_percentage / 100.0))
    WHERE dept = p_dept;

    RAISE NOTICE 'Successfully applied % percent bonus to % department!', p_percentage, p_dept;
END;
$$;

-- Execute Procedure
CALL sp_give_dept_bonus('Engineering', 10.0);
```

---

### 🏧 Procedure with Internal Transaction Management (Bank Transfer Model)
```sql
CREATE OR REPLACE PROCEDURE sp_transfer_funds(
    p_sender_id INT,
    p_receiver_id INT,
    p_amount NUMERIC
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_sender_balance NUMERIC;
BEGIN
    -- Check sender salary/balance
    SELECT salary INTO v_sender_balance FROM staff WHERE emp_id = p_sender_id;

    IF v_sender_balance < p_amount THEN
        RAISE EXCEPTION 'Insufficient balance! Cannot transfer.';
    END IF;

    -- Debit from sender
    UPDATE staff SET salary = salary - p_amount WHERE emp_id = p_sender_id;

    -- Credit to receiver
    UPDATE staff SET salary = salary + p_amount WHERE emp_id = p_receiver_id;

    -- Explicit commit inside procedure (PostgreSQL 11+)
    COMMIT;
    RAISE NOTICE 'Transfer of % completed successfully!', p_amount;
END;
$$;
```

---

## 4️⃣ User Defined Functions (UDF) & PL/pgSQL
> **Timestamp**: [04:03:05](https://youtu.be/cnzka7kF5Zk?t=14585)

> 💡 **UDF**: Ek function jo inputs leta hai, calculation/processing karta hai, aur **value(s) RETURN karta hai**. Isko direct `SELECT` query ke andar expressions me call kar sakte hain.

---

### ⚖️ Stored Procedure vs User-Defined Function (UDF)

| Feature | User-Defined Function (`FUNCTION`) | Stored Procedure (`PROCEDURE`) |
|---------|-----------------------------------|--------------------------------|
| **Execution** | Called inside SQL expressions (`SELECT fn(...)`) | Called standalone (`CALL proc(...)`) |
| **Return Value** | **MUST return** a value (`RETURNS type`) | Does **NOT** directly return via SELECT |
| **Transactions** | Cannot execute `COMMIT` / `ROLLBACK` | Can execute `COMMIT` / `ROLLBACK` |
| **Primary Use** | Calculations, data transformation, validations | Business workflows, batch jobs, bulk updates |

---

### 💻 1. Scalar Function (Returns Single Value)
```sql
-- Function to calculate annual tax slab for an employee
CREATE OR REPLACE FUNCTION fn_calculate_annual_tax(p_salary NUMERIC)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
DECLARE
    v_annual_income NUMERIC;
    v_tax NUMERIC := 0.00;
BEGIN
    v_annual_income := p_salary * 12;

    IF v_annual_income > 1500000 THEN
        v_tax := v_annual_income * 0.30;
    ELSIF v_annual_income > 1000000 THEN
        v_tax := v_annual_income * 0.20;
    ELSE
        v_tax := v_annual_income * 0.10;
    END IF;

    RETURN v_tax;
END;
$$;

-- Calling Function directly in SELECT query:
SELECT 
    name,
    salary AS monthly_salary,
    (salary * 12) AS annual_package,
    fn_calculate_annual_tax(salary) AS annual_tax_deduction
FROM staff;
```

---

### 📋 2. Table-Valued Function (Returns a Virtual Table)
```sql
CREATE OR REPLACE FUNCTION fn_get_dept_staff(p_dept_name VARCHAR)
RETURNS TABLE (
    employee_id INT,
    employee_name VARCHAR,
    monthly_pay NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY 
    SELECT emp_id, name, salary 
    FROM staff 
    WHERE dept = p_dept_name;
END;
$$;

-- Querying Table Function:
SELECT * FROM fn_get_dept_staff('Sales');
```

---

## 5️⃣ WINDOW Functions (The Most Powerful SQL Feature!)
> **Timestamp**: [04:13:39](https://youtu.be/cnzka7kF5Zk?t=15219)

> 💡 **Difference between GROUP BY & WINDOW FUNCTION**:
> - `GROUP BY`: Rows ko compress karke single aggregate row bana deta hai (Individual row identity kho jaati hai).
> - `WINDOW FUNCTION`: Individual row identity **preserve** rehti hai aur saath me aggregate / ranking calculation bhi har row ke saamne add ho jaati hai!

```
                  GROUP BY (Collapses Rows):
     ┌──────────┬────────┐           ┌──────────┬────────┐
     │ Dept     │ Salary │           │ Dept     │ AvgSal │
     ├──────────┼────────┤  ======>  ├──────────┼────────┤
     │ Eng      │  95000 │           │ Eng      │  90000 │
     │ Eng      │  85000 │           └──────────┴────────┘
     └──────────┴────────┘

            WINDOW FUNCTION OVER() (Keeps All Rows):
     ┌──────────┬────────┬─────────────┐
     │ Name     │ Salary │ Dept_AvgSal │  ◄── Calculated across partition
     ├──────────┼────────┼─────────────┤      without collapsing!
     │ Priya    │  95000 │    90000    │
     │ Amit     │  85000 │    90000    │
     └──────────┴────────┴─────────────┘
```

---

### 1. Ranking Functions: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`

```sql
SELECT 
    name,
    dept,
    salary,
    ROW_NUMBER() OVER(PARTITION BY dept ORDER BY salary DESC) AS row_num,
    RANK()       OVER(PARTITION BY dept ORDER BY salary DESC) AS rank_num,
    DENSE_RANK() OVER(PARTITION BY dept ORDER BY salary DESC) AS dense_rank_num
FROM staff;
```

#### 🆚 Ranking Functions Comparison Table:
| Salary Values | `ROW_NUMBER()` | `RANK()` (Skips on Ties) | `DENSE_RANK()` (No Skip) |
|---------------|----------------|--------------------------|--------------------------|
| 100K | 1 | 1 | 1 |
| 90K  | 2 | 2 | 2 |
| 90K  | 3 | 2 *(Tie!)* | 2 *(Tie!)* |
| 80K  | 4 | **4** *(Skipped 3)* | **3** *(Continuous)* |

---

### 2. Value Functions: `LEAD()` and `LAG()`
> Previous row ya Next row ki value current row me dekhne ke liye (Period-over-period growth nikalne ke liye best).

```sql
SELECT 
    sale_id,
    product_name,
    sale_date,
    amount,
    -- Previous sale amount
    LAG(amount, 1) OVER(ORDER BY sale_date) AS prev_sale_amount,
    -- Difference from previous sale
    amount - LAG(amount, 1) OVER(ORDER BY sale_date) AS sale_difference,
    -- Next sale amount
    LEAD(amount, 1) OVER(ORDER BY sale_date) AS next_sale_amount
FROM sales;
```

---

### 3. Running Total (Cumulative Sum) & Department Aggregates
```sql
SELECT 
    sale_id,
    emp_id,
    sale_date,
    amount,
    -- Running total over time
    SUM(amount) OVER(ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_sales,
    -- Average of that specific employee's sales
    ROUND(AVG(amount) OVER(PARTITION BY emp_id), 2) AS emp_avg_sale
FROM sales;
```

---

## 6️⃣ CTE (Common Table Expressions) — `WITH` Clause
> **Timestamp**: [04:27:06](https://youtu.be/cnzka7kF5Zk?t=16026)

> 💡 **What is a CTE?** Ek temporary named result-set jo sirf usi query ke execution time tak exist karta hai. Yeh nested spaghetti subqueries ko clean, readable step-by-step code me convert karta hai.

---

### 🍝 Subquery (Messy) vs 🍱 CTE (Clean)

#### Subquery Approach:
```sql
SELECT * FROM (
    SELECT name, dept, salary,
           DENSE_RANK() OVER(PARTITION BY dept ORDER BY salary DESC) AS rnk
    FROM staff
) sub
WHERE sub.rnk = 1;
```

#### CTE Approach (Clean & Professional):
```sql
WITH RankedEmployees AS (
    SELECT 
        name,
        dept,
        salary,
        DENSE_RANK() OVER(PARTITION BY dept ORDER BY salary DESC) AS dept_rank
    FROM staff
)
SELECT 
    name,
    dept,
    salary
FROM RankedEmployees
WHERE dept_rank = 1; -- Har department ka highest paid person
```

---

### 🌳 Recursive CTE (Hierarchical Organizational Chart)
> Database me Employee -> Manager -> CEO hierarchy ko loop karke tree structure nikalna!

```sql
WITH RECURSIVE OrgHierarchy AS (
    -- Anchor Member: CEO (manager_id is NULL)
    SELECT 
        emp_id,
        name,
        manager_id,
        1 AS hierarchy_level,
        CAST(name AS VARCHAR(255)) AS management_chain
    FROM staff
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive Member: Find all employees reporting to the level above
    SELECT 
        s.emp_id,
        s.name,
        s.manager_id,
        h.hierarchy_level + 1,
        CAST(h.management_chain || ' -> ' || s.name AS VARCHAR(255))
    FROM staff s
    INNER JOIN OrgHierarchy h ON s.manager_id = h.emp_id
)
SELECT 
    emp_id,
    name,
    hierarchy_level,
    management_chain
FROM OrgHierarchy
ORDER BY hierarchy_level, emp_id;
```

```
Result:
┌────────┬─────────────────┬─────────────────┬──────────────────────────────────────────────┐
│ emp_id │ name            │ hierarchy_level │ management_chain                             │
├────────┼─────────────────┼─────────────────┼──────────────────────────────────────────────┤
│      1 │ Vikram Malhotra │               1 │ Vikram Malhotra (CEO)                        │
│      2 │ Rohit Sharma    │               2 │ Vikram Malhotra -> Rohit Sharma              │
│      5 │ Neha Gupta      │               2 │ Vikram Malhotra -> Neha Gupta                │
│      8 │ Pooja Joshi     │               2 │ Vikram Malhotra -> Pooja Joshi               │
│      3 │ Priya Verma     │               3 │ Vikram Malhotra -> Rohit Sharma -> Priya     │
│      4 │ Amit Patel      │               3 │ Vikram Malhotra -> Rohit Sharma -> Amit      │
│      6 │ Karan Singh     │               3 │ Vikram Malhotra -> Neha Gupta -> Karan       │
│      7 │ Simran Kaur     │               3 │ Vikram Malhotra -> Neha Gupta -> Simran      │
└────────┴─────────────────┴─────────────────┴──────────────────────────────────────────────┘
```

---

## 7️⃣ TRIGGERS & Trigger Functions
> **Timestamp**: [04:39:34](https://youtu.be/cnzka7kF5Zk?t=16774)

> 💡 **What is a Trigger?** Ek automated event-listener function jo table pe specific action hone par (`INSERT`, `UPDATE`, ya `DELETE`) automatically execute hota hai.

```
                      TABLE EVENT (INSERT/UPDATE/DELETE)
                                      │
                                      ▼
                        ┌───────────────────────────┐
                        │      TRIGGER FIRES        │
                        │    (BEFORE / AFTER)       │
                        └─────────────┬─────────────┘
                                      │
                                      ▼
                        ┌───────────────────────────┐
                        │ TRIGGER FUNCTION EXECUTES │
                        │  (Accesses NEW & OLD rows)│
                        └───────────────────────────┘
```

---

### 🛡️ Real-World Production Example: Automated Audit Logging System

#### Step 1: Create the Audit Log Table
```sql
CREATE TABLE staff_audit_log (
    audit_id SERIAL PRIMARY KEY,
    emp_id INT,
    action_type VARCHAR(20),       -- 'UPDATE', 'DELETE'
    old_salary NUMERIC(10,2),
    new_salary NUMERIC(10,2),
    changed_by VARCHAR(50) DEFAULT CURRENT_USER,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

#### Step 2: Create Trigger Function (PL/pgSQL)
> PostgreSQL me triggers do hisson me bante hain: (1) Trigger Function, (2) Trigger Definition.

```sql
CREATE OR REPLACE FUNCTION fn_audit_staff_salary_change()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Only log if salary actually changed
    IF OLD.salary IS DISTINCT FROM NEW.salary THEN
        INSERT INTO staff_audit_log (
            emp_id,
            action_type,
            old_salary,
            new_salary,
            changed_by,
            changed_at
        ) VALUES (
            OLD.emp_id,
            'UPDATE_SALARY',
            OLD.salary,
            NEW.salary,
            CURRENT_USER,
            NOW()
        );
    END IF;

    RETURN NEW;
END;
$$;
```

---

#### Step 3: Attach Trigger to Table
```sql
CREATE OR REPLACE TRIGGER trg_staff_salary_audit
AFTER UPDATE ON staff
FOR EACH ROW
EXECUTE FUNCTION fn_audit_staff_salary_change();
```

---

#### 🧪 Test Trigger in Action!
```sql
-- Update Priya's salary from 95000 to 110000
UPDATE staff 
SET salary = 110000.00 
WHERE name = 'Priya Verma';

-- Check Audit Log table
SELECT * FROM staff_audit_log;
```

```
Audit Log Output (Automatically Recorded by Trigger!):
┌──────────┬────────┬───────────────┬────────────┬────────────┬────────────┬─────────────────────┐
│ audit_id │ emp_id │ action_type   │ old_salary │ new_salary │ changed_by │ changed_at          │
├──────────┼────────┼───────────────┼────────────┼────────────┼────────────┼─────────────────────┤
│        1 │      3 │ UPDATE_SALARY │   95000.00 │  110000.00 │ postgres   │ 2026-09-20 18:55:00 │
└──────────┴────────┴───────────────┴────────────┴────────────┴────────────┴─────────────────────┘
```

---

## 🧠 Masterclass Final Summary & Cheatsheet

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       ADVANCED POSTGRESQL CHEATSHEET                        │
├─────────────────────────────────────────────────────────────────────────────┤
│  VIEW              → CREATE VIEW v_name AS SELECT ...                       │
│  MATERIALIZED VIEW → CREATE MATERIALIZED VIEW mv_name AS SELECT ...         │
│                      REFRESH MATERIALIZED VIEW mv_name;                     │
├─────────────────────────────────────────────────────────────────────────────┤
│  PROCEDURE         → CREATE PROCEDURE p(...) AS $$ ... $$; CALL p(...);     │
│                      (Supports COMMIT / ROLLBACK transactions)              │
│  FUNCTION (UDF)    → CREATE FUNCTION f(...) RETURNS type AS $$ ... $$;      │
│                      SELECT f(...); (Used inside queries)                   │
├─────────────────────────────────────────────────────────────────────────────┤
│  WINDOW FUNCTIONS  → func() OVER (PARTITION BY col ORDER BY sort_col)       │
│    - ROW_NUMBER()  → 1, 2, 3, 4 (Unique sequential integer)                 │
│    - RANK()        → 1, 2, 2, 4 (Ties get same rank, skips next)            │
│    - DENSE_RANK()  → 1, 2, 2, 3 (Ties get same rank, NO skipping)          │
│    - LEAD(col, 1)  → Next row's value                                       │
│    - LAG(col, 1)   → Previous row's value                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│  CTE               → WITH cte_name AS (SELECT ...) SELECT * FROM cte_name;  │
│  RECURSIVE CTE     → WITH RECURSIVE cte AS (Anchor UNION ALL Recursive) ... │
├─────────────────────────────────────────────────────────────────────────────┤
│  TRIGGER           → CREATE TRIGGER trg_name                                │
│                      BEFORE/AFTER INSERT/UPDATE/DELETE ON table             │
│                      FOR EACH ROW EXECUTE FUNCTION fn_trigger();            │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

🎉 **Congratulations! You have completed the entire MPrashant PostgreSQL Masterclass (00:00 → 04:47:00)!** 🚀

