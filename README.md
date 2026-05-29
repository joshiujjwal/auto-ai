# AutoAI 🚗🤖

> AI-powered mobile mechanic — diagnose car problems, decode OBD codes, analyze damage photos, and track maintenance.

![Status](https://img.shields.io/badge/status-🚧%20Early%20Development-orange)
![Stack](https://img.shields.io/badge/stack-React%20Native%20%2B%20Node.js%20%2B%20GPT--4o-blue)

---

## What It Does

AutoAI puts an AI mechanic in your pocket. Snap a photo of damage, describe a symptom, scan an OBD code, or ask when your next oil change is due — and get expert-level answers powered by GPT-4o.

| Feature | Description |
|---|---|
| 💬 AI Mechanic Chat | Conversational diagnosis from symptoms in plain English |
| 🔌 OBD Code Lookup | Decode P/B/C/U codes with cause, severity, and repair steps |
| 📸 Photo Analysis | GPT-4o Vision analyzes damage, wear, and part identification |
| 🔔 Maintenance Reminders | Track service history and get proactive alerts by mileage/date |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Mobile | React Native (Expo) |
| Backend API | Node.js + Express |
| AI | OpenAI GPT-4o (chat + vision) |
| Database | PostgreSQL (backend) + AsyncStorage (mobile) |
| Auth | JWT |
| Testing | Jest + Supertest (backend), Jest + RNTL (mobile) |
| CI | GitHub Actions |

---

## Getting Started

### Prerequisites

- Node.js 20+
- Expo CLI (`npm install -g expo-cli`)
- PostgreSQL
- OpenAI API key

### Backend

```bash
cd backend
npm install
cp .env.example .env          # add OPENAI_API_KEY, DATABASE_URL, JWT_SECRET
npm run db:migrate
npm run dev
```

### Mobile

```bash
cd mobile
npm install
cp .env.example .env          # add EXPO_PUBLIC_API_URL
npx expo start
```

### Run All Tests

```bash
# backend
cd backend && npm test

# mobile
cd mobile && npm test
```

---

## Project Structure

```
auto-ai/
├── mobile/                   # React Native (Expo) app
│   ├── src/
│   │   ├── screens/          # Chat, OBD, Camera, Maintenance, Settings
│   │   ├── components/       # Shared UI components
│   │   ├── services/         # API client, OpenAI, notifications
│   │   ├── hooks/            # useVehicle, useChat, useMaintenance
│   │   └── utils/            # OBD code helpers, formatters
│   └── tests/
├── backend/                  # Node.js/Express API
│   ├── src/
│   │   ├── routes/           # /chat, /obd, /photo, /maintenance, /vehicles
│   │   ├── controllers/      # Request handlers
│   │   ├── services/         # OpenAI service, DB queries
│   │   ├── middleware/        # Auth, rate-limiting, error handling
│   │   └── utils/            # OBD code database, prompt builders
│   └── tests/
├── docs/
│   ├── spec.md               # Feature specification
│   └── adr/                  # Architecture Decision Records
├── .github/
│   └── copilot-instructions.md
├── README.md
├── TODO.md
├── CLAUDE.md
└── AGENTS.md
```

---

## Contributing

1. **Write tests first** (red phase) before any implementation
2. Implement until tests pass (green phase)
3. Review your own diff before pushing
4. PRs require evidence: test output, screenshots, or API response samples
5. Keep PRs small and focused — one feature or fix per PR
6. Update `CLAUDE.md` / `AGENTS.md` if you discover new conventions

---

## License

Private — All rights reserved.
