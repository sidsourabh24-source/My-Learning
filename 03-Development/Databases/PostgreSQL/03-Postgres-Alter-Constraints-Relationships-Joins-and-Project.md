# 📘 PostgreSQL Day 3 — ALTER Table, Advanced Constraints, Relationships, JOINs & E-Store Project
> **Video**: [PostgreSQL Masterclass by MPrashant](https://youtu.be/cnzka7kF5Zk?t=8565) | Timestamps: `02:22:00 → 03:45:00`

---

## 🗂️ Topics Covered

| # | Topic | Timestamp | Key Concepts |
|---|-------|-----------|--------------|
| 1 | **ALTER Query — Modify Table** | [02:22:45](https://youtu.be/cnzka7kF5Zk?t=8565) | `ADD`, `DROP`, `RENAME COLUMN/TABLE`, `ALTER TYPE`, `SET/DROP DEFAULT` |
| 2 | **Advanced Constraints & CHECK** | [02:30:46](https://youtu.be/cnzka7kF5Zk?t=9046) | `CHECK`, `UNIQUE`, Named Constraints, Validation Rules |
| 3 | **Database Relationships** | [02:46:41](https://youtu.be/cnzka7kF5Zk?t=10001) | 1:1 (One-to-One), 1:N (One-to-Many), M:N (Many-to-Many) |
| 4 | **Foreign Key & Referential Integrity** | [02:54:48](https://youtu.be/cnzka7kF5Zk?t=10488) | `REFERENCES`, `ON DELETE CASCADE`, `ON DELETE SET NULL`, `RESTRICT` |
| 5 | **SQL JOINs (All Types)** | [03:00:41](https://youtu.be/cnzka7kF5Zk?t=10841) | `INNER`, `LEFT`, `RIGHT`, `FULL OUTER`, `CROSS JOIN`, `ON` vs `WHERE` |
| 6 | **Many-to-Many & Junction Table** | [03:12:25](https://youtu.be/cnzka7kF5Zk?t=11545) | Bridge / Pivot Table, Composite Primary Keys |
| 7 | **Hands-On Project: E-STORE Database** | [03:25:42](https://youtu.be/cnzka7kF5Zk?t=12342) | Real-world E-Commerce Schema, Multi-table JOIN Queries & Analytics |

---

## 1️⃣ ALTER Query — Modify Existing Table Structure
> **Timestamp**: [02:22:45](https://youtu.be/cnzka7kF5Zk?t=8565)

> 💡 **Analogy**: Jab ek house ban jaata hai aur baad mein ek extra room add karna ho, ya wall ka color change karna ho — tab `ALTER TABLE` use hota hai bina pura ghar tode (`DROP` kiye).

```
          ┌───────────────────────────────────────────────┐
          │                  ALTER TABLE                  │
          ├──────────────┬────────────────┬───────────────┤
          │  ADD Column  │  DROP Column   │ RENAME Col/Tab│
          ├──────────────┼────────────────┼───────────────┤
          │  ALTER Type  │ SET/DROP Dflt  │ ADD Constraint│
          └──────────────┴────────────────┴───────────────┘
```

### 🧪 Base Table Setup
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

INSERT INTO users (name, email) VALUES
('Sourabh', 'sourabh@gmail.com'),
('Aman', 'aman@yahoo.com');
```

---

### ➕ 1. Add New Column (`ADD COLUMN`)
```sql
-- Syntax: ALTER TABLE table_name ADD COLUMN column_name datatype [constraint];
ALTER TABLE users ADD COLUMN age INT;
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT true;
```

---

### ➖ 2. Delete a Column (`DROP COLUMN`)
```sql
-- Syntax: ALTER TABLE table_name DROP COLUMN column_name;
ALTER TABLE users DROP COLUMN age;

-- Agar column pe koi view ya dependency ho toh CASCADE use karo:
ALTER TABLE users DROP COLUMN age CASCADE;
```

---

### ✏️ 3. Rename Column & Rename Table
```sql
-- Column Rename
ALTER TABLE users RENAME COLUMN name TO full_name;

-- Table Rename
ALTER TABLE users RENAME TO app_users;
```

---

### 🔄 4. Modify Column Data Type (`ALTER COLUMN ... TYPE`)
```sql
-- full_name ko VARCHAR(50) se badha kar VARCHAR(150) karna
ALTER TABLE app_users ALTER COLUMN full_name TYPE VARCHAR(150);

-- Data type convert karna (e.g. TEXT to INT using USING clause)
-- ALTER TABLE table_name ALTER COLUMN col_name TYPE INT USING col_name::integer;
```

---

### ⚙️ 5. Set or Drop Default Values
```sql
-- Default value set karo
ALTER TABLE app_users ALTER COLUMN is_active SET DEFAULT false;

-- Default value hatao
ALTER TABLE app_users ALTER COLUMN is_active DROP DEFAULT;
```

---

### 🔒 6. Add & Drop NOT NULL Constraint via ALTER
```sql
-- Column ko NOT NULL banao
ALTER TABLE app_users ALTER COLUMN email SET NOT NULL;

-- NOT NULL constraint hatao
ALTER TABLE app_users ALTER COLUMN email DROP NOT NULL;
```

---

## 2️⃣ Advanced Constraints & CHECK Constraint
> **Timestamp**: [02:30:46](https://youtu.be/cnzka7kF5Zk?t=9046)

> 💡 **CHECK Constraint**: Table level pe custom validation lagata hai ki entered value valid business rule satisfy karti hai ya nahi.

---

### 🛡️ 1. Table Creation ke time CHECK lagana
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2) CHECK (price > 0),             -- Price must be positive
    discount_price NUMERIC(10, 2),
    stock INT CHECK (stock >= 0),                       -- Stock cannot be negative
    rating NUMERIC(2,1) CHECK (rating BETWEEN 1 AND 5), -- 1.0 to 5.0 only
    CONSTRAINT chk_discount CHECK (discount_price < price) -- Multi-column check rule
);
```

---

### 🧪 Test Validations:
```sql
-- ✅ Success: valid row
INSERT INTO products (name, price, discount_price, stock, rating)
VALUES ('Wireless Mouse', 1200.00, 999.00, 50, 4.5);

-- ❌ Error: price <= 0 violates CHECK constraint
INSERT INTO products (name, price, discount_price, stock, rating)
VALUES ('Free Sample', -10.00, 0, 10, 5.0);
-- ERROR: new row for relation "products" violates check constraint "products_price_check"

-- ❌ Error: discount_price >= price violates chk_discount
INSERT INTO products (name, price, discount_price, stock, rating)
VALUES ('Gaming Keyboard', 2000.00, 2500.00, 10, 4.0);
-- ERROR: violates check constraint "chk_discount"
```

---

### ➕ 2. Existing Table pe CHECK Constraint Add karna (via ALTER)
```sql
-- Syntax: ALTER TABLE table_name ADD CONSTRAINT constraint_name CHECK (condition);
ALTER TABLE app_users ADD CONSTRAINT chk_age CHECK (age >= 18);

-- Constraint ko Drop karna
ALTER TABLE app_users DROP CONSTRAINT chk_age;
```

---

## 3️⃣ Database Relationships & Types
> **Timestamp**: [02:46:41](https://youtu.be/cnzka7kF5Zk?t=10001)

> 💡 **Why Relationships?** Normalization ke through data redundancy (duplicate data) prevent hoti hai. Data alag tables me logically store hota hai aur Keys se connect hota hai.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            RELATIONSHIP TYPES                               │
├───────────────────┬───────────────────────────────┬─────────────────────────┤
│ 1:1 (One-to-One)  │ Ek row sirf 1 row se link     │ User ↔ User Profile     │
├───────────────────┼───────────────────────────────┼─────────────────────────┤
│ 1:N (One-to-Many) │ Ek row multiple rows se link  │ Customer ↔ Orders       │
├───────────────────┼───────────────────────────────┼─────────────────────────┤
│ M:N (Many-to-Many)│ Multiple rows ↔ Multiple rows │ Students ↔ Courses      │
└───────────────────┴───────────────────────────────┴─────────────────────────┘
```

---

## 4️⃣ Foreign Key & Referential Integrity
> **Timestamp**: [02:54:48](https://youtu.be/cnzka7kF5Zk?t=10488)

> 💡 **Foreign Key**: Ek table ka column jo dusri table ke `PRIMARY KEY` ko point karta hai. Yeh ensure karta hai ki koi invalid/ghost data insert na ho sake.

```
┌─────────────────────────┐                 ┌─────────────────────────┐
│     CUSTOMERS (Parent)  │                 │      ORDERS (Child)     │
├─────────────────────────┤                 ├─────────────────────────┤
│ customer_id (PK) ◄──────┼─────────────────┼── customer_id (FK)      │
│ name                    │     1 : N       │   order_id (PK)         │
│ email                   │                 │   amount                │
└─────────────────────────┘                 └─────────────────────────┘
```

---

### 🔑 1. One-to-One (1:1) Relationship Example
> User aur uska Aadhaar/Passport Card (Ek user ka 1 hi passport hoga, aur 1 passport 1 hi user ka hoga).  
> **Key**: Foreign key column pe `UNIQUE` constraint lagana zaroori hai!

```sql
CREATE TABLE users_1to1 (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL
);

CREATE TABLE passports (
    passport_id SERIAL PRIMARY KEY,
    passport_number VARCHAR(20) UNIQUE NOT NULL,
    user_id INT UNIQUE, -- UNIQUE ensures 1:1 relationship!
    FOREIGN KEY (user_id) REFERENCES users_1to1(user_id) ON DELETE CASCADE
);
```

---

### 🔑 2. One-to-Many (1:N) Relationship Example
> Ek Customer ke multiple Orders ho sakte hain, lekin har Order ka sirf 1 Customer hota hai.

```sql
-- Parent Table
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    city VARCHAR(50)
);

-- Child Table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    order_date DATE DEFAULT CURRENT_DATE,
    amount NUMERIC(10,2) NOT NULL,
    customer_id INT,
    CONSTRAINT fk_customer 
        FOREIGN KEY (customer_id) 
        REFERENCES customers(customer_id)
        ON DELETE CASCADE
);
```

---

### ⚡ `ON DELETE` Actions Cheat-Sheet

| ON DELETE Action | What Happens when Parent Row is Deleted? |
|------------------|------------------------------------------|
| `CASCADE` | Child rows bhi automatically delete ho jayengi (Sabse common). |
| `SET NULL` | Child rows ka FK column `NULL` ban jayega (History preserve rakhne ke liye). |
| `RESTRICT` / `NO ACTION` | Error throw karega, parent row tab tak delete nahi ho sakti jab tak child rows maujood hain (Default). |
| `SET DEFAULT` | Child rows me FK ki default value set ho jayegi. |

```sql
-- Example with SET NULL:
CREATE TABLE orders_archived (
    order_id SERIAL PRIMARY KEY,
    amount NUMERIC(10,2),
    customer_id INT REFERENCES customers(customer_id) ON DELETE SET NULL
);
```

---

## 5️⃣ SQL JOINs in Depth
> **Timestamp**: [03:00:41](https://youtu.be/cnzka7kF5Zk?t=10841)

> 💡 **JOINs**: Do ya do se zyada tables ko unke common columns ke base pe combine karke single result nikalna.

---

### 📊 Visual Representation of JOINs

```
       INNER JOIN                   LEFT JOIN                  RIGHT JOIN
     ┌───────┬───────┐            ┌───────┬───────┐           ┌───────┬───────┐
     │       │█████│       │            │███████│█████│       │           │       │█████│███████│
     │   A   │█████│   B   │            │██ A ██│█████│   B   │           │   A   │█████│██ B ██│
     │       │█████│       │            │███████│█████│       │           │       │█████│███████│
     └───────┴───────┘            └───────┴───────┘           └───────┴───────┘
     Matching in BOTH             All of A + Match in B       All of B + Match in A

                               FULL OUTER JOIN
                             ┌───────┬───────┐
                             │███████│█████│███████│
                             │██ A ██│█████│██ B ██│
                             │███████│█████│███████│
                             └───────┴───────┘
                               Everything in A & B
```

---

### 🧪 Sample Data for JOINs
```sql
-- Customers Data
INSERT INTO customers (name, city) VALUES
('Rohan Verma', 'Delhi'),       -- id: 1
('Sanya Malhotra', 'Mumbai'),   -- id: 2
('Kabir Khan', 'Bangalore'),    -- id: 3
('Deepika Sen', 'Kolkata');     -- id: 4 (Has no orders)

-- Orders Data
INSERT INTO orders (amount, customer_id) VALUES
(2500.00, 1),   -- Rohan
(1200.00, 1),   -- Rohan
(4500.00, 2),   -- Sanya
(890.00, NULL); -- Guest checkout (No customer_id)
```

---

### 1. INNER JOIN (Common records only)
> Dono tables me jo rows match hongi sirf wahi dikhegi.

```sql
SELECT 
    orders.order_id,
    customers.name,
    customers.city,
    orders.amount
FROM orders
INNER JOIN customers ON orders.customer_id = customers.customer_id;
```

```
Result:
┌──────────┬────────────────┬───────────┬─────────┐
│ order_id │ name           │ city      │ amount  │
├──────────┼────────────────┼───────────┼─────────┤
│        1 │ Rohan Verma    │ Delhi     │ 2500.00 │
│        2 │ Rohan Verma    │ Delhi     │ 1200.00 │
│        3 │ Sanya Malhotra │ Mumbai    │ 4500.00 │
└──────────┴────────────────┴───────────┴─────────┘
(Deepika and Guest Order are excluded)
```

---

### 2. LEFT JOIN / LEFT OUTER JOIN (All Left Table Rows)
> Left table (`customers`) ki saari rows aayengi. Agar right table (`orders`) me match nahi hai toh `NULL` aayega.

```sql
SELECT 
    customers.customer_id,
    customers.name,
    orders.order_id,
    orders.amount
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id;
```

```
Result:
┌─────────────┬────────────────┬──────────┬─────────┐
│ customer_id │ name           │ order_id │ amount  │
├─────────────┼────────────────┼──────────┼─────────┤
│           1 │ Rohan Verma    │        1 │ 2500.00 │
│           1 │ Rohan Verma    │        2 │ 1200.00 │
│           2 │ Sanya Malhotra │        3 │ 4500.00 │
│           3 │ Kabir Khan     │     NULL │    NULL │  ← No orders
│           4 │ Deepika Sen    │     NULL │    NULL │  ← No orders
└─────────────┴────────────────┴──────────┴─────────┘
```

> 🔥 **Pro Interview Trick**: Customers jinhone kabhi order place nahi kiya:
```sql
SELECT customers.name 
FROM customers
LEFT JOIN orders ON customers.customer_id = orders.customer_id
WHERE orders.order_id IS NULL;
-- Kabir Khan, Deepika Sen
```

---

### 3. RIGHT JOIN / RIGHT OUTER JOIN (All Right Table Rows)
> Right table (`orders`) ki saari rows aayengi, chahe koi customer associated ho ya na ho.

```sql
SELECT 
    customers.name,
    orders.order_id,
    orders.amount
FROM customers
RIGHT JOIN orders ON customers.customer_id = orders.customer_id;
```

```
Result:
┌────────────────┬──────────┬─────────┐
│ name           │ order_id │ amount  │
├────────────────┼──────────┼─────────┤
│ Rohan Verma    │        1 │ 2500.00 │
│ Rohan Verma    │        2 │ 1200.00 │
│ Sanya Malhotra │        3 │ 4500.00 │
│ NULL           │        4 │  890.00 │  ← Guest order (no customer)
└────────────────┴──────────┴─────────┘
```

---

### 4. FULL OUTER JOIN (Everything from Both Tables)
> Left aur Right dono tables ka saara data combine hoga. Non-matching jagahon pe `NULL` fill ho jayega.

```sql
SELECT 
    customers.name,
    orders.order_id,
    orders.amount
FROM customers
FULL OUTER JOIN orders ON customers.customer_id = orders.customer_id;
```

---

### 5. CROSS JOIN (Cartesian Product)
> Har Left row ko Har Right row ke saath pair karta hai ($M \times N$ rows).

```sql
SELECT customers.name, orders.order_id
FROM customers
CROSS JOIN orders;
-- 4 customers * 4 orders = 16 rows
```

---

## 6️⃣ Many-to-Many (M:N) Relationship & Junction Table
> **Timestamp**: [03:12:25](https://youtu.be/cnzka7kF5Zk?t=11545)

> 💡 **Problem**: Ek student multiple courses padh sakta hai, aur ek course me multiple students enroll ho sakte hain. Relational database me directly 2 tables ke beech M:N nahi banaya ja sakta!  
> **Solution**: Beech me ek **Junction Table (Bridge / Associative Table)** banayi jaati hai jo 2 One-to-Many (1:N) relationships banati hai.

```
┌──────────────┐         ┌─────────────────────────┐         ┌──────────────┐
│   STUDENTS   │         │     ENROLLMENTS         │         │   COURSES    │
│ (student_id) │ 1 ─── N │ (student_id, course_id) │ N ─── 1 │ (course_id)  │
└──────────────┘         └─────────────────────────┘         └──────────────┘
```

---

### 🛠️ Schema Implementation
```sql
-- 1. Students Table
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);

-- 2. Courses Table
CREATE TABLE courses (
    course_id SERIAL PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    fee NUMERIC(8,2) NOT NULL
);

-- 3. Junction / Bridge Table (Composite Primary Key)
CREATE TABLE enrollments (
    student_id INT REFERENCES students(student_id) ON DELETE CASCADE,
    course_id INT REFERENCES courses(course_id) ON DELETE CASCADE,
    enrolled_date DATE DEFAULT CURRENT_DATE,
    PRIMARY KEY (student_id, course_id) -- Duplicate enrollment prevent karta hai
);
```

---

### 🧪 Insert & Query Many-to-Many Data
```sql
-- Data
INSERT INTO students (name, email) VALUES
('Sourabh', 'sourabh@tech.com'),
('Pooja', 'pooja@tech.com');

INSERT INTO courses (course_name, fee) VALUES
('PostgreSQL Masterclass', 4999.00),
('System Design Architecture', 7999.00),
('Docker & K8s Bootcamp', 5999.00);

-- Enrollments (Sourabh takes 2 courses, Pooja takes 2 courses)
INSERT INTO enrollments (student_id, course_id) VALUES
(1, 1),
(1, 2),
(2, 2),
(2, 3);
```

### 🔍 3-Table Multi-JOIN Query:
```sql
SELECT 
    s.name AS student_name,
    c.course_name,
    c.fee,
    e.enrolled_date
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
ORDER BY s.name;
```

```
Result:
┌──────────────┬────────────────────────────┬─────────┬───────────────┐
│ student_name │ course_name                │ fee     │ enrolled_date │
├──────────────┼────────────────────────────┼─────────┼───────────────┤
│ Pooja        │ System Design Architecture │ 7999.00 │ 2026-09-19    │
│ Pooja        │ Docker & K8s Bootcamp      │ 5999.00 │ 2026-09-19    │
│ Sourabh      │ PostgreSQL Masterclass     │ 4999.00 │ 2026-09-19    │
│ Sourabh      │ System Design Architecture │ 7999.00 │ 2026-09-19    │
└──────────────┴────────────────────────────┴─────────┴───────────────┘
```

---

## 7️⃣ Hands-On Project: E-STORE Database System
> **Timestamp**: [03:25:42](https://youtu.be/cnzka7kF5Zk?t=12342)

> 🛒 Complete Real-world E-Commerce Database schema with 5 linked tables, constraints, foreign keys, and analytical JOIN queries.

---

### 📐 Entity Relationship (ER) Architecture

```
  ┌──────────────┐         ┌──────────────┐
  │  CATEGORIES  │         │    USERS     │
  └──────┬───────┘         └──────┬───────┘
         │ 1                      │ 1
         │ N                      │ N
  ┌──────┴───────┐         ┌──────┴───────┐
  │   PRODUCTS   │         │    ORDERS    │
  └──────┬───────┘         └──────┬───────┘
         │ 1                      │ 1
         │ N                      │ N
  ┌──────┴────────────────────────┴───────┐
  │             ORDER_ITEMS               │
  │  (order_id, product_id, qty, price)   │
  └───────────────────────────────────────┘
```

---

### 💻 Full Project SQL DDL Schema

```sql
-- Clean up if exists
DROP TABLE IF EXISTS order_items;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS categories;
DROP TABLE IF EXISTS estore_users;

-- 1. Users Table
CREATE TABLE estore_users (
    user_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(120) UNIQUE NOT NULL,
    phone VARCHAR(15),
    city VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Categories Table
CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    category_name VARCHAR(50) UNIQUE NOT NULL
);

-- 3. Products Table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    title VARCHAR(150) NOT NULL,
    category_id INT REFERENCES categories(category_id) ON DELETE SET NULL,
    price NUMERIC(10,2) NOT NULL CHECK (price > 0),
    stock_qty INT NOT NULL DEFAULT 0 CHECK (stock_qty >= 0)
);

-- 4. Orders Table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT REFERENCES estore_users(user_id) ON DELETE CASCADE,
    order_status VARCHAR(20) DEFAULT 'PLACED' CHECK (order_status IN ('PLACED', 'SHIPPED', 'DELIVERED', 'CANCELLED')),
    total_amount NUMERIC(10,2) DEFAULT 0.00,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 5. Order Items (Junction Table between Orders & Products)
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
    product_id INT NOT NULL REFERENCES products(product_id) ON DELETE RESTRICT,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10,2) NOT NULL CHECK (unit_price > 0)
);
```

---

### 📦 Seed Data Insertion

```sql
-- Insert Categories
INSERT INTO categories (category_name) VALUES 
('Electronics'), ('Fashion'), ('Home & Kitchen');

-- Insert Users
INSERT INTO estore_users (name, email, phone, city) VALUES
('Sourabh Sharma', 'sourabh@mail.com', '9876543210', 'Delhi'),
('Rohit Verma',   'rohit@mail.com',   '9811223344', 'Bangalore'),
('Ananya Sen',    'ananya@mail.com',  '9922334455', 'Mumbai');

-- Insert Products
INSERT INTO products (title, category_id, price, stock_qty) VALUES
('Sony WH-1000XM5 Headphones', 1, 26990.00, 25),
('MacBook Air M2',             1, 94990.00, 15),
('Nike Air Jordan Sneaker',    2, 11495.00, 40),
('Instant Pot Duo 7-in-1',     3, 8999.00,  30);

-- Insert Orders
INSERT INTO orders (user_id, order_status, total_amount) VALUES
(1, 'DELIVERED', 121980.00), -- Sourabh bought Laptop + Headphones
(2, 'PLACED',    11495.00),  -- Rohit bought Sneaker
(1, 'PLACED',    8999.00);   -- Sourabh bought Instant Pot

-- Insert Order Items
INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
(1, 1, 1, 26990.00), -- Order 1: 1x Headphones
(1, 2, 1, 94990.00), -- Order 1: 1x MacBook
(2, 3, 1, 11495.00), -- Order 2: 1x Sneaker
(3, 4, 1, 8999.00);  -- Order 3: 1x Cooker
```

---

### 📊 Real-World Analytical Queries on E-STORE

#### Query 1: Customer Order Summary with Product Details (4-Table JOIN)
```sql
SELECT 
    o.order_id,
    u.name AS customer_name,
    u.city,
    p.title AS product_name,
    c.category_name,
    oi.quantity,
    oi.unit_price,
    (oi.quantity * oi.unit_price) AS line_total,
    o.order_status
FROM orders o
INNER JOIN estore_users u ON o.user_id = u.user_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
INNER JOIN categories c ON p.category_id = c.category_id
ORDER BY o.order_id;
```

#### Query 2: Top Spending Customers (JOIN + GROUP BY + Aggregation)
```sql
SELECT 
    u.user_id,
    u.name,
    COUNT(DISTINCT o.order_id) AS total_orders_placed,
    SUM(oi.quantity * oi.unit_price) AS total_money_spent
FROM estore_users u
INNER JOIN orders o ON u.user_id = o.user_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY u.user_id, u.name
ORDER BY total_money_spent DESC;
```

#### Query 3: Category-wise Revenue & Units Sold
```sql
SELECT 
    c.category_name,
    SUM(oi.quantity) AS units_sold,
    SUM(oi.quantity * oi.unit_price) AS total_revenue
FROM categories c
INNER JOIN products p ON c.category_id = p.category_id
INNER JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY c.category_name
ORDER BY total_revenue DESC;
```

#### Query 4: Find Inactive Users (Users who never placed an order)
```sql
SELECT 
    u.user_id, 
    u.name, 
    u.email 
FROM estore_users u
LEFT JOIN orders o ON u.user_id = o.user_id
WHERE o.order_id IS NULL;
-- Ananya Sen (No orders yet)
```

---

## 🧠 Quick Revision Cheatsheet

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            ALTER COMMANDS                                   │
│  ALTER TABLE t ADD COLUMN col datatype;                                     │
│  ALTER TABLE t DROP COLUMN col [CASCADE];                                   │
│  ALTER TABLE t RENAME COLUMN old_col TO new_col;                            │
│  ALTER TABLE t ALTER COLUMN col TYPE new_datatype;                          │
│  ALTER TABLE t ALTER COLUMN col SET/DROP DEFAULT val;                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                          CHECK CONSTRAINTS                                  │
│  CHECK (col > 0)                                → Positive numbers only     │
│  CHECK (rating BETWEEN 1 AND 5)                 → Range check               │
│  ALTER TABLE t ADD CONSTRAINT cname CHECK (...); → Add custom rule          │
├─────────────────────────────────────────────────────────────────────────────┤
│                          RELATIONSHIPS & FK                                 │
│  1:1  → UNIQUE constraint on Foreign Key column                             │
│  1:N  → Standard Foreign Key referencing Parent Primary Key                 │
│  M:N  → Junction/Bridge Table with Composite Primary Key (PK(id1, id2))    │
│  ON DELETE CASCADE     → Delete parent = child rows deleted automatically   │
│  ON DELETE SET NULL    → Delete parent = child FK becomes NULL              │
│  ON DELETE RESTRICT    → Block parent delete if children exist              │
├─────────────────────────────────────────────────────────────────────────────┤
│                              SQL JOINS                                      │
│  INNER JOIN       → Rows matching in BOTH tables                            │
│  LEFT JOIN        → All rows from Left + matching from Right                │
│  RIGHT JOIN       → All rows from Right + matching from Left                │
│  FULL OUTER JOIN  → All rows from both (NULL where no match)                │
│  CROSS JOIN       → Cartesian product (M x N rows)                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

> **Next**: Day 4 — Views, Subqueries & Window Functions 🚀
