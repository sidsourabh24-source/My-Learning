# 🎬 System Design #7: How Video Streaming Works at Scale
*(Notes from Piyush Garg System Design Series - Video Streaming Pipeline)*

> **Video Reference**: [How Video Streaming Works on Scale - Piyush Garg](https://youtu.be/-JtjQ-OA7XE)  
> **Core Question**: YouTube, Netflix, aur Hotstar hazaro gigabytes ki 4K videos millions of users ko bina buffer kiye instantly kaise stream karte hain?

---

## ❌ 1. Why Simple `.mp4` Delivery Fails Miserably

Agar hum naive tarike se video deliver karein:
```text
[ Browser / App ] ◄──(Download 2GB video.mp4 via HTTP)── [ S3 / Server ]
```

### 🔴 The 3 Disasters of Raw MP4:
1. **Bandwidth Waste**: Agar user ne 1 hour ki video (2 GB) sirf 30 seconds dekh kar band kar di, to baki 1.9 GB download hone ka bandwidth aur cloud bill faltu me waste ho gaya!
2. **Network Buffering**: User train me travel kar raha hai aur network 4G se 2G par drop hua, to video freeze ho jayegi kyunki single high-bitrate file download nahi ho paayegi.
3. **Device Incompatibility**: Mobile screen (360p/720p) ke liye 4K file download karna CPU aur battery dono kill kar deta hai.

---

## 💡 2. The Solution: Chunking & Adaptive Bitrate Streaming (ABR)

Production streaming me hum poori video kabhi ek saath nahi bhejte!

```text
Full 1-Hour Video (10 GB) 
          │
          ▼ (FFmpeg Processing)
1. Split into Tiny Chunks (2 to 6 seconds each) ➔ [ chunk_0.ts ], [ chunk_1.ts ], [ chunk_2.ts ] ...
2. Encode into Multiple Resolutions           ➔ 360p, 480p, 720p, 1080p, 4K
```

### 📶 Adaptive Bitrate Streaming (ABR) Kya Hai?
Video player client-side par user ki current internet speed measure karta rehta hai:
- Network Fast (WiFi) ➔ Player automatically **1080p chunks** request karega.
- Network Slow (2G/3G) ➔ Player smoothly bina atke **360p chunks** par switch ho jayega!
- **User Experience**: Video kabhi pause/buffer nahi hoti, sirf quality auto-adjust hoti hai!

---

## 📜 3. Streaming Protocols: HLS & MPEG-DASH

Streaming industry standard protocols use karti hai jisme ek **Index (Manifest) File** aur **Video Chunks** hote hain:

```text
┌────────────────────────────────────────────────────────┐
│ 🍎 HLS (HTTP Live Streaming - Created by Apple)        │
│ • Manifest File: .m3u8 (Playlist index)                │
│ • Video Chunk Format: .ts (Transport Stream)           │
│ • Universal Support: iOS, Android, Web, Smart TVs      │
├────────────────────────────────────────────────────────┤
│ 🌐 MPEG-DASH (Dynamic Adaptive Streaming over HTTP)    │
│ • Manifest File: .mpd (XML based presentation)         │
│ • Video Chunk Format: .m4s (Fragmented MP4)            │
│ • Open International Standard                          │
└────────────────────────────────────────────────────────┘
```

### 🔍 How `.m3u8` Works Internally:
Jab aap video play karte ho, player pehle ek **Master Playlist** fetch karta hai:

```text
master.m3u8 (Master Index)
├── 360p  ➔ 360p_stream.m3u8  ➔ [chunk_0.ts, chunk_1.ts, ...]
├── 720p  ➔ 720p_stream.m3u8  ➔ [chunk_0.ts, chunk_1.ts, ...]
└── 1080p ➔ 1080p_stream.m3u8 ➔ [chunk_0.ts, chunk_1.ts, ...]
```
Player video ke aage ke sirf **2-3 chunks buffer** karke rakhta hai. Agar user tab close kar de, to koi bandwidth waste nahi hoti!

---

## 🏗️ 4. End-to-End Video Streaming Architecture (YouTube / Netflix Pipeline)

Piyush Garg video me complete production pipeline explain karte hain:

```text
[ Creator ] ──► (1. Direct Upload via Presigned URL) ──► [ Raw S3 Bucket ]
                                                                │
                                            (2. Upload Event)   ▼
                                                       [ AWS SQS / Kafka ]
                                                                │
                                            (3. Pull Job)       ▼
                                                    [ Transcoding Workers ]
                                                    (EC2 Spot Instances + FFmpeg)
                                                                │
                                                                ├── Chunks into 360p, 720p, 1080p
                                                                └── Generates .m3u8 Playlists
                                                                │
                                            (4. Save Processed) ▼
                                                     [ Production S3 Bucket ]
                                                                │
                                            (5. Global Cache)   ▼
                                                     [ CloudFront CDN Edge ]
                                                                │
                                            (6. Stream Fast)    ▼
                                                      [ End User App ] 📱
```

### 🛠️ Pipeline Step-by-Step Breakdown:

1. **Direct Upload to S3 (Presigned URL)**:
   - Creator video ko direct server par upload nahi karta (warna server memory crash ho jayegi).
   - Backend API se **S3 Presigned URL** milta hai aur video client se direct S3 me stream hoti hai.
2. **Event Notification (SQS / Kafka)**:
   - S3 me video aate hi `s3:ObjectCreated` event SQS queue me push hota hai taaki heavy processing asynchronous rahe.
3. **Transcoding Cluster (FFmpeg Workers)**:
   - Background worker servers (GPU-enabled EC2 / Spot instances) queue se job uthate hain.
   - **FFmpeg tool** video ko alag-alag bitrates me convert karta hai aur 4-second `.ts` chunks create karta hai.
4. **Metadata DB (PostgreSQL / MongoDB)**:
   - Video title, description, creator ID, aur final `master.m3u8` CDN URL database me save hoti hai.
5. **CDN Caching (CloudFront / Akamai)**:
   - Video chunks **Immutable** hote hain (kabhi edit nahi hote).
   - Isiliye CDN par **95%+ Cache Hit Ratio** milta hai! User ke nearest local city edge server se chunk deliver hota hai in <20ms!

---

## 📊 Summary Comparison: Raw MP4 vs HLS Streaming

| Feature | Raw `.mp4` File | HLS / DASH Streaming |
| :--- | :--- | :--- |
| **Delivery Model** | Full Progressive Download | On-demand 2-6 sec Chunks |
| **Quality Adjustment** | ❌ Fixed (360p or 1080p) | 🟢 **Adaptive Bitrate (Dynamic)** |
| **Bandwidth Efficiency** | 🔴 Horrible (Full file wasted) | 🟢 **Optimal (Fetches as you watch)** |
| **Buffering on Poor Network**| 🔴 Frequent video pause | 🟢 Smooth quality degradation |
| **CDN Cache Friendliness** | 🟡 Mediocre (Large files) | 🟢 **Super High (Small cached chunks)** |

---

## 💡 Key Architectural Takeaways

1. **Never Transcode on Main API Server**: Video encoding bohot CPU/GPU heavy task hota hai. Iske liye hamesha dedicated worker pool aur message queue use karo.
2. **Cost Optimization with Spot Instances**: AWS Spot instances 70% cheaper hote hain, jo video transcoding jaise batch jobs ke liye perfect hain.
3. **Edge Caching is King**: YouTube aur Netflix ka 95% traffic origin servers tak pahuche bina CDN edge locations se hi serve ho jata hai.

