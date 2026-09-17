# 🐘 PostgreSQL #2: Aggregations, Grouping, JOINS & Relationships
*(Notes from MPrashant PostgreSQL Masterclass - Part 2 from 1:17:00 to 2:22:00)*

> **Video Reference**: [Master POSTGRESQL in ONE VIDEO - MPrashant](https://youtu.be/cnzka7kF5Zk)  
> **Module**: Intermediate SQL Queries, Multi-Table Relational Design & Data Analysis  
> **Coverage**: 1:17:00 to 2:22:00 (Aggregate Functions, GROUP BY, HAVING, Foreign Keys, All 5 JOINS & Set Operations)

---

## 📊 1. Aggregate Functions (Data Summarization)

Aggregate functions multiple rows ke data ko calculate karke ek **single summary value** return karte hain:

| Function | What it does | Example Syntax |
| :--- | :--- | :--- |
| `COUNT()` | Total rows ya non-null values count karta hai | `SELECT COUNT(*) FROM employees;` |
| `SUM()` | Numeric column ka total sum nikalta hai | `SELECT SUM(salary) FROM employees;` |
| `AVG()` | Average (Mean) calculate karta hai | `SELECT AVG(price) FROM products;` |
| `MIN()` | Minimum value find karta hai | `SELECT MIN(age) FROM users;` |
| `MAX()` | Maximum value find karta hai | `SELECT MAX(salary) FROM employees;` |

### ⚡ Pro Tip: `COUNT(*)` vs `COUNT(column_name)`
* `COUNT(*)`: Table ki **saari rows count karta hai** (chahe values NULL ho ya na ho).
* `COUNT(email)`: Sirf un rows ko count karta hai jahan `email` **NULL nahi hai**!

```sql
-- Round off average to 2 decimal places
SELECT 
    COUNT(*) AS total_employees,
    ROUND(AVG(salary), 2) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees;
```

---

## 👥 2. `GROUP BY` & `HAVING` (Data Grouping)

### 📌 `GROUP BY` Clause:
Jab humein category-wise data summarize karna ho (e.g. Har department me kitne log hain? Har category me total sales kitni hui?):

```sql
-- Har Department ka total employee count aur average salary
SELECT 
    department, 
    COUNT(*) AS total_staff, 
    ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY department;
```

---

### 🥊 `WHERE` vs `HAVING` (The Classic Interview Trap!)

Interviewers ka favorite question: *"WHERE aur HAVING me kya farak hai?"*

```text
[ Raw Table Rows ] ──► [ WHERE Filter ] ──► [ GROUP BY ] ──► [ HAVING Filter ] ──► [ Final Output ]
                         (Filters Rows                        (Filters Groups
                          BEFORE grouping)                     AFTER aggregation)
```

* **`WHERE`**: Aggregation hone se **PEHLE** individual rows ko filter karta hai. Isme aggregate functions (`SUM`, `AVG`) use nahi ho sakte!
* **`HAVING`**: Aggregation hone ke **BAAD** groups ko filter karta hai (e.g. Sirf wo departments dikhao jinki average salary ₹50,000 se zyada ho).

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE is_active = TRUE               -- Step 1: Pehle active employees filter karo
GROUP BY department                  -- Step 2: Department-wise group banao
HAVING AVG(salary) > 50000           -- Step 3: Sirf un groups ko rakho jinka avg > 50k
ORDER BY avg_salary DESC;
```

---

## 🔗 3. Database Relationships & Foreign Keys

Relational databases ki taakat unke tables ke beech ke **Relations** me hoti hai:

```text
1. One-to-One (1:1)   ➔ Ek User ka ek hi Profile / Aadhaar card hota hai.
2. One-to-Many (1:N)  ➔ Ek Customer ke multiple Orders ho sakte hain.
3. Many-to-Many (M:N) ➔ Ek Student multiple Courses le sakta hai, aur ek Course me multiple Students ho sakte hain (Requires Junction Table).
```

### 🛡️ Foreign Key Referential Actions:
Jab parent table ki row delete hoti hai, to child data ke sath kya hona chahiye?
* `ON DELETE CASCADE`: Parent user delete hua to uske saare orders bhi auto-delete ho jayein.
* `ON DELETE SET NULL`: Parent delete hone par child ka `user_id` `NULL` ban jaye.
* `ON DELETE RESTRICT` / `NO ACTION`: Jab tak child orders exist karte hain, parent user ko delete hi na karne do (Default safe mode!).

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    order_date DATE DEFAULT CURRENT_DATE,
    amount NUMERIC(10, 2) NOT NULL,
    customer_id INT REFERENCES customers(id) ON DELETE CASCADE
);
```

---

## 🤝 4. SQL JOINS (Visual Master Guide)

JOINS do ya do se zyada tables ko unke common columns ke basis par combine karte hain:

```text
Table A (Customers)                  Table B (Orders)
┌────┬─────────┐                     ┌──────────┬─────────────┬────────┐
│ id │ name    │                     │ order_id │ customer_id │ amount │
├────┼─────────┤                     ├──────────┼─────────────┼────────┤
│ 1  │ Rahul   │                     │ 101      │ 1           │ 500    │
│ 2  │ Sneha   │                     │ 102      │ 1           │ 1200   │
│ 3  │ Amit    │                     │ 103      │ 2           │ 350    │
│ 4  │ Priya   │                     │ 104      │ 99 (Guest)  │ 800    │
└────┴─────────┘                     └──────────┴─────────────┴────────┘
```

---

### 1️⃣ `INNER JOIN` (Only Common Matches)
Sirf wo rows return karta hai jo **dono tables me match** karti hain.
```sql
SELECT customers.name, orders.order_id, orders.amount
FROM customers
INNER JOIN orders ON customers.id = orders.customer_id;
```
* **Result**: Rahul aur Sneha ke orders dikhenge. Amit aur Priya (jinhone order nahi kiya) ignore ho jayenge.

---

### 2️⃣ `LEFT JOIN` (Left Table Sabhi + Right Table Matches)
Left table ke **saare records** aayenge. Agar right table me match nahi mila to right side `NULL` show hogi.
```sql
SELECT customers.name, orders.order_id, orders.amount
FROM customers
LEFT JOIN orders ON customers.id = orders.customer_id;
```
* **Use Case**: *"Un customers ki list nikalo jinhone abhi tak koi order nahi kiya!"*
```sql
SELECT customers.name 
FROM customers
LEFT JOIN orders ON customers.id = orders.customer_id
WHERE orders.order_id IS NULL;
```

---

### 3️⃣ `RIGHT JOIN` (Right Table Sabhi + Left Table Matches)
Right table ke **saare records** aayenge. Agar left customer nahi mila to left side `NULL` hogi (e.g. Guest orders).
```sql
SELECT customers.name, orders.order_id, orders.amount
FROM customers
RIGHT JOIN orders ON customers.id = orders.customer_id;
```

---

### 4️⃣ `FULL OUTER JOIN` (Everything from Both Sides)
Dono tables ka saara data combine karta hai. Jahan match nahi hoga, wahan `NULL` fill ho jayega.
```sql
SELECT customers.name, orders.order_id, orders.amount
FROM customers
FULL OUTER JOIN orders ON customers.id = orders.customer_id;
```

---

### 5️⃣ `CROSS JOIN` (Cartesian Product)
Table A ki har row Table B ki har row ke sath multiply hoti hai ($M \times N$ combinations).
```sql
-- e.g. T-Shirt Sizes (S, M, L) × Colors (Red, Blue)
SELECT sizes.size_name, colors.color_name
FROM sizes
CROSS JOIN colors;
```

---

## ⚡ 5. Real-World Multi-Table Join (3 Tables)

Production me e-commerce query jisme User, Order, aur Product teeno join hote hain:

```sql
SELECT 
    users.name AS customer_name,
    orders.order_id,
    products.title AS product_name,
    order_items.quantity,
    (order_items.quantity * products.price) AS total_price
FROM users
INNER JOIN orders ON users.id = orders.user_id
INNER JOIN order_items ON orders.order_id = order_items.order_id
INNER JOIN products ON order_items.product_id = products.id;
```

---

## 🧮 6. Set Operations (`UNION`, `INTERSECT`, `EXCEPT`)

Multiple `SELECT` queries ke result sets ko mathematically combine karne ke liye:

```text
┌─────────────────┐       ┌─────────────────┐
│     Query A     │       │     Query B     │
│ (Delhi Users)   │       │ (Mumbai Users)  │
└─────────────────┘       └─────────────────┘
         │                         │
         └───────────┬─────────────┘
                     ▼
             [ Set Operations ]
```

### 1️⃣ `UNION` vs `UNION ALL`
* `UNION`: Dono queries ka data merge karta hai aur **duplicate rows ko remove** kar deta hai.
* `UNION ALL`: Dono queries ka data merge karta hai **duplicates ke sath** (Faster because no sorting/deduping!).

```sql
SELECT email FROM delhi_customers
UNION ALL
SELECT email FROM mumbai_customers;
```

### 2️⃣ `INTERSECT`
Sirf wo rows return karta hai jo **dono queries me common** hain.
```sql
-- Wo users jo Website aur Mobile App dono par active hain
SELECT user_id FROM web_logins
INTERSECT
SELECT user_id FROM app_logins;
```

### 3️⃣ `EXCEPT` (MINUS)
Query 1 ki wo rows return karta hai jo **Query 2 me nahi hain**.
```sql
-- Wo users jinhone sign up kiya par abhi tak purchase nahi kiya
SELECT user_id FROM registered_users
EXCEPT
SELECT user_id FROM paying_customers;
```

> ⚠️ **Set Operation Rules**:  
> 1. Dono `SELECT` statements me **same number of columns** hone chahiye.  
> 2. Columns ke **data types compatible** hone chahiye.

---

## 💡 Key Takeaways (Day 2 Summary)

1. **`WHERE` filters rows, `HAVING` filters groups**: Kabhi bhi `WHERE` ke andar `AVG()` ya `COUNT()` mat likhna.
2. **`LEFT JOIN` is the king of finding missing records**: `WHERE right_table.id IS NULL` pattern se orphan records ya non-buying users instantly mil jate hain.
3. **Prefer `UNION ALL` over `UNION` when duplicates don't matter**: `UNION` background me duplicate hatane ke liye expensive sorting karta hai.

