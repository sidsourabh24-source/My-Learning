# 🚀 System Design #1: Scalable System Architecture
*(Notes from Piyush Garg System Design Series - Video #1)*

This guide breaks down the fundamental components of building a robust, scalable, and fault-tolerant system inspired by real-world production architectures (like Amazon).

---

## 🌐 1. The Foundation: Client & Server

The simplest system consists of a **Client** (user device) and a **Server** (24/7 machine).

```text
[Client]  -->  [DNS Server]  -->  [Server (IP: 10.2.3.4)]
   │               │                    │
   │ (amazon.com)  │ (Lookup IP)        │ (Process Request)
```

- **Problem**: Servers have physical IP addresses (e.g. `10.2.3.4`) which are hard to remember.
- **Solution**: **DNS (Domain Name System)** acts as a global directory translating human-friendly names (`amazon.com`) to machine IPs (`10.2.3.4`).

---

## 📈 2. Scaling Strategies: Vertical vs Horizontal

As traffic grows, a single server becomes a bottleneck. There are two ways to scale:

| Strategy | 📈 Vertical Scaling (Scale Up) | 📉 Horizontal Scaling (Scale Out) |
| :--- | :--- | :--- |
| **How it works** | Adding more CPU/RAM to existing server | Adding more servers to the system |
| **Downtime** | Often requires server restart (Downtime) | **Zero Downtime** (Add servers while running) |
| **Limits** | Maximum hardware capacity exists | Unlimited elastic capacity |
| **Fault Tolerance** | Low (Single Point of Failure) | **High** (If one server fails, others handle load) |
| **Requirement** | Simple (No extra software) | Requires a **Load Balancer** |

---

## 🔄 3. Load Balancers & Microservices

When using Horizontal Scaling, you cannot point DNS to multiple IPs directly. You need a middleman.

### ⚖️ Load Balancer (LB)
A single entry point that distributes incoming requests across multiple backend servers.

```text
          [Load Balancer] (IP: 10.2.3.7)
          /      |      \
         /       |       \
   [Server 1] [Server 2] [Server 3]
```
- **Algorithms**:
  - **Round Robin**: Distributes requests equally (1st to $S_1$, 2nd to $S_2$, etc.).
  - **Health Checks**: Ensures traffic only goes to healthy, active servers.
- **AWS Terminology**: ELB (Elastic Load Balancer).

---

### 🏗️ Microservices Architecture & API Gateway
Instead of one monolithic app, the system is split into smaller, independent services (Auth, Orders, Payments).

```text
[Client] --> [API Gateway] --> [Router Rules]
                              /    |    \
                     [Auth LB] [Order LB] [Payment LB]
```
- **API Gateway**: A centralized entry point that routes requests based on rules (path, method, headers). Routes `/auth` ➔ Auth LB, `/orders` ➔ Order LB. Handles security and authorization.

---

## ⚡ 4. Asynchronous Processing & Queues

Synchronous calls (waiting for a response) can slow down the system. For heavy tasks (emails, bulk uploads), use **Asynchronous Processing**.

### 💬 Message Queues (e.g. AWS SQS / BullMQ)
A FIFO (First-In-First-Out) buffer where producers push messages and consumers pull them.
- **Decoupling**: Payment service doesn't wait for email service to finish.
- **Rate Limiting**: Smooths traffic if an external API (like Gmail) has rate limits (10 emails/sec).

---

### 📢 Pub/Sub Models (e.g. AWS SNS)
Publish-Subscribe allows one event to trigger multiple actions simultaneously.
- **Scenario**: User pays ➔ Send Email, SMS, WhatsApp, and Update DB.
- **How it works**: Payment Service publishes to a Topic; all subscribed services receive the message.

---

## 🔄 5. Fan-Out Architecture (Best of Both Worlds)

Combines Pub/Sub (SNS) with Queues (SQS) for reliability and multi-action triggers.

```text
[Payment Service] 
      │
      ▼ (Publish)
[ SNS Topic ] <─── (Fan Out)
      │ ╲      ╲
      │  ╲      ╲
      ▼   ▼      ▼
[Email Q] [SMS Q] [DB Q]
  │       │       │
(Worker) (Worker) (Worker)
```
- **Pattern**: Event triggers SNS ➔ SNS Fan-Out pushes message to multiple SQS Queues ➔ Each Queue has its own Worker.
- **Dead Letter Queue (DLQ)**: Failed messages are moved here for manual inspection after retries.

---

## 💎 6. Caching & Database Scaling

### 🚀 Caching (e.g. Redis)
Store frequently accessed data in memory (RAM).
- **Flow**: Check Cache ➔ If Hit, return data. If Miss, query DB ➔ Store in Cache ➔ Return data.

### 🗄️ Database Replication
- **Primary Node**: Handles all Writes and real-time reads.
- **Read Replicas**: Handle Read-only queries (Analytics, Logs).

---

## 🌍 7. Content Delivery Network (CDN / CloudFront)

Serving static content (images, videos, CSS) from the user's nearest geographic location.

```text
[User in India] --> [CloudFront Edge (India)] --> [Cache Hit] (Fast 10ms)
       │
       │ (If Miss)
       ▼
[Load Balancer] --> [Origin Server (US)]
```

---

## 🛡️ 8. Rate Limiting & Security

Protecting the system from abuse (DDoS, bots) by limiting requests per user/IP (e.g. 5 requests/sec).
- **Token Bucket**: Allows traffic bursts but averages out over time.
- **Leaky Bucket**: Smooths out traffic to a constant rate.

---

## 📝 Summary Checklist for Scalable Systems

- [x] **DNS**: Resolves domain names to IPs.
- [x] **Horizontal Scaling**: Adds servers instead of upgrading hardware.
- [x] **Load Balancer**: Distributes traffic across servers.
- [x] **API Gateway**: Routes requests to specific microservices.
- [x] **Message Queues (SQS)**: Decouples services for async processing.
- [x] **Pub/Sub (SNS)**: Triggers multiple actions from one event.
- [x] **Fan-Out Pattern**: Ensures reliable multi-service notification.
- [x] **Caching (Redis)**: Reduces DB load and latency.
- [x] **DB Replication**: Separates Read/Write operations.
- [x] **CDN (CloudFront)**: Serves static content from edge locations.
- [x] **Rate Limiting**: Protects against abuse.
