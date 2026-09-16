# 🐘 PostgreSQL #1: Fundamentals, Architecture, Data Types & CRUD
*(Notes from MPrashant PostgreSQL Masterclass - Part 1 up to 1:17:00)*

> **Video Reference**: [Master POSTGRESQL in ONE VIDEO - MPrashant](https://youtu.be/cnzka7kF5Zk)  
> **Module**: Relational Databases (RDBMS) & PostgreSQL Core  
> **Estimated Plan**: 4-Day Deep Dive (Day 1: Setup, Data Types, Table Constraints & CRUD)

---

## ❓ 1. What is PostgreSQL & Why is it the #1 RDBMS?

**PostgreSQL** (popularly called **Postgres**) ek open-source, highly reliable, aur advanced **Object-Relational Database Management System (ORDBMS)** hai.

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        Why Developers Love Postgres?                   │
├────────────────────────────────────────────────────────────────────────┤
│ 🛡️ 100% ACID Compliant     ➔ Banking-grade reliability (No data loss)  │
│ ⚡ Highly Extensible        ➔ Custom data types, functions, and plugins │
│ 📦 JSON & JSONB Support     ➔ NoSQL jaisi power SQL ke andar!           │
│ 🔍 Powerful Indexing       ➔ B-Tree, Hash, GiST, GIN (for full-text)   │
│ 🆓 100% Free & Open-Source  ➔ No license cost, backed by global comm.   │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 💻 2. Essential `psql` CLI Commands

Terminal me database inspect karne ke liye sabse common shortcut meta-commands:

| Command | What it does | Example Output |
| :--- | :--- | :--- |
| `\l` | List all databases | Lists `postgres`, `my_db`, etc. |
| `\c <dbname>` | Connect / Switch to a database | `You are now connected to database "bookstore"` |
| `\dt` | List all tables in current database | Shows list of tables |
| `\d <table_name>` | Describe table schema (columns, types, keys) | Detailed column types & constraints |
| `\dn` | List all schemas | Shows `public`, etc. |
| `\du` | List all users and roles | Lists superusers & permissions |
| `\q` | Quit / Exit psql terminal | Exits back to bash/powershell |

---

## 📊 3. PostgreSQL Core Data Types

Postgres me data store karne ke liye major categories:

### 1️⃣ Numeric Types
* `INT` / `INTEGER`: Standard 4-byte integer ($-2 \times 10^9$ to $+2 \times 10^9$).
* `BIGINT`: 8-byte integer for large IDs (e.g. billion users).
* `SERIAL` / `BIGSERIAL`: Auto-incrementing integer (MySQL ke `AUTO_INCREMENT` jaisa).
* `NUMERIC(precision, scale)` / `DECIMAL`: Exact floating numbers (Best for **Money & Currency**, e.g. `NUMERIC(10, 2)` ➔ up to 10 digits with 2 decimal places).

### 2️⃣ Character / String Types
* `VARCHAR(n)`: Variable length string with limit $n$ (e.g. `VARCHAR(100)`).
* `CHAR(n)`: Fixed length string (Padded with spaces, e.g. country codes `CHAR(2)`).
* `TEXT`: Unlimited length string (Best for blog posts, bios, descriptions).

### 3️⃣ Date & Time Types
* `DATE`: Date only (`2026-09-16`).
* `TIME`: Time only (`14:30:00`).
* `TIMESTAMP`: Date + Time (`2026-09-16 14:30:00`).
* `TIMESTAMPTZ`: Timestamp with **Time Zone** (Industry Best Practice for global apps!).

### 4️⃣ Boolean & Modern Types
* `BOOLEAN`: `TRUE`, `FALSE`, or `NULL`.
* `UUID`: Universally Unique Identifier (`uuid_generate_v4()`).
* `JSONB`: Binary JSON (Faster than plain JSON, supports indexing).

---

## 🏗️ 4. Table Creation & Constraints (DDL)

Ek clean e-commerce `users` aur `products` table ka practical example:

```sql
-- 1. Create Users Table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,                          -- Auto-incrementing Unique ID
    full_name VARCHAR(100) NOT NULL,               -- Mandatory field
    email VARCHAR(255) UNIQUE NOT NULL,            -- No duplicates allowed
    age INT CHECK (age >= 18),                     -- Validation Constraint
    is_active BOOLEAN DEFAULT TRUE,                -- Default value
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP -- Auto-generated timestamp
);

-- 2. Create Products Table with Foreign Key
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    title VARCHAR(150) NOT NULL,
    price NUMERIC(10, 2) NOT NULL CHECK (price > 0),
    stock INT DEFAULT 0 CHECK (stock >= 0),
    created_by INT REFERENCES users(id) ON DELETE CASCADE -- Foreign Key link
);
```

### 🛡️ Constraints Explained:
* `PRIMARY KEY`: Unique + Not Null identity for each row.
* `NOT NULL`: Field empty nahi ho sakta.
* `UNIQUE`: Column me duplicate value allow nahi hogi (e.g. Email).
* `CHECK`: Custom validation logic (e.g. `age >= 18`, `price > 0`).
* `DEFAULT`: Agar user value na de to default insert hoga.
* `ON DELETE CASCADE`: Agar parent user delete ho, to uske products bhi auto-delete ho jayein!

---

## 🛠️ 5. Modifying Tables (`ALTER TABLE`)

Agar table banne ke baad structure change karna ho:

```sql
-- Naya column add karna
ALTER TABLE users ADD COLUMN phone VARCHAR(15);

-- Column delete karna
ALTER TABLE users DROP COLUMN phone;

-- Column rename karna
ALTER TABLE users RENAME COLUMN full_name TO name;

-- Constraint add karna
ALTER TABLE users ADD CONSTRAINT unique_phone UNIQUE (phone);
```

---

## ⚡ 6. CRUD Operations (DML - Insert, Select, Update, Delete)

### 📥 1. Create (Insert Data)
```sql
-- Single row insert
INSERT INTO users (name, email, age) 
VALUES ('Sourabh', 'sourabh@dev.com', 22);

-- Multiple rows insert in one go
INSERT INTO users (name, email, age) VALUES 
('Rahul Sharma', 'rahul@gmail.com', 24),
('Sneha Verma', 'sneha@yahoo.com', 20),
('Aman Singh', 'aman@outlook.com', 29);
```

---

### 🔍 2. Read (Querying Data & Powerful Filtering)

```sql
-- Saare columns fetch karna
SELECT * FROM users;

-- Specific columns with Aliases
SELECT name, email AS user_email FROM users;

-- Filtering with WHERE
SELECT * FROM users WHERE age >= 22;

-- Multiple conditions (AND / OR)
SELECT * FROM users WHERE age >= 20 AND is_active = TRUE;

-- Range filter (BETWEEN)
SELECT * FROM users WHERE age BETWEEN 20 AND 25;

-- List filter (IN / NOT IN)
SELECT * FROM users WHERE email IN ('sourabh@dev.com', 'rahul@gmail.com');

-- Case-Insensitive Pattern Search (ILIKE - Postgres Special!)
-- 'LIKE' is case-sensitive, 'ILIKE' matches regardless of UPPER/lower case
SELECT * FROM users WHERE name ILIKE '%rahul%';

-- Sorting & Pagination
SELECT * FROM users 
ORDER BY created_at DESC 
LIMIT 5 OFFSET 0; -- Page 1 (first 5 records)
```

---

### ✏️ 3. Update Data
```sql
-- Safe update with WHERE condition
UPDATE users 
SET age = 23, is_active = TRUE 
WHERE email = 'sourabh@dev.com';
```
> ⚠️ **Caution**: Hamesha `WHERE` clause check karein, warna table ka saara data update ho jayega!

---

### 🗑️ 4. Delete Data
```sql
-- Specific row delete karna
DELETE FROM users WHERE id = 1;

-- Table ka saara data delete karna (Fast reset)
TRUNCATE TABLE users RESTART IDENTITY CASCADE;
```

---

## 💡 Key Takeaways (Day 1 Summary)

1. **Postgres enforces data integrity at the database level**: Application code check kare ya na kare, Postgres ke `CHECK`, `NOT NULL`, aur `UNIQUE` constraints bad data ko enter hi nahi hone dete.
2. **Always prefer `TIMESTAMPTZ` over `TIMESTAMP`**: Ye UTC me store karta hai aur client ke timezone me display karta hai, preventing international date bugs.
3. **Use `ILIKE` for Indian/Global search**: Case-insensitive substring search ke liye `ILIKE` Postgres ka super-handy feature hai.

