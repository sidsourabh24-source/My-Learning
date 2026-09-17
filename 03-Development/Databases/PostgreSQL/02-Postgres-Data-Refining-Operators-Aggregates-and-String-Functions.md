# 📘 PostgreSQL Day 2 — Data Refining, Operators, Aggregates & String Functions
> **Video**: [PostgreSQL Masterclass by MPrashant](https://youtu.be/cnzka7kF5Zk?t=4635) | Timestamps: `1:17:15 → 2:22:00`

---

## 🗂️ Topics Covered

| # | Topic | Timestamp |
|---|-------|-----------|
| 1 | Data Refining — WHERE, ORDER BY, DISTINCT, LIMIT, LIKE |
| 2 | Operators — Relational & Logical | 
| 3 | Aggregate Functions — COUNT, MIN, MAX, AVG, SUM |
| 4 | GROUP BY & HAVING | 
| 5 | String Functions — CONCAT, REPLACE, SUBSTR, etc. | 
| 6 | Exercises & Practice |

---

## 🧪 Practice Dataset — `employees` Table

> Sab examples ek hi table pe karenge — easy rahega track karna!

```sql
CREATE TABLE employees (
    id        SERIAL PRIMARY KEY,
    name      VARCHAR(100),
    dept      VARCHAR(50),
    salary    NUMERIC,
    city      VARCHAR(50),
    joined_on DATE
);

INSERT INTO employees (name, dept, salary, city, joined_on) VALUES
  ('Ravi Sharma',   'Engineering', 85000, 'Delhi',   '2021-03-15'),
  ('Priya Singh',   'Marketing',   60000, 'Mumbai',  '2020-07-22'),
  ('Amit Verma',    'Engineering', 92000, 'Delhi',   '2019-11-01'),
  ('Neha Gupta',    'HR',          45000, 'Pune',    '2022-01-10'),
  ('Rohit Mehta',   'Engineering', 78000, 'Bangalore','2021-06-05'),
  ('Simran Kaur',   'Marketing',   67000, 'Mumbai',  '2020-09-18'),
  ('Karan Patel',   'HR',          48000, 'Delhi',   '2023-02-28'),
  ('Anjali Rao',    'Engineering', 95000, 'Bangalore','2018-04-12'),
  ('Vikram Das',    'Finance',     72000, 'Hyderabad','2022-08-30'),
  ('Pooja Joshi',   'Finance',     69000, 'Pune',    '2021-12-01');
```

---

## 1️⃣ Data Refining — WHERE, ORDER BY, DISTINCT, LIMIT, LIKE

> 💡 **Analogy**: Raw data ek messy wardrobe jaisi hoti hai. Data Refining matlab — filter karo, sort karo, duplicates hatao, aur sirf zaruri cheezein dikhao.

---

### 🔍 WHERE — Rows ko filter karo

```
employees table
┌────┬──────────────┬─────────────┬────────┬──────────┐
│ id │ name         │ dept        │ salary │ city     │
├────┼──────────────┼─────────────┼────────┼──────────┤
│  1 │ Ravi Sharma  │ Engineering │ 85000  │ Delhi    │
│  3 │ Amit Verma   │ Engineering │ 92000  │ Delhi    │  ← WHERE dept = 'Engineering'
│  5 │ Rohit Mehta  │ Engineering │ 78000  │ Bangalore│
│  8 │ Anjali Rao   │ Engineering │ 95000  │ Bangalore│
└────┴──────────────┴─────────────┴────────┴──────────┘
```

```sql
-- Basic filter
SELECT * FROM employees WHERE dept = 'Engineering';

-- Salary filter
SELECT name, salary FROM employees WHERE salary > 80000;

-- Multiple conditions
SELECT name, dept FROM employees WHERE city = 'Delhi' AND dept = 'Engineering';
```

---

### 📋 ORDER BY — Data ko sort karo (ascending / descending)

```sql
-- Salary high to low (descending)
SELECT name, salary FROM employees ORDER BY salary DESC;

-- Name alphabetically (ascending — default)
SELECT name FROM employees ORDER BY name ASC;

-- Multi-column sort: first by dept, then salary high to low
SELECT name, dept, salary
FROM employees
ORDER BY dept ASC, salary DESC;
```

```
Result (sorted by dept ASC, salary DESC):
┌──────────────┬─────────────┬────────┐
│ name         │ dept        │ salary │
├──────────────┼─────────────┼────────┤
│ Anjali Rao   │ Engineering │  95000 │
│ Amit Verma   │ Engineering │  92000 │
│ Ravi Sharma  │ Engineering │  85000 │
│ Rohit Mehta  │ Engineering │  78000 │
│ Vikram Das   │ Finance     │  72000 │
│ Pooja Joshi  │ Finance     │  69000 │
│ Karan Patel  │ HR          │  48000 │
│ Neha Gupta   │ HR          │  45000 │
│ ...          │ ...         │  ...   │
└──────────────┴─────────────┴────────┘
```

---

### 🎯 DISTINCT — Duplicate values hatao

> Problem: Same city multiple rows mein repeat ho rahi hai → `DISTINCT` use karo!

```sql
-- Without DISTINCT — duplicates dikhenge
SELECT city FROM employees;
-- Delhi, Mumbai, Delhi, Pune, Bangalore, Mumbai, Delhi, Bangalore, Hyderabad, Pune

-- With DISTINCT — only unique cities
SELECT DISTINCT city FROM employees;
-- Delhi, Mumbai, Pune, Bangalore, Hyderabad

-- DISTINCT on multiple columns
SELECT DISTINCT dept, city FROM employees;
-- Unique (dept + city) combinations
```

---

### 🔢 LIMIT — Sirf N rows dikhao

```sql
-- Top 3 highest paid employees
SELECT name, salary FROM employees
ORDER BY salary DESC
LIMIT 3;

-- Skip first 3, next 3 dikhao (OFFSET ke saath)
SELECT name, salary FROM employees
ORDER BY salary DESC
LIMIT 3 OFFSET 3;
```

```
LIMIT 3 (Top 3):          LIMIT 3 OFFSET 3 (Next 3):
┌─────────────┬────────┐  ┌─────────────┬────────┐
│ Anjali Rao  │  95000 │  │ Ravi Sharma │  85000 │
│ Amit Verma  │  92000 │  │ Rohit Mehta │  78000 │
│ Ravi Sharma │  85000 │  │ Vikram Das  │  72000 │
└─────────────┴────────┘  └─────────────┴────────┘
```

---

### 🔎 LIKE — Pattern matching (search jaise)

> `%` → koi bhi characters (0 ya zyada)  
> `_` → exactly ek character

```sql
-- Name 'R' se shuru ho
SELECT name FROM employees WHERE name LIKE 'R%';
-- Ravi Sharma, Rohit Mehta

-- Name 'a' pe khatam ho
SELECT name FROM employees WHERE name LIKE '%a';
-- Neha Gupta → nahi, Anjali Rao → nahi, Pooja Joshi → nahi
-- (try: '%i' → Rohit Mehta, Priya Singh)

-- City mein 'a' kahi bhi ho
SELECT name, city FROM employees WHERE city LIKE '%a%';
-- Bangalore, Hyderabad

-- Exactly 4 characters wali city
SELECT city FROM employees WHERE city LIKE '____';
-- Pune (4 chars)

-- ILIKE = case-insensitive LIKE (PostgreSQL specific!)
SELECT name FROM employees WHERE name ILIKE 'r%';
-- Ravi Sharma, Rohit Mehta (capital/small dono chalega)
```

---

## 2️⃣ Operators — Relational & Logical

---

### ⚖️ Relational Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=`  | Equal | `salary = 85000` |
| `!=` or `<>` | Not equal | `dept != 'HR'` |
| `>`  | Greater than | `salary > 70000` |
| `<`  | Less than | `salary < 70000` |
| `>=` | Greater than or equal | `salary >= 78000` |
| `<=` | Less than or equal | `salary <= 60000` |

```sql
-- Greater than
SELECT name, salary FROM employees WHERE salary > 80000;

-- Not equal
SELECT name, dept FROM employees WHERE dept <> 'HR';

-- Range check
SELECT name, salary FROM employees WHERE salary >= 60000 AND salary <= 80000;
```

---

### 🧠 Logical Operators

#### AND — Dono conditions true honi chahiye

```sql
-- Engineering department AND salary > 80000
SELECT name, dept, salary
FROM employees
WHERE dept = 'Engineering' AND salary > 80000;
-- Ravi Sharma (85000), Amit Verma (92000), Anjali Rao (95000)
```

#### OR — Koi ek condition true ho

```sql
-- Delhi ya Mumbai mein ho
SELECT name, city
FROM employees
WHERE city = 'Delhi' OR city = 'Mumbai';
```

#### NOT — Condition ulti kar do

```sql
-- Engineering ke alawa sab
SELECT name, dept FROM employees WHERE NOT dept = 'Engineering';

-- 'Delhi' wale nahi
SELECT name, city FROM employees WHERE NOT city = 'Delhi';
```

#### BETWEEN — Range check (inclusive)

```sql
-- Salary between 60000 and 80000 (dono included)
SELECT name, salary
FROM employees
WHERE salary BETWEEN 60000 AND 80000;
-- Priya Singh (60000), Rohit Mehta (78000), Simran Kaur (67000),
-- Vikram Das (72000), Pooja Joshi (69000)
```

> ⚡ `BETWEEN 60000 AND 80000` = `>= 60000 AND <= 80000`

#### IN — Multiple values check (OR ka shortcut)

```sql
-- City Delhi, Mumbai, ya Pune mein ho
SELECT name, city
FROM employees
WHERE city IN ('Delhi', 'Mumbai', 'Pune');

-- Department Engineering ya Finance ho
SELECT name, dept
FROM employees
WHERE dept IN ('Engineering', 'Finance');
```

#### IS NULL / IS NOT NULL — NULL values check

```sql
-- NULL salary wale employees
SELECT name FROM employees WHERE salary IS NULL;

-- NULL nahi, salary defined hai
SELECT name, salary FROM employees WHERE salary IS NOT NULL;
```

> ⚠️ **Note**: `salary = NULL` kaam NAHI karta PostgreSQL mein!  
> Always `IS NULL` ya `IS NOT NULL` use karo.

---

### 🔄 Operator Precedence (Priority Order)

```
NOT  →  AND  →  OR
(highest)      (lowest)
```

```sql
-- Without parentheses (NOT pehle evaluate hoga)
SELECT * FROM employees
WHERE NOT dept = 'HR' AND salary > 70000;
-- = (dept != 'HR') AND (salary > 70000)

-- OR ke saath parentheses important hai!
SELECT * FROM employees
WHERE dept = 'HR' OR dept = 'Finance' AND salary > 70000;
-- AND pehle → dept='Finance' AND salary>70000 ... phir OR dept='HR'

-- Sahi way — parentheses lagao
SELECT * FROM employees
WHERE (dept = 'HR' OR dept = 'Finance') AND salary > 70000;
```

---

## 3️⃣ Aggregate Functions — COUNT, MIN, MAX, AVG, SUM

> 💡 **Analogy**: Aggregate functions matlab — puri class ke marks leke ek summary nikalna (total students, min marks, max marks, average).

```
employees table (10 rows)
         ↓ Aggregate Function apply karo
    ek single value milegi!
```

---

### 📊 All Aggregate Functions

```sql
-- COUNT: Kitne rows hain?
SELECT COUNT(*) FROM employees;             -- 10
SELECT COUNT(salary) FROM employees;        -- NULL values count nahi hoti
SELECT COUNT(DISTINCT dept) FROM employees; -- Unique departments: 4

-- SUM: Total salary?
SELECT SUM(salary) FROM employees;          -- 711000

-- AVG: Average salary?
SELECT AVG(salary) FROM employees;          -- 71100.00

-- MIN: Minimum salary?
SELECT MIN(salary) FROM employees;          -- 45000

-- MAX: Maximum salary?
SELECT MAX(salary) FROM employees;          -- 95000
```

---

### 🎨 Alias ke saath — Clean output

```sql
SELECT
    COUNT(*)        AS total_employees,
    SUM(salary)     AS total_salary,
    ROUND(AVG(salary), 2) AS avg_salary,
    MIN(salary)     AS min_salary,
    MAX(salary)     AS max_salary
FROM employees;
```

```
Output:
┌──────────────────┬──────────────┬────────────┬────────────┬────────────┐
│ total_employees  │ total_salary │ avg_salary │ min_salary │ max_salary │
├──────────────────┼──────────────┼────────────┼────────────┼────────────┤
│        10        │   711000     │  71100.00  │   45000    │   95000    │
└──────────────────┴──────────────┴────────────┴────────────┴────────────┘
```

---

### ⚠️ Aggregate + Non-Aggregate = Error!

```sql
-- ❌ WRONG — yeh error dega!
SELECT name, AVG(salary) FROM employees;
-- ERROR: "name" must appear in GROUP BY or aggregate function

-- ✅ CORRECT — alag alag
SELECT AVG(salary) FROM employees;

-- ✅ CORRECT — WHERE ke saath filter karke
SELECT AVG(salary) FROM employees WHERE dept = 'Engineering';
-- Avg of Engineering dept only → 87500
```

---

## 4️⃣ GROUP BY & HAVING

> 💡 **Analogy**: `GROUP BY` = classroom mein students ko sections mein baant do (A, B, C), phir har section ka average nikalo!

---

### 📦 GROUP BY — Rows ko groups mein baanto

```
employees (10 rows)
        ↓ GROUP BY dept
Engineering → [Ravi, Amit, Rohit, Anjali]   → AVG = 87500
Marketing   → [Priya, Simran]               → AVG = 63500
HR          → [Neha, Karan]                 → AVG = 46500
Finance     → [Vikram, Pooja]               → AVG = 70500
```

```sql
-- Har department ka average salary
SELECT dept, AVG(salary) AS avg_salary
FROM employees
GROUP BY dept;

-- Har department mein kitne employees hain
SELECT dept, COUNT(*) AS headcount
FROM employees
GROUP BY dept;

-- Har city ka total salary bill
SELECT city, SUM(salary) AS total_cost
FROM employees
GROUP BY city
ORDER BY total_cost DESC;
```

```
Output (dept + headcount):
┌─────────────┬───────────┐
│ dept        │ headcount │
├─────────────┼───────────┤
│ Engineering │     4     │
│ Marketing   │     2     │
│ HR          │     2     │
│ Finance     │     2     │
└─────────────┴───────────┘
```

---

### 🔽 HAVING — GROUP BY ke baad filter karo

> `WHERE` rows filter karta hai (before grouping)  
> `HAVING` groups filter karta hai (after grouping)

```
                Query Execution Order:
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

```sql
-- Departments jahan average salary > 65000 ho
SELECT dept, AVG(salary) AS avg_salary
FROM employees
GROUP BY dept
HAVING AVG(salary) > 65000;
-- Engineering (87500), Marketing (63500 ❌), Finance (70500), HR (46500 ❌)
-- Output: Engineering, Finance

-- Departments jahan 2 se zyada employees hain
SELECT dept, COUNT(*) AS headcount
FROM employees
GROUP BY dept
HAVING COUNT(*) > 2;
-- Only Engineering (4 employees)

-- WHERE + GROUP BY + HAVING combo
SELECT dept, AVG(salary) AS avg_salary
FROM employees
WHERE city != 'Delhi'           -- Pehle Delhi wale hata do (WHERE)
GROUP BY dept                   -- Phir group karo
HAVING AVG(salary) > 65000;    -- Phir groups filter karo (HAVING)
```

---

### 🆚 WHERE vs HAVING — Clear Difference

```sql
-- WHERE: Individual rows filter (aggregation se PEHLE)
SELECT dept, COUNT(*) FROM employees
WHERE salary > 60000            -- ← rows filter
GROUP BY dept;

-- HAVING: Groups filter (aggregation ke BAAD)
SELECT dept, COUNT(*) FROM employees
GROUP BY dept
HAVING COUNT(*) > 1;            -- ← groups filter
```

| Feature | WHERE | HAVING |
|---------|-------|--------|
| Works on | Individual rows | Groups |
| Runs | Before GROUP BY | After GROUP BY |
| Can use aggregate? | ❌ No | ✅ Yes |
| Example | `WHERE salary > 50000` | `HAVING AVG(salary) > 50000` |

---

## 5️⃣ String Functions

> 💡 **Analogy**: String functions = text pe tools — uppercase karo, kaat do, replace karo, dhundo!

---

### 🔗 CONCAT — Strings join karo

```sql
-- Basic concat
SELECT CONCAT('Hello', ' ', 'World');
-- Hello World

-- Column values join karo
SELECT CONCAT(name, ' works in ', dept) AS description
FROM employees;
-- "Ravi Sharma works in Engineering"

-- || operator bhi kaam karta hai (PostgreSQL)
SELECT name || ' - ' || city AS name_city
FROM employees;
-- "Ravi Sharma - Delhi"

-- CONCAT_WS (with separator) — separator ek baar likhna padta hai
SELECT CONCAT_WS(', ', name, dept, city) AS info
FROM employees;
-- "Ravi Sharma, Engineering, Delhi"
```

---

### 🔤 UPPER & LOWER — Case change karo

```sql
SELECT UPPER('hello postgresql');    -- HELLO POSTGRESQL
SELECT LOWER('RAVI SHARMA');         -- ravi sharma

-- Table pe apply karo
SELECT UPPER(name) AS name_upper FROM employees;
SELECT LOWER(dept) AS dept_lower FROM employees;

-- Use case: Case-insensitive search
SELECT * FROM employees WHERE LOWER(name) = 'ravi sharma';
```

---

### 📏 LENGTH — String ki length nikalo

```sql
SELECT LENGTH('PostgreSQL');         -- 10
SELECT LENGTH('');                   -- 0

-- Har employee ke naam ki length
SELECT name, LENGTH(name) AS name_length
FROM employees
ORDER BY name_length DESC;

-- Names jiske 10 ya zyada characters hain
SELECT name FROM employees WHERE LENGTH(name) >= 10;
```

---

### ✂️ SUBSTRING / SUBSTR — Part of string nikalo

```sql
-- SUBSTRING(string FROM start FOR length)
SELECT SUBSTRING('PostgreSQL' FROM 1 FOR 4);    -- Post
SELECT SUBSTRING('PostgreSQL' FROM 5);          -- greSQL (end tak)

-- SUBSTR(string, start, length) — same cheez
SELECT SUBSTR('Hello World', 7, 5);             -- World

-- Naam ka first name nikalo (pehle 4 letters)
SELECT SUBSTR(name, 1, 4) AS short_name FROM employees;
-- "Ravi", "Priy", "Amit"...
```

---

### 🔄 REPLACE — Text replace karo

```sql
SELECT REPLACE('Hello World', 'World', 'PostgreSQL');
-- Hello PostgreSQL

-- Table mein: 'Engineering' ko 'Eng' se replace karo
SELECT name, REPLACE(dept, 'Engineering', 'Eng') AS dept_short
FROM employees;

-- Special character remove karo
SELECT REPLACE('9876-5432-10', '-', '') AS phone_clean;
-- 9876543210
```

---

### 🔍 POSITION — String ke andar kuch dhundo

```sql
SELECT POSITION('SQL' IN 'PostgreSQL');    -- 9 (9th position se shuru)
SELECT POSITION('xyz' IN 'Hello');         -- 0 (not found)

-- '@' ka position email mein
SELECT POSITION('@' IN 'ravi@company.com');    -- 5
```

---

### 🧹 TRIM, LTRIM, RTRIM — Spaces hatao

```sql
SELECT TRIM('   hello   ');          -- 'hello' (dono side ke spaces)
SELECT LTRIM('   hello   ');         -- 'hello   ' (left side only)
SELECT RTRIM('   hello   ');         -- '   hello' (right side only)

-- Specific character trim karo
SELECT TRIM('x' FROM 'xxxHelloxxxx');    -- 'Hello'

-- Use case: User input clean karna
INSERT INTO employees (name, dept, salary, city, joined_on)
VALUES (TRIM('  Rahul Sinha  '), 'Marketing', 55000, 'Pune', '2024-01-01');
```

---

### 🔢 LPAD & RPAD — Padding add karo

```sql
SELECT LPAD('42', 5, '0');     -- 00042 (left padded with zeros)
SELECT RPAD('Hi', 5, '!');     -- Hi!!! (right padded with !)

-- Employee IDs zero-padded
SELECT LPAD(CAST(id AS TEXT), 4, '0') AS emp_id FROM employees;
-- 0001, 0002, 0003...
```

---

### 🔀 REVERSE — String ulta karo

```sql
SELECT REVERSE('PostgreSQL');    -- LQSergatsoP
SELECT REVERSE('madam');         -- madam (palindrome!)
```
---

### 📋 String Functions Quick Reference

```sql
-- All in one example
SELECT
    name,
    UPPER(name)                          AS upper_name,
    LOWER(name)                          AS lower_name,
    LENGTH(name)                         AS name_len,
    SUBSTR(name, 1, POSITION(' ' IN name) - 1) AS first_name,
    CONCAT(name, ' | ', dept)            AS full_info,
    REPLACE(dept, 'Engineering', 'Tech') AS dept_alias,
    TRIM(city)                           AS city_clean
FROM employees;
```

---

## 6️⃣ Exercises & Practice

> Try these on the `employees` table — pehle khud karo, phir answer dekho! 💪

---

### 🎯 Exercise Set 1 — Data Refining

```sql
-- Q1: Find all employees from 'Bangalore'
-- Q2: List employees with salary > 75000, sorted high to low
-- Q3: Show distinct departments in the company
-- Q4: Show top 5 highest paid employees
-- Q5: Find employees whose name starts with 'A'
-- Q6: Find employees whose city contains 'bad' (like Hyderabad)
```

<details>
<summary>✅ Answers — Click to reveal</summary>

```sql
-- A1:
SELECT * FROM employees WHERE city = 'Bangalore';
-- A2:
SELECT name, salary FROM employees WHERE salary > 75000 ORDER BY salary DESC;
-- A3:
SELECT DISTINCT dept FROM employees;
-- A4:
SELECT name, salary FROM employees ORDER BY salary DESC LIMIT 5;
-- A5:
SELECT name FROM employees WHERE name LIKE 'A%';
-- A6:
SELECT name, city FROM employees WHERE city LIKE '%bad%';
```

</details>

---

### 🎯 Exercise Set 2 — Operators

```sql
-- Q1: Find employees with salary BETWEEN 65000 AND 90000
-- Q2: Find employees NOT in 'Delhi' or 'Mumbai'
-- Q3: Find employees in HR or Marketing department (use IN)
-- Q4: List employees joined after '2021-01-01'
-- Q5: Employees where salary is NOT NULL (all, since we set values)
```

<details>
<summary>✅ Answers — Click to reveal</summary>

```sql
-- A1:
SELECT name, salary FROM employees WHERE salary BETWEEN 65000 AND 90000;
-- A2:
SELECT name, city FROM employees WHERE city NOT IN ('Delhi', 'Mumbai');
-- A3:
SELECT name, dept FROM employees WHERE dept IN ('HR', 'Marketing');
-- A4:
SELECT name, joined_on FROM employees WHERE joined_on > '2021-01-01';
-- A5:
SELECT name FROM employees WHERE salary IS NOT NULL;
```

</details>

---

### 🎯 Exercise Set 3 — Aggregates & GROUP BY

```sql
-- Q1: Total number of employees
-- Q2: Average salary of Marketing department
-- Q3: Highest salary in each department
-- Q4: Departments with more than 1 employee
-- Q5: City-wise total salary, only cities where total > 100000
-- Q6: Count of employees per city, sorted by count descending
```

<details>
<summary>✅ Answers — Click to reveal</summary>

```sql
-- A1:
SELECT COUNT(*) AS total FROM employees;
-- A2:
SELECT AVG(salary) FROM employees WHERE dept = 'Marketing';
-- A3:
SELECT dept, MAX(salary) AS highest_salary FROM employees GROUP BY dept;
-- A4:
SELECT dept, COUNT(*) AS cnt FROM employees GROUP BY dept HAVING COUNT(*) > 1;

-- A5:
SELECT city, SUM(salary) AS total_sal
FROM employees
GROUP BY city
HAVING SUM(salary) > 100000;

-- A6:
SELECT city, COUNT(*) AS cnt
FROM employees
GROUP BY city
ORDER BY cnt DESC;
```

</details>

---

### 🎯 Exercise Set 4 — String Functions

```sql
-- Q1: Display all names in UPPERCASE
-- Q2: Find length of each employee's name, show top 3 longest names
-- Q3: Extract first name of every employee (before the space)
-- Q4: Replace 'HR' with 'Human Resources' in dept column output
-- Q5: Concatenate name + ' from ' + city for all employees
-- Q6: Find employees whose name has 'ra' anywhere (case-insensitive)
```

<details>
<summary>✅ Answers — Click to reveal</summary>

```sql
-- A1:
SELECT UPPER(name) FROM employees;
-- A2:
SELECT name, LENGTH(name) AS len FROM employees ORDER BY len DESC LIMIT 3;
-- A3:
SELECT SUBSTR(name, 1, POSITION(' ' IN name) - 1) AS first_name FROM employees;
-- A4:
SELECT name, REPLACE(dept, 'HR', 'Human Resources') AS dept_full FROM employees;
-- A5:
SELECT CONCAT(name, ' from ', city) AS intro FROM employees;
-- A6:
SELECT name FROM employees WHERE name ILIKE '%ra%';
```

</details>

---

## 🧠 Quick Revision Cheatsheet

```
┌─────────────────────────────────────────────────────────────┐
│                    DATA REFINING                            │
│  WHERE    → rows filter karo                                │
│  ORDER BY → sort karo (ASC/DESC)                            │
│  DISTINCT → duplicates hatao                                │
│  LIMIT    → sirf N rows dikhao                              │
│  LIKE     → pattern match (% = any chars, _ = one char)     │
├─────────────────────────────────────────────────────────────┤
│                    OPERATORS                                │
│  =, !=, >, <, >=, <=   → Relational                        │
│  AND, OR, NOT           → Logical                           │
│  BETWEEN x AND y        → Range (inclusive)                 │
│  IN (a, b, c)           → Multiple values                   │
│  IS NULL / IS NOT NULL  → NULL check                        │
├─────────────────────────────────────────────────────────────┤
│                 AGGREGATE FUNCTIONS                         │
│  COUNT(*) → total rows                                      │
│  SUM()    → total of values                                 │
│  AVG()    → average                                         │
│  MIN()    → smallest value                                  │
│  MAX()    → largest value                                   │
├─────────────────────────────────────────────────────────────┤
│                   GROUP BY & HAVING                         │
│  GROUP BY → rows ko groups mein baanto                      │
│  HAVING   → groups filter karo (aggregate ke baad)          │
│  WHERE    → individual rows filter (aggregate se pehle)     │
├─────────────────────────────────────────────────────────────┤
│                   STRING FUNCTIONS                          │
│  CONCAT(a,b) / a||b  → join strings                        │
│  UPPER() / LOWER()   → case change                          │
│  LENGTH()            → string length                        │
│  SUBSTR(s, start, n) → part of string                       │
│  REPLACE(s, old, new)→ text replace                         │
│  POSITION(x IN s)    → find position                        │
│  TRIM()              → remove spaces                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔄 Query Execution Order (Important!)

```
FROM        → table select karo
WHERE       → rows filter karo (before grouping)
GROUP BY    → groups banao
HAVING      → groups filter karo (after grouping)
SELECT      → columns choose karo
ORDER BY    → result sort karo
LIMIT       → rows limit karo
```

---

> **Next**: Day 3 — JOINs (INNER, LEFT, RIGHT, FULL), Subqueries 🔗

