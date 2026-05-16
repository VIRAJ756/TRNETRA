# TRINETRA 🛡️
### Your Campus Safety Companion
**Smart Technology. Safer Tomorrow.**

> Every location. Every moment. Protected every movement and your moves

---

## 🚨 What is TRINETRA?

TRINETRA is a dark, intelligence-first campus safety platform built for real-world threat response. It goes beyond a simple "emergency button" — giving users operator-grade situational awareness with AI-powered risk analysis, automatic evidence capture, and instant guardian escalation.

Built for **IMPACTHON 2026**.

---

## ✨ Features

| Feature | Description |
|---|---|
| 🆘 **Hold-to-Activate SOS** | 3-second hold mechanic with SVG progress ring — prevents accidental triggers |
| 📸 **Smart Evidence Capture** | Auto-captures photo (front camera, 1.5s warmup) + 10s ambient audio on SOS |
| 🔗 **Evidence Link in WhatsApp** | Files uploaded to server instantly — guardian receives a real link with photo + audio |
| 🗺️ **Live Risk Heatmap** | Real-time incident-based heatmaps on Leaflet + OpenStreetMap |
| 🧠 **AI Risk Analysis** | Gemini-powered predictive analytics with heuristic fallback |
| 🛤️ **Safe Route Navigation** | AI-computed safest path between two points |
| 👥 **Guardian Tracking** | Trusted contacts track your live location during a journey |
| ⏱️ **Journey Overdue Detection** | Alerts guardians if you don't check in on time |
| 📢 **Community Incident Feed** | Crowdsourced real-time incident reporting |
| 📵 **Offline Emergency Mode** | Service Worker queues SOS when offline, flushes when reconnected |
| 🎤 **Voice SOS Trigger** | Say "Help" or "SOS" — hands-free activation |
| 📞 **Fake Call Escape** | Simulate an incoming call to exit unsafe situations |
| 🔔 **Real-time Alerts** | Socket.io-powered instant alerts to guardians and nearby users |

---

## 🏗️ Architecture

```
┌──────────────────────────┐     WebSocket     ┌──────────────────────────┐
│  React + Vite (web)       │◀────────────────▶│ Node + Express (api)      │
│  Leaflet + SOS + Journey  │                  │ REST + Socket.io          │
│  React Query + Zustand    │──────REST───────▶│ Zod + Multer              │
└─────────────┬────────────┘                  └─────────────┬────────────┘
              │                                              │
              │                                              │ Prisma ORM
              │                                              ▼
              │                                    ┌──────────────────┐
              │                                    │ SQLite (dev.db)   │
              │                                    └──────────────────┘
              │
              ├──────────────▶ Browser Geolocation API
              ├──────────────▶ MediaRecorder + Canvas (evidence capture)
              ├──────────────▶ Service Worker (offline SOS queue)
              └──────────────▶ Google Gemini AI + heuristic fallback
```

---

## 🗄️ Database Schema

```prisma
model Incident      { id, lat, lng, type, description, createdAt }
model Journey       { id, token, startLat, startLng, destLat, destLng, status, createdAt }
model Contact       { id, name, phone, relation, createdAt }
model EvidenceRecord {
  id        String   @id @default(cuid())
  token     String   @unique          // UUID — used in public evidence URL
  photoFile String?                   // Filename in /uploads
  audioFile String?                   // Filename in /uploads
  lat       Float?
  lng       Float?
  timestamp String?
  createdAt DateTime @default(now())
}
```

---

## 🔌 API Reference

### Core
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/health` | Health check |

### Incidents
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/incidents` | Fetch all incidents |
| POST | `/api/incidents` | Report a new incident |
| GET | `/api/zones/heatmap` | Risk zone heatmap data |

### SOS & Evidence
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/sos` | Trigger SOS alert |
| POST | `/api/evidence` | Upload photo + audio blobs → returns evidence page URL |
| GET | `/api/evidence/:token` | Fetch evidence record by UUID token |

### Journey
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/journey/start` | Start a new journey |
| POST | `/api/journey/complete` | Mark journey complete |
| GET | `/api/journey/active` | Get active journey |
| GET | `/api/journey/track/:token` | Public guardian tracking page |

### Contacts
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/contacts` | List all contacts |
| POST | `/api/contacts` | Add a contact |
| DELETE | `/api/contacts/:id` | Remove a contact |
| POST | `/api/contacts/:id/test-alert` | Send test alert to contact |

### AI
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/ai/risk-analysis` | Get AI risk summary for a location |
| POST | `/api/ai/safe-route` | Compute safest route between two points |

---

## 🚀 Local Setup

**Prerequisites:** Node.js 18+, npm

### 1. Clone

```bash
git clone https://github.com/VIRAJ756/TRNETRA.git
cd TRNETRA
```

### 2. Environment

```bash
cp apps/api/.env.example apps/api/.env
```

Edit `apps/api/.env`:

```env
DATABASE_URL="file:./dev.db"
PORT=4000
GEMINI_API_KEY=your_key_here        # optional — heuristic fallback works without it
```

### 3. Install

```bash
cd apps/api && npm install
cd ../web && npm install
```

### 4. Database

```bash
cd apps/api
npx prisma generate
npx prisma db push
```

### 5. Run

Open two terminals:

```bash
# Terminal 1 — API
cd apps/api && npm run dev

# Terminal 2 — Web
cd apps/web && npm run dev
```

Open **http://localhost:3000**

---

## 🎭 Demo Mode

Add `?demo=true` to the URL to seed fake incidents, simulate a journey, trigger a zone alert, and show a pre-filled AI panel.

```
http://localhost:3000/dashboard?demo=true
```

---

## 🔬 How Evidence Capture Works

```
User holds SOS button (3 seconds)
        ↓
SOS screen appears immediately (non-blocking)
        ↓
Camera opens → 1.5s warmup → canvas snapshot (JPEG base64)
Microphone opens → 10s MediaRecorder → audio/webm blob
        ↓
Both files converted to File objects → POST /api/evidence (multipart)
        ↓
API saves to apps/api/uploads/ → inserts EvidenceRecord → returns UUID token
        ↓
WhatsApp message opens pre-filled with:
  📍 Google Maps link
  📸 Evidence page: https://your-domain/evidence/{token}
        ↓
Guardian opens link → sees real photo + plays real audio
```

---

## 🛠️ Tech Stack

### Frontend
- React 18 + TypeScript + Vite
- Tailwind CSS v3 + shadcn/ui
- Leaflet + OpenStreetMap
- Zustand + React Query v5
- Socket.io client
- Service Worker (offline mode)

### Backend
- Node.js + Express + TypeScript
- Prisma ORM + SQLite
- Socket.io (real-time)
- Multer (file uploads)
- Zod (validation)

### AI & Notifications
- Google Gemini API (risk analysis + safe route)
- Heuristic fallback (works with no API key)
- WhatsApp `wa.me` deep links
- Simulated SMS (demo mode)

---

## 📁 Project Structure

```
skyshield/
├── apps/
│   ├── api/                    # Express backend
│   │   ├── prisma/
│   │   │   ├── schema.prisma
│   │   │   └── migrations/
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   │   ├── evidence.ts   # Evidence upload + retrieval
│   │   │   │   ├── sos.ts
│   │   │   │   ├── journey.ts
│   │   │   │   ├── contacts.ts
│   │   │   │   ├── incidents.ts
│   │   │   │   ├── zones.ts
│   │   │   │   └── ai.ts
│   │   │   ├── services/
│   │   │   │   ├── geminiService.ts
│   │   │   │   ├── riskEngine.ts
│   │   │   │   └── notifier.ts
│   │   │   └── index.ts
│   │   └── uploads/              # Evidence files (gitignored)
│   │
│   └── web/                    # React frontend
│       ├── public/
│       │   └── sw.js             # Service worker (offline SOS)
│       └── src/
│           ├── components/
│           │   ├── sos/
│           │   │   ├── SOSButton.tsx       # Hold-to-activate + evidence capture
│           │   │   └── SOSActivePanel.tsx  # Emergency active UI
│           │   ├── map/                    # Leaflet heatmap + layers
│           │   └── layout/
│           │       └── NetworkBanner.tsx   # Online/offline indicator
│           ├── hooks/
│           │   └── useSosCapture.ts        # Camera + audio + upload logic
│           └── pages/
│               ├── Dashboard.tsx
│               ├── EvidenceViewer.tsx      # Public /evidence/:token page
│               ├── JourneyShare.tsx
│               └── TrackView.tsx
```

---

## 👥 Team

| Name | Role |
|---|---|
| Viraj | Full Stack + AI Integration |
| (teammate) | (role) |
| (teammate) | (role) |

---

## 📄 License

MIT — built for IMPACTHON 2026.

----

<div align="center">
  <strong>TRINETRA</strong> · Your Campus Safety Companion<br/>
  <em>Smart Technology. Safer Tomorrow.</em>
</div>


