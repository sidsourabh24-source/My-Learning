# 🏗️ 01. What is System Design? (Piyush Garg Series Notes)

> **Video Reference**: [System Design Course #1 - Piyush Garg](https://youtu.com/lFeYU31TnQ8)  
> **Topic**: Introduction to System Design, Client-Server Model, Monolith vs Microservices & Key Metrics.

---

## 🎯 1. What is System Design? (Hinglish Summary)

Simple words me: **System Design** ka matlab hai ek aisa software architecture design karna jo **Scalable**, **Reliable**, **Maintainable**, aur **Fast** ho jab user base 1 user se **1 Million+ users** tak grow kare.

- **Coding vs System Design**:
  - **Coding**: Code kaise likhna hai (Syntax, Algorithms, Functions).
  - **System Design**: System ke parts ko kaise arrange karna hai (Servers, Databases, Load Balancers, Caching, Messaging).

---

## 🌐 2. The Core Client-Server Architecture

Her Web & Mobile app ka basic foundation:

```text
┌──────────────┐         HTTP Request (GET/POST)         ┌──────────────────┐
│              │ ──────────────────────────────────────► │                  │
│    Client    │                                         │   Backend App    │─► [ Database ]
│ (Browser/App)│ ◄────────────────────────────────────── │ (Express/Node.js)│    (Mongo/SQL)
└──────────────┘             HTTP Response               └──────────────────┘
```

1. **Client**: Frontend app jo request bhejti hai (React, Android app, Postman).
2. **Server**: Backend logic compute karta hai.
3. **Database**: Permanent data store karta hai.

---

## 🏛️ 3. Monolithic vs Microservices Architecture

### 🔴 Monolithic Architecture (Single Codebase)
Ek hi continuous codebase me saare modules (Auth, Payments, Orders, Notifications) pack hote hain.

* **Pros**: Simple to build, easy to test initially, low deployment complexity.
* **Cons**: 
  - **Single Point of Failure**: Agar Auth module crash hua, to poori app crash!
  - **Scaling Issue**: Sirf Payment module par traffic badha to poori app ko scale karna padega.

---

### � Microservices Architecture (Decoupled Services)
Har module (Auth, Payment, Order) ek independent mini-server/service hota hai jo APIs ya Message Queues ke zariye communicate karte hain.

```text
                       ┌──► [ Auth Service ] ──► [ Auth DB ]
                       │
[ API Gateway ] ───────┼──► [ Order Service ] ──► [ Order DB ]
                       │
                       └──► [ Payment Service ] ──► [ Payment DB ]
```

* **Pros**: 
  - **Independent Scaling**: Payment service slow ho to sirf Payment service scale karo!
  - **Fault Isolation**: Ek service down hone se puri app band nahi hoti.
  - **Tech Stack Freedom**: Auth Node.js me, Machine Learning Python me ban sakti hai.
* **Cons**: High deployment complexity, Network Latency, Distributed Data Management.

---

## � 4. Crucial System Metrics (NFRs)

Jab bhi interviewer System Design puchta hai, in 4 metrics par baat karna zaroori hai:

| Metric | What it means | Formula / Target |
| :--- | :--- | :--- |
| **Latency** | Ek single request process hone me kitna waqt lagta hai | Lower is better ($ms$) |
| **Throughput (QPS/RPS)** | System per second kitni requests handle kar sakta hai | Higher is better (e.g. 10k RPS) |
| **Availability** | System kitna percent online/active rehta hai | High Availability (99.99% = "4 Nines") |
| **Reliability** | Failures ke bawajood system data lose nahi karta | Fault Tolerant |

---

## � Key Takeaways (Piyush Garg Notes)

1. System design is all about **Trade-offs** (No single solution is perfect for everything).
2. Start simple (Monolith) and refactor to Microservices when scale demands it.
3. Always measure **Latency vs Throughput** before picking databases or caching layers.

