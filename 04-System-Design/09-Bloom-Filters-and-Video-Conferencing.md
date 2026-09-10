# 🧠 System Design #9: Bloom Filters & Multi-Party Video Calls (WebRTC, MCU, SFU)
*(Notes from Piyush Garg System Design Series)*

> **Video References**:
> - Part 1: [What are Bloom Filters? - Piyush Garg](https://youtu.be/vz0QUa4CS3o)
> - Part 2: [System Design of Multi-Conference Video Calls - Piyush Garg](https://youtu.be/Zaz6hYVm-WE)

---

# PART 1: 🌸 Bloom Filters (Probabilistic Data Structure)

## ❓ The Core Problem: Expensive Database Searches

Jab naya user Instagram ya Twitter par sign up karta hai aur username type karta hai (e.g. `rahul_dev`), to system ko check karna padta hai: *"Kya ye username already taken hai?"*

### ❌ The Naive DB Approach:
```sql
SELECT * FROM users WHERE username = 'rahul_dev';
```
* Agar **500 Million users** hain, to har single keystroke par Disk DB query karna database ko choke kar dega.
* RAM me 500 Million usernames ka Hash Set rakhna **GBs of expensive RAM** consume karega!

---

## 💡 The Bloom Filter Solution

**Bloom Filter** ek extremely space-efficient **Probabilistic Data Structure** hai jo batata hai ki koi item set ke andar exist karta hai ya nahi.

### 🌟 The Golden Rule of Bloom Filters:
1. **"Definitely NO"**: Agar Bloom Filter ne kaha item nahi hai, to wo **100% Guaranteed nahi hai** (Zero False Negatives)!
2. **"Probably YES"**: Agar Bloom Filter ne kaha item hai, to **ho sakta hai item ho, ya false alarm (False Positive) ho**!

```text
Incoming Query: "rahul_dev"
              │
              ▼
    [ Bloom Filter Check ]
     ├── Returned "NO"  ──► 100% Sure! Username Available hai (Zero DB hit!) ⚡
     └── Returned "YES" ──► Shayad hai ➔ Verify in Actual Database 🐌
```

---

## ⚙️ How Bloom Filters Work Internally (Bit Array + Hash Functions)

Bloom Filter me ek **Bit Array** (0 aur 1) aur **$k$ Hash Functions** hote hain:

```text
Bit Array (Initially all 0):
Index:  [ 0 ][ 1 ][ 2 ][ 3 ][ 4 ][ 5 ][ 6 ][ 7 ][ 8 ][ 9 ]
Bits:     0    0    0    0    0    0    0    0    0    0
```

### 1️⃣ Adding an Element (`"alex"`):
Element ko 3 alag-alag hash functions se pass karo:
* $h_1(\text{"alex"}) = 2$
* $h_2(\text{"alex"}) = 5$
* $h_3(\text{"alex"}) = 8$
Indexes 2, 5, aur 8 par bit ko **1** set kar do:

```text
Bits:     0    0    [1]   0    0   [1]   0    0   [1]   0
                     ↑              ↑              ↑
                    h1             h2             h3
```

---

### 2️⃣ Querying an Element:
* **Check `"alex"`**: Indexes 2, 5, 8 check karo. Saare bits `1` hain ➔ **"Probably YES"**.
* **Check `"john"`**: $h_1, h_2, h_3$ output index 1, 5, 9. Index 1 par bit `0` hai!  
  Agar ek bhi bit `0` hai, to element **100% kabhi insert nahi hua tha** ➔ **"Definitely NO"**!

---

### ⚠️ Why False Positives Happen (Collision):
Agar multiple elements ke hash outputs coincidentally wahi bits 1 kar dete hain jo ek naye element ke hain, to Bloom filter kahega "MIGHT EXIST", jabki wo actually DB me nahi tha!
* **Trade-off**: Bit array ka size bada karke aur optimal hash functions ($k$) choose karke False Positive rate ko **<1%** kiya ja sakta hai.
* **Why Deletion is NOT Allowed**: Kisi bit ko `0` karoge to doosre elements ke shared bits corrupt ho jayenge!

---

## 🏭 Real-World Production Use Cases:
1. **Database Disk I/O Savings**: **Cassandra, RocksDB, Bigtable** pehle Bloom Filter check karte hain. Agar "NO" mila, to slow disk SSTable read kiye bina instant return ho jata hai!
2. **Browser Protection**: Google Chrome check karta hai ki URL malicious websites ki list me hai ya nahi.
3. **Cache Penetration Protection**: Non-existent keys ki fake requests ko Redis/DB tak pahunchne se pehle hi filter out karna.

---
---

# PART 2: 📹 Multi-Party Video Calls (WebRTC, MCU vs SFU)

Zoom, Google Meet, ya Discord par **10 se 50 log ek sath video call** kaise karte hain?

---

## 🌐 1. WebRTC Basics & The 1-on-1 Call (P2P)

**WebRTC (Web Real-Time Communication)** browsers aur mobile devices ko audio/video stream exchange karne deta hai:

```text
[ Alice ] ◄════ (Direct P2P UDP Encrypted Video Stream) ════► [ Bob ]
   │                                                             │
   └───► [ Signaling Server (WebSockets) ] ◄─────────────────────┘
         (Only for exchanging SDP & ICE Candidates / IP addresses)
```
* **Signaling Server**: Sirf shuruat me dono devices ko aapas me connect karne ke liye network info (IPs/Ports via STUN/TURN) exchange karwata hai.
* **Stream Flow**: Media streams directly device-to-device (Peer-to-Peer) flow karti hain bina kisi media server ke.

---

## 💥 2. The Disaster of Mesh Architecture in Group Calls

Agar hum 1-on-1 P2P concept ko **5 logo ke group call** me use karein (Mesh Network):

```text
              [ User 1 ]
             /    |    \
     [ User 2 ]───┼───[ User 3 ]
             \    |    /
         [ User 4 ]─[ User 5 ]
```

### 🔴 The $O(N^2)$ Bandwidth Explosion:
Har participant ko baki saare $(N-1)$ logo ko apni video stream upload karni padegi aur un sabki download karni padegi!
* 10 logo ki call me: **Each client uploads 9 streams and downloads 9 streams!**
* **Mobile phone overheat ho jayega**, home Wi-Fi choke ho jayega, aur call crash ho jayegi!

---

## 🎛️ 3. Architecture 2: MCU (Multipoint Control Unit) - The Mixer

Server-side solution jisme ek central server saare video streams ko handle karta hai.

```text
[ Client 1 ] ──(Upload 1 Stream)──► ┌────────────────────────────────┐
[ Client 2 ] ──(Upload 1 Stream)──► │     MCU Media Server           │ ──(Single Combined Stream)──► [ Client 1 ]
[ Client 3 ] ──(Upload 1 Stream)──► │ • Decodes all streams          │ ──(Single Combined Stream)──► [ Client 2 ]
[ Client 4 ] ──(Upload 1 Stream)──► │ • Composites into 1 Video Grid │ ──(Single Combined Stream)──► [ Client 3 ]
                                    │ • Re-encodes full frame video  │
                                    └────────────────────────────────┘
```

* **Pros**: Client bandwidth super low hai (Har client sirf 1 upload aur 1 composite download karta hai).
* **Cons (Massive Downsides)**:
  - 💸 **Crazy Server CPU Costs**: Server par 10 logo ki high-res video decode karke, grid bana kar re-encode karna bohot expensive GPU/CPU maangta hai.
  - ⏱️ **High Latency**: Real-time re-encoding me 200ms-500ms delay add ho jata hai.
  - ❌ **Zero Client Flexibility**: Client screen layout (e.g. pin a speaker, change grid size) customize nahi kar sakta kyunki video server se pre-stitched aati hai.

---

## 🚀 4. Architecture 3: SFU (Selective Forwarding Unit) - The Modern Standard

**Zoom, Google Meet, Discord, aur Microsoft Teams** sabhi **SFU** architecture use karte hain!

```text
[ Client 1 ] ──(Uploads 1 Stream)──► ┌─────────────────────────────────────────┐
                                     │        SFU Media Server                 │ ──(Forwards C1)──► [ Client 2 ]
[ Client 2 ] ──(Uploads 1 Stream)──► │ • Does NOT decode/re-encode video       │ ──(Forwards C1)──► [ Client 3 ]
                                     │ • Acts as a High-Speed Packet Router    │ ──(Forwards C1)──► [ Client 4 ]
[ Client 3 ] ──(Uploads 1 Stream)──► │ • Forwards streams based on client need │
                                     └─────────────────────────────────────────┘
```

### 🌟 Why SFU is the Undisputed Winner:
1. **Lightweight Server Processing**: SFU video ko decode/re-encode nahi karta! Wo sirf incoming UDP media packets ko inspect karke doosre participants ko route karta hai. Server CPU usage super low rehti hai!
2. **Flexible Client UI**: Har client ko alag streams milti hain, isiliye client apne hisab se active speaker ko spotlight ya grid layout me render kar sakta hai.
3. **Simulcast Support**:
   - Client apni video 3 quality me upload karta hai: Low (180p), Medium (360p), High (720p).
   - SFU smart decision leta hai: Jo bol raha hai (Active Speaker) uski 720p stream baki sabko bhejta hai, aur baki chup baithe logo ki 180p thumbnail stream bhejta hai! Bandwidth saved! ⚡

---

## 📊 Summary Comparison: Mesh vs MCU vs SFU

| Architecture | Client Upload | Client Download | Server CPU Load | End-to-End Latency | Best Used For |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Mesh (P2P)** | 🔴 $N-1$ Streams | 🔴 $N-1$ Streams | 🟢 Zero (No Server) | 🟢 Lowest (<50ms) | 1-on-1 calls (WhatsApp) |
| **MCU** | 🟢 1 Stream | 🟢 1 Stream | 🔴 Extremely High | 🔴 High (200-400ms) | Legacy hardware telepresence |
| **SFU** | 🟢 1 Stream (Simulcast) | 🟡 $N-1$ Streams | 🟢 Low / Scalable | 🟢 Super Low (<100ms) | **Zoom, Google Meet, Discord** |

---

## 💡 Key Architectural Takeaways

1. **Bloom Filters**: Don't waste disk IO on things that don't exist. Bit arrays + hash functions provide an instant memory shield.
2. **Video Conferencing**: Mesh fails beyond 3 users. MCU is too costly and rigid. **SFU + WebRTC Simulcast** is the industry standard for low-latency, scalable group calls!

