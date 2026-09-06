# 📐 System Design #5: Back-of-the-Envelope Estimation & Rate Limiting
*(Notes from Piyush Garg System Design Series & ByteByteGo)*

> **Video References**:
> - Part 1: [Back of the Envelope Calculation - Piyush Garg](https://youtu.be/DwqTon7ZS_s)
> - Part 2: [Master Rate Limiting - Piyush Garg](https://youtu.be/CVItTb_jdkE)
> - Deep-Dive Article: [ByteByteGo: Design a Rate Limiter](https://bytebytego.com/courses/system-design-interview/design-a-rate-limiter)

---

# PART 1: 🧮 Back-of-the-Envelope Calculations

System Design interviews me actual code likhne se pehle interviewer puchta hai: *"Is system ke liye kitne servers aur kitna storage lagega?"*  
Is estimation ko hum **Back-of-the-Envelope Calculation** bolte hain.

---

## 🎯 1. Essential Rules of Thumb (Cheat Sheet)

Numbers ko fast calculate karne ke liye powers of 10 yaad rakhein:

| Power | Approximation | Name |
| :--- | :--- | :--- |
| $10^3$ | Thousand | 1 KB |
| $10^6$ | Million | 1 MB |
| $10^9$ | Billion | 1 GB |
| $10^{12}$ | Trillion | 1 TB |
| $10^{15}$ | Quadrillion | 1 PB (Petabyte) |

> 💡 **Golden Time Shortcut**:  
> Ek din me **86,400 seconds** hote hain. Estimation me hum ise roughly **$10^5$ seconds** (100,000 seconds) maan lete hain taaki math dimag me fast ho sake!

---

## 📊 2. Step-by-Step Calculation Framework (e.g. Twitter / Instagram)

Maano hum ek social media platform design kar rahe hain with **100 Million Daily Active Users (DAU)**.

### A. QPS (Queries Per Second) Estimation
* **Assumption**: Average user din me 10 tweets read karta hai aur 1 tweet post karta hai.
* **Total Writes per day**: $100\text{ Million} \times 1 = 100\text{M writes/day}$
* **Write QPS**: $\frac{100,000,000}{100,000\text{ seconds}} = \mathbf{1,000\text{ QPS}}$
* **Peak QPS**: Normal traffic ka $2\times$ ya $3\times$ (Peak QPS $\approx \mathbf{2,000\text{ to } 3,000\text{ QPS}}$).
* **Read QPS** (10:1 Read-to-Write ratio): $1,000 \times 10 = \mathbf{10,000\text{ QPS}}$.

---

### B. Storage Estimation (5-Year Horizon)
* **Per Tweet Size**: Tweet text (140 bytes) + Metadata (user ID, timestamp, etc. $\approx 360$ bytes) = **500 Bytes**.
* **Daily Storage**: $100\text{ Million} \times 500\text{ Bytes} = 50\text{ GB / day}$.
* **1 Year Storage**: $50\text{ GB} \times 365 \approx \mathbf{18.25\text{ TB / year}}$.
* **5 Years Storage**: $18.25\text{ TB} \times 5 \approx \mathbf{91.25\text{ TB}}$.

---

### C. Memory / Caching Estimation (80/20 Rule)
* Pareto Principle (80/20 rule): **20% content generates 80% of traffic**.
* Daily Read Volume = $10\text{ tweets} \times 100\text{M users} = 1\text{ Billion reads/day} = 500\text{ GB data}$.
* **RAM needed to cache top 20%**: $20\% \text{ of } 500\text{ GB} = \mathbf{100\text{ GB RAM}}$.  
  *(Redis cluster with 128GB RAM easily pura cache sambhal lega!)*

---
---

# PART 2: 🛡️ Master Rate Limiting Architecture

## ❓ What is a Rate Limiter & Why do we need it?

A **Rate Limiter** controls the rate of traffic sent by a client or service. Agar user allowed limit se zyada requests bhejta hai, to unhe block (`HTTP 429 Too Many Requests`) kar diya jata hai.

### Why Rate Limiting is Critical:
1. **Prevent DoS / DDoS Attacks**: Spammers ko server crash karne se rokna.
2. **Cost Optimization**: Third-party paid APIs (e.g. OpenAI, Stripe, Twilio) par bill explosion se bachana.
3. **Prevent Server Overload**: Heavy queries filter out karke database health protect karna.

---

## 📍 1. Where to Place the Rate Limiter?

```text
[ Client ] ──► [ API Gateway / Middleware (Rate Limiter) ] ──► [ Backend Services ]
                                │
                                ▼
                       [ Centralized Cache (Redis) ]
```

* **Client-Side**: ❌ Unreliable (Attacker client code manipulate kar sakta hai).
* **Server-Side App Level**: ⚠️ Hard to manage if multiple microservices exist.
* **API Gateway / Middleware (Best Practice)**: ✅ Centralized entry point par traffic filter ho jata hai before hitting actual servers.

---

## ⚙️ 2. The 5 Core Rate Limiting Algorithms (Deep Dive)

Piyush Garg & ByteByteGo explain 5 major algorithms:

### 1️⃣ Token Bucket (Most Popular - Used by AWS, Stripe)
* **Concept**: Ek bucket me fixed rate par tokens girte hain (e.g. 2 tokens/sec). Har incoming request 1 token consume karti hai.
* **Burst Handling**: Agar bucket full hai, to sudden burst of requests execute ho jati hain.
* **Pros**: Memory efficient, supports short bursts of traffic.
* **Cons**: Refill rate aur bucket size do parameters tune karne padte hain.

```text
[ Refill Rate: 2 tokens/sec ] ──► [ 🪣 Bucket (Capacity: 5) ]
                                            │
                                            ▼ (Request consumes 1 token)
                                     [ Allowed ✅ / Dropped ❌ ]
```

---

### 2️⃣ Leaky Bucket (Smooth Outbound Traffic - e.g. Shopify)
* **Concept**: Bucket me requests daalo (FIFO queue). Bucket ke bottom se requests **constant fixed rate** par leak/process hoti hain.
* **Pros**: Steady outflow deta hai, burst traffic ko smooth kar deta hai.
* **Cons**: Bursts delay ho jate hain; queue full hone par fresh requests drop ho jati hain.

---

### 3️⃣ Fixed Window Counter
* **Concept**: Timeline ko fixed windows (e.g. 1 minute) me divide karo. Har window me counter lagao (Max 5 requests/min).
* **The Flaw (Boundary Spike Trap)**:
  - Window ke end (0:59) par 5 requests aayi.
  - Agli window ke start (1:01) par 5 requests aayi.
  - 2 seconds ke span me **10 requests** pass ho gayi (2x limit breach)!

---

### 4️⃣ Sliding Window Log
* **Concept**: Har request ka timestamp Redis Sorted Set (`ZSET`) me save karo. Current time se purane timestamps delete karke log count check karo.
* **Pros**: 100% accurate, koi boundary problem nahi.
* **Cons**: **Heavy Memory Consumption**! Millions of requests ke timestamps RAM me store karna expensive ho jata hai.

---

### 5️⃣ Sliding Window Counter (Industry Standard Compromise)
* **Concept**: Fixed Window ki low memory aur Sliding Log ki accuracy ko math formula se combine karta hai:
  $$\text{Requests in Window} = \text{Current Window Requests} + (\text{Previous Window Requests} \times \text{Overlap \%})$$
* **Pros**: Super low memory + smooth traffic without boundary spikes!

---

## 🥊 Comparison Matrix

| Algorithm | Memory Usage | Handles Bursts? | Traffic Smoothness | Best Used For |
| :--- | :---: | :---: | :---: | :--- |
| **Token Bucket** | 🟢 Very Low | ✅ Yes | 🟡 Moderate | APIs with burst allowance (AWS, Stripe) |
| **Leaky Bucket** | 🟢 Low | ❌ No | 🟢 Constant/Smooth | E-commerce checkout, stable job processing |
| **Fixed Window** | 🟢 Minimal | ❌ Burst Issue | 🔴 High Spikes | Simple basic prototypes |
| **Sliding Log** | 🔴 Very High | ✅ Yes | 🟢 Perfect | Strict security apps with low RPS |
| **Sliding Counter** | 🟢 Low | ✅ Yes | 🟢 Smooth | High-scale enterprise production systems |

---

## 🌐 3. Distributed Rate Limiter & Redis Race Conditions

Multiple backend servers me distributed rate limiting ke liye hum **Redis** in-memory store use karte hain.

### ⚠️ The Concurrency Race Condition Problem:
Jab 2 requests simultaneously aati hain:
1. Server 1 reads `counter = 4` from Redis.
2. Server 2 reads `counter = 4` from Redis.
3. Both increment to `5` and allow the request, even though the limit was 5!

```text
Request A ──► Read (4) ──► Write (5) ──► ALLOW ✅
                                                   ❌ Limit was 5, but 6 requests passed!
Request B ──► Read (4) ──► Write (5) ──► ALLOW ✅
```

### 💡 The Solution:
1. **Redis Lua Scripts (Recommended)**: Redis me Lua script **Atomic Execution** ensure karti hai (read + increment + expiry ek single uninterrupted step me hoti hai).
2. **Redis Sorted Sets (`ZREMRANGEBYSCORE` + `ZCARD`)**: Atomic sliding window handling.

---

## 📡 4. HTTP Headers & Status Codes

Jab Rate Limiter kisi client ko block karta hai:
* **HTTP Status Code**: `429 Too Many Requests`
* **Response Headers**:
  - `X-Ratelimit-Limit`: Total requests allowed per window (e.g. `100`).
  - `X-Ratelimit-Remaining`: Current window me kitni requests bachi hain (e.g. `23`).
  - `X-Ratelimit-Retry-After`: Kitne seconds wait karna padega before next request (e.g. `30`).

---

## 🔗 Deep-Dive Reference
- [ByteByteGo: Design a Rate Limiter Course](https://bytebytego.com/courses/system-design-interview/design-a-rate-limiter)

