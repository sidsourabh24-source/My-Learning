# 📜 System Design #3: Event Sourcing Pattern
*(Notes from Piyush Garg System Design Series - Part 3)*

> **Core Philosophy**: State ko directly update/overwrite karne ke bajaye, har state change ko ek **Immutable Event (Log)** ki tarah store karo. Past ko kabhi delete ya modify mat karo!

---

## 🎯 1. What is Event Sourcing? (Traditional CRUD vs Event Sourcing)

Ab tak hum web development me **CRUD (Create, Read, Update, Delete)** use karte aaye hain:

### ❌ Traditional CRUD Approach (Direct Overwrite)
Agar user ke wallet me balance change hota hai:
```sql
UPDATE users SET balance = 500 WHERE id = 1;
```
* **Problem**: Purana balance kya tha? Kisne kab add ya deduct kiya? History gayab! Sirf "Current State" bachi rehti hai.

---

### ✅ Event Sourcing Approach (Append-Only Event Log)
Event Sourcing me hum **Current State store hi nahi karte**! Hum har chote-bade event ko ek **Append-Only Event Store** me append karte hain:

```text
Event 1: [ AccountCreated: ₹0 ]
Event 2: [ MoneyDeposited: +₹1000 ]
Event 3: [ SwiggyOrderPlaced: -₹300 ]
Event 4: [ MovieTicketBooked: -₹200 ]
---------------------------------------
Current State (Calculated): ₹500
```

* **Bank Passbook Analogy (Best Example)**:
  Bank aapka balance direct magic ki tarah update nahi karta. Bank passbook me har credit aur debit ka transaction print hota hai. Passbook ke saare transactions calculate karke final balance banta hai!

---

## 🔄 2. Rehydration & The Snapshot Pattern

### 💧 Rehydration (Replaying the Events)
Jab bhi user ka current state chahiye hota hai, system Event Store se saare past events padhta hai aur sequentially apply (replay) karke current state nikalta hai. Is process ko **Rehydration** kehte hain.

```text
[ Event 1 (+1000) ] ➔ [ Event 2 (-300) ] ➔ [ Event 3 (-200) ] ══► (Rehydration) ══► [ Current Balance: ₹500 ]
```

---

### 📸 The Performance Problem & Snapshots
* **Problem**: Agar kisi user ke account me **1,00,000 transactions** hain, to har baar login par 1 Lakh events replay karna server ko slow kar dega!
* **Solution: Snapshots**:
  Har fixed interval (e.g. har 1,000 events ya har midnight) par current state ka ek **Snapshot** save kar liya jata hai:

```text
[ Events 1 to 10,000 ] ──► [ 📸 Snapshot taken at Event 10,000: Balance = ₹45,000 ]
                                      │
                                      ▼
             [ Only Replay Events 10,001 to 10,015 ] ──► [ Current Balance: ₹46,200 ]
```
* **Result**: Poore 1 Lakh events replay nahi karne padte, sirf last snapshot ke baad ke naye events replay hote hain! ⚡

---

## ⚖️ 3. Pros vs Cons of Event Sourcing

| Feature | Traditional CRUD | Event Sourcing |
| :--- | :--- | :--- |
| **Audit Trail** | ❌ Nahi hota (Extra audit table banani padti hai) | **100% Complete History Built-in** |
| **Time Travel** | ❌ Past state dekhna impossible | **Kisi bhi past date ki exact state dekh sakte ho** |
| **Data Loss** | ⚠️ High risk (Overwrites & hard deletes) | **Zero Loss (Append-only, immutable)** |
| **Complexity** | Simple & Straightforward | High learning curve, requires Event Store |
| **Read Latency** | Direct read fast hota hai | Replay overhead (requires Snapshots & CQRS) |

---

## 🚀 4. Real-World Tech Stack for Event Sourcing

1. **Event Store**: Specialized databases like **EventStoreDB** ya **Apache Kafka** / **AWS Kinesis** (jahan events append-only log me order maintain karke store hote hain).
2. **CQRS Companion**: Event Sourcing ko practical banane ke liye **CQRS (Command Query Responsibility Segregation)** ke sath pair kiya jata hai (Part 4 me deep dive karenge!).

---

## 💡 Key Architectural Takeaway

> Event Sourcing banking, financial ledger systems, e-commerce order lifecycles, aur healthcare me use hota hai jahan **Audit Compliance** aur **Data History** critical hoti hai!

