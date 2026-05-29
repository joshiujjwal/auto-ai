# CLAUDE.md — AutoAI

Context file for AI agents. Keep updated as conventions evolve.

---

## Commands

### Backend (`cd backend`)
```bash
npm run dev          # nodemon src/server.js
npm test             # jest --runInBand
npm run test:watch   # jest --watch
npm run lint         # eslint src/
npm run db:migrate   # node src/db/migrate.js
npm run db:seed      # node src/db/seed.js (OBD lookup table)
```

### Mobile (`cd mobile`)
```bash
npx expo start       # dev server
npx expo start --ios
npx expo start --android
npm test             # jest
npm run test:watch   # jest --watch
npx eas build --platform all --profile preview   # TestFlight/Play build
```

### From repo root
```bash
# No root-level build script yet — run backend and mobile separately
```

---

## Directory Map

```
auto-ai/
├── mobile/src/
│   ├── screens/        # One file per screen: ChatScreen, OBDScreen, CameraScreen, MaintenanceScreen, SettingsScreen, VehicleScreen
│   ├── components/     # Reusable UI: MessageBubble, SeverityBadge, FindingCard, ServiceRow, LoadingDots
│   ├── services/       # api.js (axios instance), notifications.js
│   ├── hooks/          # useChat, useOBD, useVehicle, useMaintenance, useCamera
│   └── utils/          # formatDate, formatMileage, obdCodeValidator
├── backend/src/
│   ├── routes/         # chatRoutes, obdRoutes, photoRoutes, maintenanceRoutes, vehicleRoutes, authRoutes
│   ├── controllers/    # One per route file — thin, delegate to services
│   ├── services/       # openai.js (GPT-4o wrapper), db.js (pg pool)
│   ├── middleware/     # auth.js (JWT verify), rateLimit.js, errorHandler.js, upload.js (multer)
│   └── utils/          # promptBuilders.js, obdDatabase.js, validators.js
├── docs/spec.md        # Source of truth for features — read before implementing
└── TODO.md             # Current task list — read before writing any code
```

---

## Workflow for AI Agents

1. **Read TODO.md first** — find the current phase and next unchecked task
2. **Run existing tests**: `cd backend && npm test` + `cd mobile && npm test`
3. **Write failing test first** (red phase) — commit with `test: describe what's tested`
4. **Implement to pass** (green phase) — commit with `feat:` or `fix:`
5. **Review diff** before pushing — no accidental file deletions or scope creep
6. **Update this file** if you discover a new convention or gotcha

---

## Non-Obvious Conventions

### OpenAI Service
- All OpenAI calls go through `backend/src/services/openai.js` — never call the SDK directly from controllers
- Conversation history is stored as `[{role, content}]` — always prepend the system prompt fresh, never store it in history
- Truncate history to last 20 turns before sending (not 20 messages — 20 turn *pairs*)
- Log token usage per request to stdout in dev

### Image Handling
- Images are deleted from the server immediately after GPT-4o responds — never persist them
- `multer` stores uploads in `/tmp` (memory storage in prod — don't write to disk on Railway/Render)
- Mobile must compress to < 1MB before upload — use `expo-image-manipulator`

### OBD Codes
- Always normalize to uppercase before lookup: `code.toUpperCase().trim()`
- Format validation regex: `/^[PBCU][0-9]{4}$/`
- Local DB covers P0xxx (generic powertrain) — anything else falls to GPT-4o

### Auth
- JWT payload: `{ userId, email, iat, exp }`
- All protected routes use `middleware/auth.js` — never inline token verification
- Passwords hashed with bcrypt, cost factor 12

### Error Responses
- Always return `{ error: string, code: string }` — never raw Error objects
- HTTP 400: validation errors, 401: auth, 403: forbidden, 422: business logic, 500: unexpected

### React Native
- Never import directly from `react-native` when Expo has an equivalent — use Expo SDK
- AsyncStorage key prefix: `@autoai/` (e.g. `@autoai/active_vehicle`)
- Navigation: React Navigation v6 stack navigator — no tabs yet (future feature)

---

## Environment Variables

### Backend `.env`
```
OPENAI_API_KEY=sk-...
DATABASE_URL=postgresql://user:pass@host:5432/autoai
JWT_SECRET=a-long-random-string
PORT=3000
NODE_ENV=development
MAX_AI_REQUESTS_PER_HOUR=30
```

### Mobile `.env`
```
EXPO_PUBLIC_API_URL=http://localhost:3000
```

---

## Known Gotchas

- GPT-4o vision requires images as base64 data URLs or public HTTPS URLs — use base64 from multer memory storage
- Expo Camera on Android requires explicit permission request before first use
- `pg` pool will error on cold start if `DATABASE_URL` is not set — check early in `server.js`
- React Navigation requires `NavigationContainer` to wrap everything — easy to miss in tests (mock it)
- OBD codes from physical scanners sometimes include spaces or dashes — strip non-alphanumeric before validating
