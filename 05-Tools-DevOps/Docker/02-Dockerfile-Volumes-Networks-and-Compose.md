# 🐳 Docker #2: Custom Images (Dockerfile), Volumes, Networks & Docker Compose
*(Notes from Apna College Complete Docker One-Shot & DataCamp Guide - Day 2)*

> **References**:
> - Video: [Docker Tutorial for Beginners - Complete One Shot (Apna College)](https://youtu.be/exmSJpJvIPs)
> - Article: [DataCamp: Docker Tutorial for Beginners](https://www.datacamp.com/tutorial/docker-tutorial)
> - **Core Question**: Apni custom application (Node.js/Python/React) ki Docker image kaise banayein, container crash hone par database data safe kaise rakhein, aur multi-container apps (App + DB + Redis) ko ek sath kaise orchestrate karein?

---

## 📝 1. Building Custom Images: The `Dockerfile`

**Dockerfile** ek plain text file hoti hai jisme step-by-step instructions hote hain ki application ki custom Docker image kaise build hogi.

### 📄 Production-Grade Node.js `Dockerfile`:
```dockerfile
# 1. Official lightweight Node.js Linux image select karo
FROM node:20-alpine

# 2. Container ke andar app directory bana kar usme move karo
WORKDIR /app

# 3. Pehle sirf package.json aur lock file copy karo (Layer Caching!)
COPY package*.json ./

# 4. Dependencies install karo
RUN npm install --production

# 5. Baki sara project source code copy karo
COPY . .

# 6. Environment variables set karo
ENV PORT=3000
ENV NODE_ENV=production

# 7. Port document karo
EXPOSE 3000

# 8. Container boot hone par execution command
CMD ["node", "server.js"]
```

---

### ⚡ Crucial Concept: Docker Layer Caching (Save Hours of Build Time!)

Docker har instruction (`FROM`, `COPY`, `RUN`) ko ek **Immutable Cached Layer** ki tarah save karta hai:

```text
Layer 1: [ FROM node:20-alpine ]           ──► (Cached ✅)
Layer 2: [ WORKDIR /app ]                  ──► (Cached ✅)
Layer 3: [ COPY package*.json ./ ]         ──► (Cached ✅ - Jab tak naya package add na ho)
Layer 4: [ RUN npm install ]               ──► (Cached ✅ - Instant skip!) ⚡
Layer 5: [ COPY . . ]                      ──► (Rebuilt only if code edits happen)
```

> ⚠️ **The Fatal Mistake**:  
> Agar aap `COPY . .` pehle likh dete aur `RUN npm install` baad me, to code me 1 word badalne par bhi Docker pura `npm install` dubara chalata (2-3 minute waste)!  
> **Golden Rule**: `package.json` pehle copy karo aur dependencies install karo, source code baad me copy karo!

---

### 🥊 `CMD` vs `ENTRYPOINT` (Interview Favorite)
* **`RUN`**: **Image Build Time** par chalta hai (e.g. `npm install`, packages download karna).
* **`CMD`**: **Container Run Time** par chalta hai. Agar user `docker run myapp npm start` likhega, to `CMD` override ho jayega!
* **`ENTRYPOINT`**: Fixed executable command hoti hai jo easily override nahi hoti (e.g. `ENTRYPOINT ["nginx", "-g", "daemon off;"]`).

---

## 💾 2. Data Persistence: Docker Volumes & Bind Mounts

Containers by default **Stateless (Ephemeral)** hote hain.  
*Agar container crash hua ya `docker rm` kiya, to container ke andar ka sara generated data (uploads, database tables) hamesha ke liye gayab ho jata hai!* 😱

```text
[ Docker Container (/data/db) ] 
              ║
              ╠══════► (Synced Storage Tunnel)
              ▼
[ Physical Host Machine (Named Volume / Directory) ] ──► Container mar bhi jaye, Data Safe! 🛡️
```

### 3 Types of Storage Mounts:

| Type | How It Works | Best Used For |
| :--- | :--- | :--- |
| **Named Volumes** | Docker khud host machine par safe directory manage karta hai (`/var/lib/docker/volumes/`) | **Databases (MongoDB, Postgres, MySQL)** |
| **Bind Mounts** | Aapke laptop ka exact folder path container ke folder se link hota hai (`d:/my-project:/app`) | **Local Development (Live hot reloading)** |
| **Anonymous Volumes**| Temporary volume with random hash name | Throwaway caches |

### 🛠️ Volume CLI Commands:
```bash
# Volume create karo
docker volume create mongo_data

# Volume verify karo
docker volume ls

# Database container volume ke sath run karo
docker run -d -p 27017:27017 -v mongo_data:/data/db --name db-server mongo:latest
```

---

## 🌐 3. Docker Networking (Connecting Microservices)

Jab aapke paas ek **Node.js Backend container** aur ek **MongoDB database container** hai, to wo aapas me baat kaise karenge?

```text
               ┌────────────────────────────────────────────────────────┐
               │              Custom Docker Bridge Network              │
               │                                                        │
[ Client ] ──► │  [ Node API Container ] ──► mongodb://mongo-db:27017 ──►│ ──► [ MongoDB Container ]
(Port 3000)    │       (my-backend)                                     │        (name: mongo-db)
               └────────────────────────────────────────────────────────┘
```

* **Default Bridge Network**: Containers direct container name se baat nahi kar sakte (DNS resolution off rehta hai).
* **Custom User-Defined Bridge Network**: Containers ek doosre ko **unke container name se call kar sakte hain**!

```bash
# 1. Custom network create karo
docker network create my-app-network

# 2. Database ko network me run karo
docker run -d --name mongo-db --network my-app-network mongo:latest

# 3. Backend app ko same network me connect karo
# Node app connection string ban jayegi: mongodb://mongo-db:27017/my_db
docker run -d -p 3000:3000 --name my-backend --network my-app-network my-node-image
```

---

## 🎻 4. Multi-Container Orchestration: Docker Compose

Jab application me Backend, Frontend, Database, aur Redis sab ek sath chalane ho, to 4 lambi `docker run` commands yaad rakhna mushkil hota hai.  
👉 **Docker Compose** ek single `docker-compose.yml` file se poore system ko ek click me run kar deta hai!

### 📄 Production `docker-compose.yml` Example:
```yaml
version: '3.8'

services:
  # 1. Web API Service
  backend:
    build: .
    container_name: express-api
    ports:
      - "3000:3000"
    environment:
      - PORT=3000
      - MONGO_URI=mongodb://mongo-db:27017/mydb
    depends_on:
      - mongo-db
    networks:
      - app-network

  # 2. Database Service
  mongo-db:
    image: mongo:latest
    container_name: mongo-db
    ports:
      - "27017:27017"
    volumes:
      - db_data:/data/db
    networks:
      - app-network

# Persistent Volumes
volumes:
  db_data:

# Shared Bridge Network
networks:
  app-network:
    driver: bridge
```

### 🚀 Docker Compose Commands:
```bash
# Background mode me saare containers build aur start karo
docker compose up -d

# Saare running services ka status dekhna
docker compose ps

# Saare containers, networks aur dependencies ko safely stop & remove karna
docker compose down

# Saare containers ke logs ek sath stream dekhna
docker compose logs -f
```

---

## 💡 Summary Checklist (Docker Masterclass)

- [x] **Dockerfile**: Declarative recipe for reproducible custom images.
- [x] **Layer Caching**: Place frequent code edits at the bottom; static dependencies at the top.
- [x] **Volumes**: Never run stateful databases without persistent volume mounts.
- [x] **Custom Networks**: Enables microservices to talk via container DNS names.
- [x] **Docker Compose**: Single configuration file to run full-stack multi-container stacks with `docker compose up -d`.

