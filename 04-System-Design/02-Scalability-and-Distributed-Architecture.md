# 🚀 System Architecture & Scalability Guide
*(Production System Design Fundamentals - Real World Amazon Architecture)*

---

## 🌐 1. The Foundation: Client, Server & DNS

Har real-world internet system **Client** aur **Server** se start hota hai.

```text
[ Client (Browser/App) ] ──(1. amazon.com)──► [ DNS Server (Global Directory) ]
           │                                          │ (2. Returns IP: 10.2.3.4)
           └────────────(3. HTTP Request)─────────────► [ Server (10.2.3.4) ]
```

### 💡 Why DNS? (Domain Name System)
- **Problem**: Har machine/server ka physical **IP Address** (e.g. `10.2.3.4`) hota hai jo insaan yaad nahi rakh sakte.
- **Solution**: **DNS Server** Internet ka Phonebook/Address Directory ki tarah kaam karta hai jo domain name (`amazon.com`) ko IP Address (`10.2.3.4`) me convert karta hai.

---

## 📈 2. Scaling Strategies: Vertical vs Horizontal

Jab app par users badhte hain, to single server overload hone lagta hai. Isse solve karne ke 2 tarike hain:

| Feature | 📈 Vertical Scaling (Scale Up) | 📉 Horizontal Scaling (Scale Out) |
| :--- | :--- | :--- |
| **How it Works** | Single machine me CPU/RAM increase karna | Extra new servers add karna |
| **Downtime** | Requires Server Restart (Downtime) | **Zero Downtime** (Live addition) |
| **Limit** | Hardware Limit exist karti hai | Unlimited Scaling (Elasticity) |
| **Fault Tolerance** | Low (Single Point of Failure) | **High** (Ek server down ho to baki handle kar lenge) |
| **Requirement** | Simple (No extra software) | Requires a **Load Balancer** |

---

## 🔄 3. Load Balancers & API Gateway (Microservices)

### ⚖️ Load Balancer (LB)
Jab multiple servers ho jayein, to traffic equal distribute karne ke liye **Load Balancer** middleman ki tarah kaam karta hai.

```text
                               ┌──► [ Server 1 ]
[ User Request ] ──► [ Load Balancer (LB) ] ┼──► [ Server 2 ]
                               └──► [ Server 3 ]
```
* **Routing Algorithms**:
  - **Round Robin**: Requests barabar sequence me rotate karti hain ($S_1 \to S_2 \to S_3$).
  - **Health Checks**: Dead/crashed servers ko auto-detect karke traffic nahi bhejta.
* **AWS Component**: Elastic Load Balancer (ELB).

---

### 🔑 API Gateway (Microservices Router)
Monolith ko tod kar jab hum **Microservices** (Auth, Orders, Payments) banate hain:

```text
[ Client ] ──► [ API Gateway ] ───┬──► /auth ────► [ Auth Load Balancer ]
                                  ├──► /orders ──► [ Order Load Balancer ]
                                  └──► /pay ─────► [ Payment Load Balancer ]
```
* **API Gateway Role**: Centralized entry point jo URL Path (`/auth`, `/orders`), Headers, aur Authentication Check ke basis par request specific Microservice tak redirect karta hai.

---

## ⚡ 4. Asynchronous Processing: Queues & Pub/Sub

Synchronous (blocking) API calls system ko slow kar deti hain. Long-running tasks (Emails, SMS, Video processing) ke liye **Asynchronous Architecture** use hota hai.

### 📩 Message Queues (FIFO Buffer - e.g. AWS SQS / BullMQ)
Producer task queue me push karta hai, Consumer/Worker aage se process karta hai.

* **Decoupling**: Payment Service email bhejane ke liye block nahi hoti.
* **Rate Limiting Protection**: Agar Gmail API 10 emails/sec ki limit rakhti hai, to Queue buffering handle karti hai.

---

### 📢 Pub/Sub Pattern (Publish-Subscribe - e.g. AWS SNS)
Ek single event se multiple actions trigger karne ke liye.

**Scenario**: User Order place karta hai ➔ Email, SMS, Notification, aur Inventory charo update hone hain.
* **Flow**: Order Service SNS Topic par **Publish** karegi ➔ Subscribed services (**Email, SMS, DB**) message read karengi.

---

## 🔄 5. Fan-Out Architecture (The Best of Both Worlds)

Pub/Sub (SNS) aur Message Queues (SQS) ko combine karke **Reliable Multi-Action Trigger** banaya jata hai:

```text
                         ┌──► [ Email Queue ] ──► (Email Worker)
                         │
[ Payment Service ] ──► [ SNS Topic ] ┼──► [ SMS Queue ]   ──► (SMS Worker)
                         │
                         └──► [ DB Queue ]    ──► (DB Worker)
                                  │ (If Fails)
                                  └──► [ Dead Letter Queue (DLQ) ]
```

- **Benefit**: Multi-service event distribution + Failure Isolation.
- **Dead Letter Queue (DLQ)**: Agar koi message process hone me baar-baar fail ho, to use safe manual debugging ke liye **DLQ** me move kar diya jata hai.

---

## 💎 6. Caching & Database Scaling

### 🚀 Caching (Redis / Memcached)
Frequently accessed data ko Memory (RAM) me store karke DB load 90% tak kam karna.
- **Flow**: Request ➔ Check Cache (Hit) ➔ Return. If Miss ➔ Fetch from DB ➔ Store in Cache ➔ Return.

### 🗄️ DB Replication (Read/Write Separation)
- **Primary / Master Node**: Handles all **Writes** (INSERT, UPDATE, DELETE).
- **Read Replicas**: Handles all **Reads** (SELECT queries for Analytics/Users).
- **Benefit**: Read-heavy applications me primary DB par load crash nahi hota.

---

## 🌍 7. Content Delivery Network (CDN / CloudFront)

Static assets (Images, Videos, CSS, JS) ko globally serve karne ke liye:

```text
[ User in India ] ──► [ Edge Location (Mumbai) ] ──(Cache Hit)──► FAST (10ms)
                                  │
                       (Cache Miss)│ (Fetch from Origin)
                                  ▼
                     [ Origin Server (US East) ]
```
- **Benefits**: Super low latency + Origin server bandwidth savings.

---

## 🛡️ 8. Rate Limiting & Protection

System ko DDoS Attacks, Bots, aur Abuse se bachane ke liye Rate Limiting hoti hai (e.g. Max 5 Requests / sec per IP).

* **Token Bucket Algorithm**: Fixed tokens add hote hain; traffic bursts handle karta hai.
* **Leaky Bucket Algorithm**: Fixed rate par traffic ko smooth outbound flow deta hai.

---

## 📋 Summary Checklist for Designing Scalable Systems

- [x] **DNS**: Translates human domains to machine IP addresses.
- [x] **Horizontal Scaling**: Scales out via multiple servers instead of upgrading hardware.
- [x] **Load Balancer**: Distributes incoming traffic smoothly (ELB).
- [x] **API Gateway**: Microservice routing and central security check.
- [x] **Message Queue (SQS)**: Asynchronous decoupled task processing.
- [x] **Pub/Sub (SNS)**: One-to-many event notification.
- [x] **Fan-Out Pattern**: SNS + SQS combination with Dead Letter Queue (DLQ).
- [x] **Caching (Redis)**: Sub-millisecond in-memory data retrieval.
- [x] **DB Replication**: Primary Node for Writes, Replicas for Reads.
- [x] **CDN (CloudFront)**: Serves static assets from nearest edge servers.
- [x] **Rate Limiter**: Token Bucket / Leaky Bucket protection against abuse.

