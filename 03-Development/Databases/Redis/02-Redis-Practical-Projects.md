# 🛠️ Redis Practical Projects (Express + ioredis)
*(Chai aur Code Playlist - Production-Level Implementations)*

---

## 🟢 Project 1: Site Banner API Caching

### ❓ The Problem
E-commerce site par home page banner data har second update nahi hota. Lekin jab millions of users app open karte hain, tab har user ke liye primary database (MongoDB / PostgreSQL) me heavy query chalne se DB server crash ho sakta hai.

### 💡 The Solution (Cache Hit vs Cache Miss Architecture)
1. Jab request aati hai, pehle **Redis RAM** me key check karo (`site_banners`).
2. Agar data mil jata hai ➔ **Cache Hit** ⚡ (Direct Redis se return karo in <1 ms).
3. Agar data nahi milta ➔ **Cache Miss** 🐌 (Primary DB se query karo, response Redis me 60s TTL ke sath store karo, aur user ko bhejo).

---

### 💻 Full Node.js Code Implementation

```javascript
const express = require('express');
const Redis = require('ioredis');

const app = express();
const redis = new Redis(); // Connects to 127.0.0.1:6379

// Simulated Slow Primary Database Query
async function getBannersFromDatabase() {
  console.log('🐌 Fetching from Primary Database (Simulated 2 sec delay)...');
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 1, title: 'Diwali Mega Sale 50% OFF', image: '/banners/diwali.jpg' },
        { id: 2, title: 'Chai aur Code Backend Batch', image: '/banners/chai.jpg' }
      ]);
    }, 2000);
  });
}

// Banner API Route with Caching Layer
app.get('/api/v1/site-banners', async (req, res) => {
  const cacheKey = 'site_banners';

  try {
    // Step 1: Check Redis Cache First
    const cachedBanners = await redis.get(cacheKey);

    if (cachedBanners) {
      console.log('⚡ CACHE HIT! Returning data instantly from Redis RAM');
      return res.json({
        source: 'redis-cache',
        data: JSON.parse(cachedBanners)
      });
    }

    // Step 2: Cache Miss -> Fetch from DB
    console.log('❌ CACHE MISS! Fetching from Database...');
    const banners = await getBannersFromDatabase();

    // Step 3: Save data into Redis RAM with 60 seconds Expiry ('EX')
    await redis.set(cacheKey, JSON.stringify(banners), 'EX', 60);

    return res.json({
      source: 'primary-database',
      data: banners
    });

  } catch (error) {
    console.error('Redis API Error:', error);
    return res.status(500).json({ error: 'Server Error' });
  }
});

app.listen(3000, () => console.log('Banner Service running on http://localhost:3000'));
```

---

## 🟡 Project 2: OTP Verification & Cooldown System

### ❓ The Problem
Jab user Signup ya Forgot Password karta hai:
1. OTP **5 minutes** ke baad automatically expire ho jaana chahiye.
2. User ko spamming se rokne ke liye **60-second cooldown period** (Rate Limiting) lagna chahiye.

### 💡 The Strategy (Using Redis TTL Expiry)
- **Key 1 (`otp:<phone>`)**: Store 6-digit OTP code with **300 seconds TTL (5 mins)**.
- **Key 2 (`cooldown:<phone>`)**: Store active flag with **60 seconds TTL (1 min)**.

---

### 💻 Full Node.js Code Implementation

```javascript
const express = require('express');
const Redis = require('ioredis');

const app = express();
app.use(express.json());
const redis = new Redis();

// Helper: Generate Random 6-Digit OTP
function generateOTP() {
  return Math.floor(100000 + Math.random() * 900000).toString();
}

// 1. Send OTP Route (With Rate Limiting & Auto Expiry)
app.post('/api/v1/auth/send-otp', async (req, res) => {
  const { phone } = req.body;
  if (!phone) return res.status(400).json({ error: 'Phone number is required' });

  const otpKey = `otp:${phone}`;
  const cooldownKey = `cooldown:${phone}`;

  // Rate Limiting Check: Cooldown active hai?
  const isCooldownActive = await redis.get(cooldownKey);
  if (isCooldownActive) {
    return res.status(429).json({
      error: 'Please wait 60 seconds before requesting a new OTP.'
    });
  }

  // Generate OTP
  const otp = generateOTP();

  // Store OTP in Redis for 5 minutes (300 seconds)
  await redis.set(otpKey, otp, 'EX', 300);

  // Set Cooldown Flag in Redis for 60 seconds
  await redis.set(cooldownKey, 'ACTIVE', 'EX', 60);

  console.log(`📩 SMS Sent to ${phone}: OTP = ${otp}`);
  return res.json({ message: 'OTP sent successfully. Valid for 5 minutes.' });
});

// 2. Verify OTP Route (One-Time Verification)
app.post('/api/v1/auth/verify-otp', async (req, res) => {
  const { phone, otp } = req.body;
  if (!phone || !otp) return res.status(400).json({ error: 'Phone and OTP required' });

  const otpKey = `otp:${phone}`;
  const storedOTP = await redis.get(otpKey);

  // OTP expired or invalid
  if (!storedOTP) {
    return res.status(400).json({ error: 'OTP has expired or invalid request.' });
  }

  // OTP Match Check
  if (storedOTP === otp) {
    // Delete OTP key immediately after successful verification (One-time use)
    await redis.del(otpKey);
    return res.json({ success: true, message: 'Phone number verified successfully!' });
  }

  return res.status(400).json({ success: false, error: 'Incorrect OTP. Try again.' });
});

app.listen(3001, () => console.log('OTP Service running on http://localhost:3001'));
```

---

## 📌 Production Best Practices
1. **Cache Invalidation**: Jab Banner data update ho, tab `redis.del('site_banners')` call karo.
2. **One-Time OTP Use**: OTP verify hote hi `redis.del(otpKey)` call karke duplicate use attack ko roko!

