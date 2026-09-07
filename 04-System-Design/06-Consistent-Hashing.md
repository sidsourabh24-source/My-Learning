# ⭕ System Design #6: Consistent Hashing
*(Notes from Piyush Garg System Design Series & ByteByteGo)*

> **References**:
> - Video: [Consistent Hashing - Piyush Garg](https://youtu.be/IC5Y1EE-aj4)
> - Deep-Dive Article: [ByteByteGo: Design Consistent Hashing](https://bytebytego.com/courses/system-design-interview/design-consistent-hashing)

---

## 🎯 1. The Core Problem: Why Traditional Hashing Fails

Imagine hamare paas **4 Cache Servers** ($S_0, S_1, S_2, S_3$) hain aur millions of user requests ko in servers me distribute karna hai.

### ❌ Traditional Modulo Hashing:
Hum standard formula use karte hain:
$$\text{Server Index} = \text{hash}(\text{key}) \pmod N$$
*(jahan $N = 4$ servers hain)*

* `hash("user:101") = 25` ➔ $25 \pmod 4 = \mathbf{1}$ ➔ Server $S_1$ me save hoga.
* `hash("user:102") = 30` ➔ $30 \pmod 4 = \mathbf{2}$ ➔ Server $S_2$ me save hoga.

---

### 💥 The Disaster (Server Addition or Crash):
Ab imagine karo **Server $S_1$ crash ho gaya** (ya naya server $S_4$ add ho gaya).  
Total servers ab $N = 3$ ho gaye!

* `hash("user:101") = 25` ➔ $25 \pmod 3 = \mathbf{1}$ (Different server!)
* `hash("user:102") = 30` ➔ $30 \pmod 3 = \mathbf{0}$ (Server badal gaya!)

```text
Server Crash (N=4 ➔ N=3)
           │
           ▼
[ ~99% Keys Ki Location Badal Gayi ] ──► [ Massive Cache Miss Storm! ⛈️ ]
                                                    │
                                                    ▼
                                    [ Primary Database Flooded & Crashed! 💥 ]
```

> **The Problem**: Traditional hashing me 1 server badalne se **almost 100% keys ka location change ho jata hai**, jisse poora cache invalidate ho jata hai!

---

## ⭕ 2. The Solution: Consistent Hashing (The Hash Ring)

MIT ke researchers (Karger et al.) ne **Consistent Hashing** invent kiya.

### 🕐 The Hash Ring Concept:
1. Hash function (e.g. SHA-1) ki output range hoti hai $0$ se $2^{160}-1$.
2. Is linear line ko mod kar ek **Gol Chakkar (Circular Ring)** bana dete hain, jahan $0$ aur $2^{160}-1$ aapas me jud jaate hain!

```text
                      [ 0 / 2^160-1 ]
                       /           \
               [ S0 ]                 [ S1 ]
              /                             \
             │            HASH               │
             │            RING               │
              \                             /
               [ S3 ]                 [ S2 ]
                       \           /
                       [ Mid-point ]
```

---

### 📍 How Keys are Mapped to Servers:
1. **Servers ko Ring par rakho**: Server ke IP ya Hostname ko hash karke ring par place karte hain (`hash("192.168.1.1")`).
2. **Keys ko Ring par rakho**: Data key ko hash karke ring par point karte hain (`hash("user_101")`).
3. **Lookup Rule (Clockwise Traversal)**:  
   Key ki location se **Clockwise (ghadi ki disha me)** ghumte jao. Jo **pehla server** milega, key usi server me store hogi!

```text
Key k1 ────► Clockwise search ────► Stores in Server S1
Key k2 ────► Clockwise search ────► Stores in Server S2
```

---

## 🔄 3. What Happens When a Server is Added or Removed?

Consistent Hashing ka asli magic yahi par hai!

### ➕ Adding a New Server ($S_{\text{new}}$):
Agar hum $S_0$ aur $S_1$ ke beech me naya server **$S_4$** add karte hain:
- Sirf $S_0$ aur $S_4$ ke beech wali choti si range ki keys $S_4$ par move hongi.
- **Baki saari keys ($S_1, S_2, S_3$) apni purani jagah par 100% safe rahengi!**

```text
Traditional Hashing:  ~100% keys relocated (System breakdown)
Consistent Hashing:   Only k / N keys relocated (Smooth scaling) ⚡
```

### ➖ Removing a Server ($S_1$ Crashes):
Agar $S_1$ crash ho jaye, to sirf $S_1$ ki keys clockwise next server ($S_2$) par transfer hongi. Baki poora system unbothered rahega!

---

## ⚠️ 4. The Two Big Flaws in Basic Consistent Hashing

Consistent Hashing paper me achi lagti hai, par real world me iske 2 khatarnak issues hain:

### Flaw 1: Non-Uniform Partitions (Unequal Slices)
Servers random hash values par girte hain. Ho sakta hai $S_0$ aur $S_1$ bohot paas ho jayein, aur $S_1$ aur $S_2$ ke beech aadha ring khali ho. Ek server par 80% load chala jayega aur baki free baithe rahenge!

### Flaw 2: The Domino / Cascading Crash 🎳
Agar Server $S_1$ par load badha aur wo crash hua:
- $S_1$ ka sara load immediate agle server ($S_2$) par shift ho jayega.
- $S_2$ double load handle nahi kar payega aur wo bhi crash ho jayega.
- Ek-ek karke saare servers domino tiles ki tarah crash ho jayenge!

---

## 🌟 5. The Master Solution: Virtual Nodes (Vnodes / Tokens)

ByteByteGo aur Piyush Garg explain karte hain ki production me hum **Virtual Nodes** use karte hain:

```text
Instead of 1 Physical Server = 1 Point on Ring...
1 Physical Server = 100 to 200 Virtual Points on Ring!
```

* Server 1 ke multiple copies banate hain: `S1_0`, `S1_1`, `S1_2`, ..., `S1_199`.
* Server 2 ke multiple copies: `S2_0`, `S2_1`, `S2_2`, ..., `S2_199`.

```text
                       [ S1_0 ]
                    /            \
             [ S2_1 ]            [ S3_0 ]
            /                              \
       [ S3_1 ]         VIRTUAL             [ S1_1 ]
       │                 NODES                     │
       [ S1_2 ]          RING               [ S2_0 ]
            \                              /
             [ S3_2 ]            [ S2_2 ]
                    \            /
                       [ S1_3 ]
```

### 🚀 Why Virtual Nodes Win:
1. **Perfect Load Balance**: Ring par har server ke tukde evenly fail jate hain. Standard deviation 5% tak gir jata hai (Balanced traffic).
2. **No Cascading Crash**: Agar Server 1 fail hua, to uske 100 virtual nodes the jo alag-alag jagah the. Load kisi ek server par nahi balki **saare remaining servers me equally divide** ho jayega!

---

## 📊 Summary Comparison: Modulo vs Consistent Hashing

| Feature | Traditional Modulo Hashing | Consistent Hashing with Vnodes |
| :--- | :--- | :--- |
| **Formula** | `hash(key) % N` | Hash Ring + Clockwise Traversal |
| **Impact of Server Scaling** | **Disastrous**: ~100% keys remapped | **Minimal**: Only $\frac{k}{N}$ keys remapped |
| **Cache Stampede Risk** | 🔴 Extreme (DB will crash) | 🟢 Zero risk (Smooth transition) |
| **Hotspot Handling** | Poor | 🟢 Handled via Virtual Nodes |
| **Industry Adoption** | Small single-server apps | **Amazon DynamoDB, Cassandra, Discord, Akamai CDN** |

---

## 💡 Real-World Production Systems Using Consistent Hashing

1. **Amazon DynamoDB & Apache Cassandra**: Data partitions ko nodes par scale karne ke liye consistent hashing use karte hain.
2. **Discord**: Har user/guild ko Elixir voice/text nodes par route karne ke liye consistent hashing ring use hoti hai.
3. **Akamai & Cloudflare CDN**: Web caching me URL request ko nearest edge cache server par map karne ke liye.

