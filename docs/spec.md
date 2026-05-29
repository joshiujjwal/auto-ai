# AutoAI — Feature Specification

## Overview

AutoAI is a mobile-first AI mechanic assistant. Users describe symptoms, snap photos of damage, scan OBD codes, and track maintenance — all answered by GPT-4o with vehicle-aware context.

**Problem**: Most people don't know if a car symptom is a $20 fix or a $2,000 repair. Mechanic consultations are expensive and inaccessible. OBD scanners give codes but no plain-English explanation.

**Solution**: A pocket mechanic powered by GPT-4o that understands your specific vehicle and gives honest, actionable advice.

---

## Functional Requirements

### F1 — Conversational AI Mechanic

- [ ] User can send text messages describing car symptoms
- [ ] System injects vehicle profile (year/make/model/mileage/trim) into every GPT-4o system prompt
- [ ] Conversation history is maintained per session (up to 20 turns)
- [ ] AI responds with: likely cause, severity (low/medium/high/urgent), recommended action, estimated cost range
- [ ] User can start a new conversation at any time (clears history)
- [ ] If user hasn't set a vehicle, AI asks for one before diagnosing

### F2 — OBD Code Diagnosis

- [ ] User can enter any OBD-II code (P, B, C, U formats)
- [ ] Local lookup handles common P0xxx codes instantly (no API call)
- [ ] Unknown codes fall back to GPT-4o
- [ ] Response includes: code title, affected systems, severity level, likely causes, DIY fix steps, when-to-see-mechanic threshold
- [ ] Severity displayed as color-coded badge: 🟢 info / 🟡 caution / 🔴 urgent
- [ ] "Ask about this code" button pre-fills chat with OBD context
- [ ] Input validated — rejects malformed codes (must match `/^[PBCU][0-9]{4}$/i`)

### F3 — Photo Analysis

- [ ] User can capture photo with camera or select from gallery
- [ ] Image sent to GPT-4o Vision API via backend (never directly from mobile)
- [ ] Response includes: identified parts/damage, severity assessment, recommended actions, confidence level
- [ ] Backend validates: file type (jpg/png/webp), max size 10MB
- [ ] Mobile compresses image to < 1MB before upload
- [ ] "Add to chat" button injects photo analysis summary into active conversation

### F4 — Maintenance Reminders

- [ ] User can add service records (oil change, tires, brakes, coolant, timing belt, etc.)
- [ ] Each record tracks: last service date, last service mileage, interval (miles and/or months)
- [ ] App calculates next due date and next due mileage
- [ ] `GET /maintenance/due` returns items that are overdue or due within 30 days / 500 miles
- [ ] Push notifications sent for due/overdue services
- [ ] Mileage update prompt shown on app open (user can skip)
- [ ] "Suggest schedule" — GPT-4o generates a starter maintenance plan from vehicle profile

### F5 — Vehicle Profile

- [ ] User can add multiple vehicles (year, make, model, trim, mileage, VIN optional)
- [ ] One vehicle is "active" at a time — all AI features use active vehicle context
- [ ] Vehicle stored in backend DB and locally in AsyncStorage for offline access

---

## Non-Functional Requirements

- [ ] API response time < 2s for local OBD lookup, < 8s for GPT-4o calls
- [ ] App loads to home screen in < 2s on mid-range Android
- [ ] Backend handles 100 concurrent users without degradation
- [ ] Images never stored permanently on backend — delete after GPT-4o response
- [ ] JWT tokens expire after 7 days; refresh token flow for seamless re-auth
- [ ] All API routes require auth except `/auth/register`, `/auth/login`, `/health`
- [ ] OpenAI API key only on backend — never in mobile bundle
- [ ] Rate limit: 30 AI requests per user per hour (configurable)

---

## Data Models

### User
```
id            UUID PRIMARY KEY
email         TEXT UNIQUE NOT NULL
password_hash TEXT NOT NULL
created_at    TIMESTAMP
updated_at    TIMESTAMP
```

### Vehicle
```
id            UUID PRIMARY KEY
user_id       UUID FK → users.id
year          INTEGER NOT NULL
make          TEXT NOT NULL
model         TEXT NOT NULL
trim          TEXT
mileage       INTEGER
vin           TEXT
is_active     BOOLEAN DEFAULT false
created_at    TIMESTAMP
updated_at    TIMESTAMP
```

### ConversationSession
```
id            UUID PRIMARY KEY
user_id       UUID FK → users.id
vehicle_id    UUID FK → vehicles.id
messages      JSONB   -- [{role, content, timestamp}]
created_at    TIMESTAMP
updated_at    TIMESTAMP
```

### MaintenanceRecord
```
id                 UUID PRIMARY KEY
vehicle_id         UUID FK → vehicles.id
service_type       TEXT NOT NULL   -- "Oil Change", "Tire Rotation", etc.
last_done_date     DATE
last_done_mileage  INTEGER
interval_miles     INTEGER
interval_months    INTEGER
notes              TEXT
created_at         TIMESTAMP
updated_at         TIMESTAMP
```

### OBDLookup (static, seed data)
```
code          TEXT PRIMARY KEY   -- e.g. "P0300"
title         TEXT NOT NULL
systems       TEXT[]
severity      TEXT               -- "info" | "caution" | "urgent"
description   TEXT
common_causes TEXT[]
```

---

## API Design

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/auth/register` | ❌ | Create account |
| POST | `/auth/login` | ❌ | Login, receive JWT |
| GET | `/vehicles` | ✅ | List user's vehicles |
| POST | `/vehicles` | ✅ | Add vehicle |
| PUT | `/vehicles/:id` | ✅ | Update vehicle (mileage, etc.) |
| DELETE | `/vehicles/:id` | ✅ | Remove vehicle |
| POST | `/chat` | ✅ | Send message to AI mechanic |
| GET | `/chat/history` | ✅ | Get conversation history |
| DELETE | `/chat/history` | ✅ | Clear conversation |
| POST | `/obd/diagnose` | ✅ | Diagnose OBD code |
| POST | `/photo/analyze` | ✅ | Analyze car photo |
| GET | `/maintenance` | ✅ | List maintenance records |
| POST | `/maintenance` | ✅ | Add service record |
| PUT | `/maintenance/:id` | ✅ | Update service record |
| DELETE | `/maintenance/:id` | ✅ | Delete record |
| GET | `/maintenance/due` | ✅ | Get overdue/upcoming services |
| POST | `/maintenance/suggest` | ✅ | AI-generated maintenance plan |
| GET | `/health` | ❌ | Health check |

---

## System Prompt Design

Every AI mechanic call includes this system context:

```
You are AutoAI, an expert automotive mechanic assistant.
The user's vehicle: {year} {make} {model} {trim}, current mileage: {mileage} miles.

Rules:
- Give specific, actionable advice for THIS vehicle
- Always state severity: low / medium / high / urgent
- Include rough cost range when relevant ($X–$Y)
- Recommend professional inspection for anything safety-critical
- Never recommend illegal modifications
- Be concise — mechanics don't write essays
```

---

## Test Plan

### Unit Tests
- OBD code validator: valid P/B/C/U codes, malformed inputs, edge cases
- Maintenance due-soon logic: overdue by date, overdue by mileage, both, neither
- Prompt builder: correct vehicle context injected, history truncated at 20 turns
- JWT middleware: valid token, expired token, missing token, tampered token

### Integration Tests
- Full auth flow: register → login → protected route
- Chat flow: POST /chat with vehicle context → valid OpenAI response shape
- OBD flow: known code → local response; unknown code → GPT response
- Photo flow: valid image → analysis response; oversized → 400; wrong type → 400
- Maintenance CRUD + due-soon calculation

### Mobile Tests (RNTL)
- ChatScreen: renders messages, sends input, shows loading state
- OBDScreen: renders input, shows result card with correct severity badge
- MaintenanceScreen: renders list, overdue items highlighted

### Edge Cases
- Chat with no active vehicle
- OBD code in lowercase (`p0300`) — normalize before lookup
- Photo with zero-byte content
- Maintenance with only miles interval (no date)
- OpenAI API timeout — return 503 with retry-after header

---

## Open Questions

- [ ] Should conversation history persist across sessions (DB) or be ephemeral (in-memory)?
- [ ] Do we want a VIN decoder integration (NHTSA API) to auto-fill vehicle details?
- [ ] Rate limits: per-user hourly or daily? What's the token budget per request?
- [ ] Should photo analysis results be stored for reference history?
- [ ] Monetization: free tier (N AI calls/day) vs. subscription?
