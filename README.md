# 🚀 Student Commute Optimizer

### A privacy-aware, low-cost student carpool app that matches students with overlapping routes and provides anonymous in-app chat to coordinate rides.

---

## 📚 Table of Contents
- [🛠️ Tech Stack & Tools](#️-tech-stack--tools)
- [✨ Core Features](#-core-features)
- [🏗️ High-Level Architecture](#️-high-level-architecture)
- [🗺️ User Workflow](#️-user-workflow)
- [🧠 Matching Algorithm](#-matching-algorithm)
- [💻 Code Snippets](#-code-snippets)
- [🔮 Future Scope ](#-future-scope)
- [🚀 Getting Started](#-getting-started)

---

## 🛠️ Tech Stack & Tools


This project leverages a modern, scalable, and efficient technology stack.

| Frontend        | Backend     | Database & Caching | Services          |
|-----------------|-------------|--------------------|-------------------|
| **Next.js**     | **Node.js** | **MongoDB**        | **Google Maps**   |
| **Tailwind CSS**| **TypeScript** | **Redis**       | **Shadcn/UI**     |
|                 |             |                    | **AI (Gemini/GPT)** |

---

## ✨ Core Features

- **📍 Map-Based Interface**: Students enter origin and destination using Google Maps.  
- **🤫 Privacy-First**: Anonymous usernames keep identities safe.  
- **🤝 Smart Matching**: Finds peers with overlapping routes and times.  
- **💬 Secure Chat**: Coordinate directly in-app without sharing phone numbers.  
- **⚡ Scalable Backend**: Designed for MVP → enterprise scale.  

---


## 🏗️ High-Level Architecture

```plaintext
[Client: Next.js + Tailwind + Google Maps]
   |
   | REST / WebSocket (for Realtime)
   v
[API Layer: Next.js API Routes (TypeScript)]
   |
   |--> Auth Service: Supabase
   |--> Primary DB: MongoDB Atlas
   |--> Realtime DB: Supabase
   |--> Mapping Service: Google Maps
   |
   +---->[Matching Worker (Node.js / Serverless)]
               |
               +---->[Job Queue: Redis]
```


---

## 🗺️ User Workflow

1. **Sign Up** → Supabase Auth (email/SSO)  
2. **Create Trip** → Select origin & destination, store route  
3. **Find Matches** → Background worker suggests top 5 matches  
4. **Chat & Coordinate** → Secure in-app chat  
5. **Ride Lifecycle** → Confirm → Ride → Auto-expire trip  

---

## 🧠 Matching Algorithm

<details>
<summary><strong>Expand Detailed Algorithm</strong></summary>

- Sample polyline every 150m  
- Query trips within 200m buffer using MongoDB `$near`  
- Filter by departure time ±15min  
- Score by overlap, start/end proximity, time difference  
- Return **top 5 matches**  

</details>

---

## 💻 Code Snippets

### Generate Anonymous Username
```ts
import crypto from 'crypto';

const ADJECTIVES = ['Violet', 'Swift', 'Quiet', 'Brave', 'Silent'];
const NOUNS = ['Fox', 'Otter', 'Raven', 'Bear', 'Wolf'];

async function generateAnonUsername(db: Db): Promise<string> {
  while (true) {
    const adj = ADJECTIVES[Math.floor(Math.random() * ADJECTIVES.length)];
    const noun = NOUNS[Math.floor(Math.random() * NOUNS.length)];
    const hex = crypto.randomBytes(2).toString('hex');
    const username = `${adj}${noun}${hex}`;
    if (!(await db.collection('users').findOne({ anonUsername: username }))) {
      return username;
    }
  }
}
```

### Create a Trip API
```ts
import { getDirections, samplePolyline } from '@/lib/maps';
import { db } from '@/lib/db';

export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).json({ error: 'Method not allowed' });

  const { userId, origin, dest, departTime } = req.body;
  const directions = await getDirections(origin, dest);
  const sampledPoints = samplePolyline(directions.polyline, 150);

  const trip = await db.collection('trips').insertOne({
    userId,
    origin,
    dest,
    polyline: directions.polyline,
    sampledPoints,
    departTime: new Date(departTime),
    status: 'active',
    createdAt: new Date(),
  });

  await enqueueMatchJob(trip.insertedId);

  res.status(201).json({ success: true, tripId: trip.insertedId });
}
```

---

## 🔮 Future Scope

- **LLM Integration** → Pickup point suggestions, chat ice-breakers.  
- **Safety Features** → SOS button, mutual verification, rating system.  
- **Scaling** → Workers, sharding, alternative routing APIs.  

---

## 🚀 Getting Started

### Prerequisites
- Node.js (v18+)  
- npm / yarn / pnpm  
- MongoDB Atlas  
- Supabase  
- Google Maps API Key  

### Installation
```bash
git clone https://github.com/your_username/student-commute-optimizer.git
cd student-commute-optimizer
npm install
```

### Environment Variables
```env
MONGODB_URI=your_mongodb_connection_string
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=your_google_maps_api_key
```

### Run Locally
```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

---
## 🏗️ Architecture Flow (Text)

```plaintext
                 ┌────────────────────┐
                 │     Frontend       │
                 │  Next.js + Tailwind│
                 │   Google Maps UI   │
                 └─────────┬──────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │      API Layer       │
                │ Next.js API Routes   │
                │  (TypeScript)        │
                └─────────┬────────────┘
                          │
        ┌─────────────────┼────────────────────┐
        ▼                 ▼                    ▼
┌─────────────┐   ┌────────────────┐   ┌────────────────┐
│ Auth Service│   │ Primary DB     │   │ Realtime DB    │
│ Supabase    │   │ MongoDB Atlas  │   │ Supabase       │
│ (SSO/Email) │   │ (Trip + Routes)│   │ (Chat/Notif.)  │
└─────────────┘   └────────────────┘   └────────────────┘
                          │
                          ▼
                ┌──────────────────────┐
                │ Mapping Service      │
                │ Google Maps API      │
                │ (Directions/Geocode) │
                └─────────┬────────────┘
                          │
                          ▼
                ┌──────────────────────┐
                │ Matching Worker      │
                │ Node.js / Serverless │
                └─────────┬────────────┘
                          │
                          ▼
                ┌──────────────────────┐
                │   Job Queue (Redis)  │
                │ Schedules/Processes  │
                │    Match Jobs        │
                └──────────────────────┘
