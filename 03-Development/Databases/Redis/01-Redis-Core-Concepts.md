# ⚡ Redis Core Concepts & CLI Guide
*(Chai aur Code Playlist Notes - Personal Study Log)*

---

## 📌 1. What is Redis & Why do we need it?

**Redis** (Remote Dictionary Server) ek super-fast, open-source **In-Memory Key-Value Data Store** hai.

### 🧠 RAM vs Disk Storage (The Core Concept)
* **Traditional DBs (MongoDB / PostgreSQL)**: Data Hard Disk / SSD me store karte hain. Reading/writing requires disk I/O, jo slow hota hai (~1-10 ms).
* **Redis**: Data pure **RAM (Memory)** me store rakhta hai. Memory access nanoseconds me hoti hai, so response sub-millisecond (<1 ms) milta hai!

```text
[ User Request ] ──► [ Express App ] ──► [ Redis RAM (Sub-millisecond) ] ⚡ FAST
                                      └──► [ MongoDB / SQL Disk (Slow) ] 🐌 SLOW
```

### 🎯 Real-World Use Cases (Mene Kya Samjha):
1. **Database Load Reduction**: Home Page Banners ya Trending Products constant rehne par DB query bar-bar run karne se accha Redis RAM me cache karo.
2. **Session / Token Storage**: User login status and JWT sessions store karne ke liye.
3. **Auto Expiry (TTL)**: OTPs, Password Reset Tokens jo 5 min me expire hone chahiye.
4. **Leaderboards & Queues**: Gaming scores track karne ke liye (Sorted Sets) aur Message Queues.

---

## 🛠️ 2. Redis Setup & Top CLI Commands

Connection check karne ke liye run `redis-cli`:

### 📋 Essential CLI Command Cheat Sheet

| Category | Command | Syntax & Example | Output / Meaning |
| :--- | :--- | :--- | :--- |
| **Connection** | `PING` | `PING` | `PONG` (Server active hai) |
| **String Set** | `SET` | `SET user:name "ChaiCode"` | `OK` |
| **String Get** | `GET` | `GET user:name` | `"ChaiCode"` |
| **Expiry Set** | `SETEX` | `SETEX otp:101 60 "492810"` | Set value with 60 sec expiry |
| **Time To Live** | `TTL` | `TTL otp:101` | Returns remaining seconds |
| **Check Key** | `EXISTS` | `EXISTS user:name` | `1` (True) or `0` (False) |
| **Delete Key** | `DEL` | `DEL user:name` | `1` (Deleted) |

---

## 📦 3. Redis Data Structures Deep Dive

Redis sirf simple key-value nahi hai, follows 5 core data structures:

### 1️⃣ Strings
Most basic key-value data type (Text, JSON string, Numbers).
* `INCR page_views` ➔ Atomically increment count by 1.
* `DECR page_views` ➔ Decrement count by 1.

### 2️⃣ Hashes (Objects)
Single key ke andar object fields save karne ke liye.
```bash
127.0.0.1:6379> HSET user:101 name "Hitesh" role "Instructor"
127.0.0.1:6379> HGET user:101 name
"Hitesh"
127.0.0.1:6379> HGETALL user:101
1) "name"  2) "Hitesh"  3) "role"  4) "Instructor"
```

### 3️⃣ Lists (Arrays / Queues)
Ordered sequence of strings. Message queues aur activity logs ke liye best.
* `LPUSH notifications "Video 1 Uploaded"` ➔ Push to left
* `RPOP notifications` ➔ Pop from right (FIFO Queue)

### 4️⃣ Sets (Unique Collections)
Unordered collection of unique strings (Duplicates automatically ignore hote hain).
```bash
127.0.0.1:6379> SADD post:1:likes "user_1" "user_2" "user_1" # "user_1" duplicate ignore ho jayega
127.0.0.1:6379> SMEMBERS post:1:likes
```

### 5️⃣ Sorted Sets (ZSets - Leaderboards)
Har element ek **Score** ke sath associated hota hai. Scores ke basis par automatic sorting hoti hai!
```bash
127.0.0.1:6379> ZADD leaderboard 1500 "Player_A" 2800 "Player_B"
127.0.0.1:6379> ZREVRANGE leaderboard 0 -1 WITHSCORES # Top players highest score first
```

---

## 🔑 Key Takeaways & Summary

> Redis primary database ko replacement nahi karta. Ye Primary DB (Mongo/SQL) ke samne ek **Fast In-Memory Layer** ki tarah stand karta hai to boost speed and prevent server crashes!

