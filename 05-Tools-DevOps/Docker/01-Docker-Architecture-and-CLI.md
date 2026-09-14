# 🐳 Docker #1: Architecture, Core Concepts & CLI Commands
*(Notes from Apna College Complete Docker One-Shot & DataCamp Guide - Day 1)*

> **References**:
> - Video: [Docker Tutorial for Beginners - Complete One Shot (Apna College)](https://youtu.be/exmSJpJvIPs)
> - Article: [DataCamp: Docker Tutorial for Beginners](https://www.datacamp.com/tutorial/docker-tutorial)
> - **Core Philosophy**: *"Build Once, Run Anywhere"* — Software development ka sabse bada headache *"Bhai mere computer pe chal raha tha, tere computer/production pe kyu fail ho gaya?"* ko jad se khatam karna!

---

## ❓ 1. Why Docker? (The Developer Nightmare)

Pehele jab koi team software develop karti thi:
* Developer 1 ke laptop par **Node.js v20** aur **Ubuntu** tha.
* Developer 2 ke paas **Windows** aur missing Python/C++ build tools the.
* Production server par **Node.js v16** aur different Linux packages the.
* **The Result**: Environment mismatches, dependency conflicts, aur deployment failures!

```text
[ Dev Laptop (Ubuntu / Node 20) ] ──► "Works fine here! 😃"
                  │
                  ▼ (Deploy Code)
[ Production Server (CentOS / Node 16) ] ──► Crash! Missing Libraries! 💥
```

### 💡 The Solution: Containerization
Docker aapke **Application Code + Runtime (Node/Python) + Dependencies (Libraries) + Configuration Files** sabko ek chote isolated package (Container) me seal kar deta hai. Ab ye container chahe Windows par chale, Mac par, ya AWS Cloud par—**har jagah exact identical behave karega**!

---

## 🥊 2. Virtual Machines (VMs) vs Docker Containers

Interviewers ka number-one favorite question!

```text
┌─────────────────────────────────┐       ┌─────────────────────────────────┐
│        Virtual Machine          │       │        Docker Container         │
│  [ App 1 ]           [ App 2 ]  │       │  [ App 1 ]           [ App 2 ]  │
│  [ Guest OS (3GB) ]  [ Guest OS]│       │  [ Isolated Binaries & Libs ]   │
│  [ Hypervisor (Type 1 or 2)   ] │       │  [ Docker Engine Daemon ]       │
│  [ Host Operating System ]      │       │  [ Shared Host OS Kernel ]      │
│  [ Physical Hardware (CPU/RAM) ]│       │  [ Physical Hardware (CPU/RAM) ]│
└─────────────────────────────────┘       └─────────────────────────────────┘
```

| Comparison Metric | Virtual Machine (VM) | Docker Container |
| :--- | :--- | :--- |
| **Architecture** | Har VM ke andar apna **Complete Guest OS (3-5 GB)** hota hai | Host machine ka **OS Kernel share** karta hai |
| **Size** | Gigabytes (4 GB – 10 GB) | **Megabytes (50 MB – 300 MB)** ⚡ |
| **Boot Time** | Minutes (Pura OS boot hone ka time leta hai) | **Milliseconds (Instant like starting a process)** |
| **Resource Usage**| High RAM & CPU overhead (Fixed allocation) | Dynamic / On-demand resource consumption |
| **Density** | Ek system par 2-4 VMs chalao to laptop hang | Ek system par **20-50 containers** smoothly chal sakte hain |

---

## 🏛️ 3. Docker Architecture & Components

Docker **Client-Server (Daemon)** architecture follow karta hai:

```text
┌────────────────┐                     ┌────────────────────────────────────────┐                     ┌────────────────────┐
│ Docker Client  │                     │              Docker Host               │                     │  Docker Registry   │
│                │                     │                                        │                     │    (Docker Hub)    │
│  $ docker run  │ ──(REST API Call)──►│  [ Docker Daemon (dockerd) ]           │ ──(docker pull)────►│                    │
│  $ docker pull │                     │    ├── Manages Images (nginx, node)    │                     │  Public & Private  │
│  $ docker build│                     │    ├── Manages Containers (App 1, App2)│ ◄─(Downloads Img)──│  Image Repository  │
│                │                     │    └── Manages Networks & Volumes      │                     │                    │
└────────────────┘                     └────────────────────────────────────────┘                     └────────────────────┘
```

### The 3 Golden Terms:
1. **Docker Image (The Blueprint)**: Read-only template jisme code, libraries, aur runtime files packed hoti hain (Jaise OOP me **Class** hoti hai).
2. **Docker Container (The Running Instance)**: Us image ka live, running instance jisme actual execution hoti hai (Jaise OOP me **Object** hota hai).
3. **Docker Hub / Registry**: Online library jahan se pre-built official images (Ubuntu, Node, MongoDB, Redis, Python) download karte hain (Jaise code ke liye **GitHub** hai).

---

## 🛠️ 4. Master Docker CLI Cheat Sheet (Day 1 Practical)

### 🚀 A. Running & Managing Containers
```bash
# 1. Image pull karke detached (background) mode me container run karna
docker run -d -p 8080:80 --name my-web-server nginx

# 2. Interactive terminal ke sath container me ghusna (e.g. Ubuntu)
docker run -it ubuntu /bin/bash

# 3. Running containers list karna
docker ps

# 4. Saare containers (Running + Stopped) list karna
docker ps -a

# 5. Container stop aur start karna
docker stop my-web-server
docker start my-web-server

# 6. Container delete karna (Pehle stop hona chahiye)
docker rm my-web-server

# 7. Running container ko forcefully delete karna
docker rm -f my-web-server
```

---

### 📦 B. Managing Images
```bash
# Local system ki saari downloaded images dekhna
docker images

# Docker Hub se image download karna
docker pull mongo:latest

# Unused image ko delete karna
docker rmi nginx

# Saare unused/dangling containers, networks, aur images ko ek sath clean karna
docker system prune -f
```

---

### 🔍 C. Logs, Debugging & Execution
```bash
# Live logs stream dekhna (Ctrl+C to exit)
docker logs -f my-web-server

# Running container ke andar naya terminal shell open karna
docker exec -it my-web-server /bin/sh

# Container ki detailed internal configuration (IP, Mounts, Ports) dekhna
docker inspect my-web-server
```

---

## 🌐 5. Understanding Port Mapping (`-p 8080:80`)

Containers isolated virtual network me chalte hain. Aapka computer bahar se container ke port ko tab tak access nahi kar sakta jab tak aap **Port Forwarding** na karo:

```text
[ Aapka Laptop Browser: localhost:8080 ] 
                 │
                 ▼
     (Host Machine Port: 8080)
                 │  
                 ▼  [-p 8080:80]
  (Container Internal Port: 80)
                 │
                 ▼
        [ Nginx Web Server ]
```
* **Syntax**: `-p <Host_Machine_Port>:<Container_Internal_Port>`
* Example: `-p 3000:5000` ka matlab hai browser par `localhost:3000` kholo, traffic container ke port `5000` par forward hoga!

---

## 💡 Key Takeaways (Day 1)

1. **Containers are lightweight processes**: Ye koi heavy operating system nahi hain, balki Linux kernel ke **Namespaces** (isolation) aur **Cgroups** (resource limits) par chalne wale smart processes hain.
2. **Images are immutable**: Ek baar image build ho gayi, to wo read-only hoti hai. Container us image ke upar ek choti si writable layer add karta hai.

