# AutoAI — Task Breakdown

## How to Use This File

Workflow per task:
1. Write failing tests FIRST (red phase)
2. Implement until tests pass (green phase)
3. Review your diff manually before committing
4. Commit with a descriptive message referencing the task
5. Update `CLAUDE.md` or `AGENTS.md` if you discover new conventions (compound loop)
6. Check the phase evidence gate before moving to the next phase

**Evidence gate**: Each phase must have ✅ passing tests + 👀 human diff review before proceeding.

---

## Phase 0: Foundation ⬜

### Backend
- [ ] `cd backend && npm init` — set up package.json with `type: "module"`
- [ ] Install deps: `express`, `cors`, `helmet`, `dotenv`, `pg`, `openai`, `multer`, `jsonwebtoken`, `bcrypt`, `zod`
- [ ] Install devDeps: `jest`, `supertest`, `nodemon`, `eslint`, `prettier`
- [ ] Create `src/index.js` — Express app factory (no side effects at import)
- [ ] Create `src/server.js` — entry point that calls `app.listen()`
- [ ] Write smoke test: `GET /health` returns `{ status: "ok" }` (red → green)
- [ ] Set up `.env.example` with `OPENAI_API_KEY`, `DATABASE_URL`, `JWT_SECRET`, `PORT`
- [ ] GitHub Actions CI: lint + test on push to `main` and PRs

### Mobile
- [ ] `cd mobile && npx create-expo-app . --template blank-typescript`
- [ ] Install deps: `axios`, `@react-navigation/native`, `@react-navigation/stack`, `expo-camera`, `expo-image-picker`, `expo-notifications`, `@react-native-async-storage/async-storage`
- [ ] Install devDeps: `jest`, `@testing-library/react-native`, `@testing-library/jest-native`
- [ ] Write smoke test: App renders without crashing (red → green)
- [ ] Set up `.env.example` with `EXPO_PUBLIC_API_URL`

### Review gate ⛔
All tests pass. CI is green. Diff reviewed. → Proceed to Phase 1.

---

## Phase 1: Auth + Vehicle Profile ⬜

### Backend
- [ ] Write tests for `POST /auth/register` and `POST /auth/login` (red)
- [ ] Implement User model (id, email, password_hash, created_at)
- [ ] Implement Vehicle model (id, user_id, year, make, model, trim, mileage, vin)
- [ ] Implement auth routes with JWT (green)
- [ ] Write tests for `GET/POST/PUT /vehicles` — CRUD for user's cars (red)
- [ ] Implement vehicle routes with auth middleware (green)
- [ ] Rate-limit auth endpoints (5 req/min per IP)

### Mobile
- [ ] Write tests for Login and Register screens render correctly (red)
- [ ] Implement Login / Register screens with form validation (green)
- [ ] Write tests for `useVehicle` hook — add/select active vehicle (red)
- [ ] Implement `useVehicle` hook with AsyncStorage persistence (green)
- [ ] Implement vehicle onboarding flow (year → make → model → mileage)

### Review gate ⛔
Auth flow works end-to-end. Vehicle CRUD tested. → Proceed to Phase 2.

---

## Phase 2: AI Mechanic Chat ⬜

### Backend
- [ ] Write tests for `POST /chat` — validates request body, returns structured response (red)
- [ ] Build `services/openai.js` — wraps OpenAI client, handles retries and errors
- [ ] Build system prompt in `utils/promptBuilders.js` — injects vehicle context (year/make/model/mileage)
- [ ] Implement `/chat` controller: maintains conversation history per session (green)
- [ ] Write tests for conversation context — second message references first (red)
- [ ] Implement session-based conversation memory (Redis or in-memory for now) (green)
- [ ] Cost guardrail: truncate conversation history at 20 turns to control token spend

### Mobile
- [ ] Write tests for ChatScreen renders message list and input (red)
- [ ] Implement `ChatScreen` with FlatList message bubbles (user/assistant) (green)
- [ ] Write tests for `useChat` hook — sends message, appends to history (red)
- [ ] Implement `useChat` hook calling `/chat` API (green)
- [ ] Add loading indicator during AI response
- [ ] Add "new conversation" button (clears history)

### Review gate ⛔
End-to-end chat tested with a real vehicle context. Token usage logged. → Proceed to Phase 3.

---

## Phase 3: OBD Code Diagnosis ⬜

### Backend
- [ ] Write tests for `POST /obd/diagnose` — accepts code string, returns structured diagnosis (red)
- [ ] Build `utils/obdDatabase.js` — local lookup table for 200+ common OBD-II P-codes (P0100–P0999)
- [ ] Implement OBD controller: local lookup first, GPT-4o fallback for unknown codes
- [ ] Response schema: `{ code, title, systems, severity, likely_causes[], diy_fixes[], when_to_see_mechanic }` (green)
- [ ] Write tests for unknown code fallback path (red → green)
- [ ] Write tests for malformed code input (e.g. "P01" — return 400) (red → green)

### Mobile
- [ ] Write tests for OBDScreen renders code input and result card (red)
- [ ] Implement `OBDScreen` — text input for code entry, result display card (green)
- [ ] Write tests for severity color coding (red/yellow/green badge) (red → green)
- [ ] Implement severity badge component
- [ ] Add "Ask about this code" button → pre-fills chat with OBD context

### Review gate ⛔
≥5 OBD codes tested. Severity levels verified. Chat integration working. → Proceed to Phase 4.

---

## Phase 4: Photo Analysis ⬜

### Backend
- [ ] Write tests for `POST /photo/analyze` — accepts multipart image, returns analysis (red)
- [ ] Set up `multer` middleware for image uploads (max 10MB, jpg/png/webp only)
- [ ] Implement photo controller: encode image to base64, send to GPT-4o vision
- [ ] Build photo prompt in `utils/promptBuilders.js` — instructs GPT-4o to identify damage, parts, severity
- [ ] Response schema: `{ findings[], severity, recommended_actions[], confidence }` (green)
- [ ] Write tests for oversized file rejection (red → green)
- [ ] Write tests for non-image file rejection (red → green)

### Mobile
- [ ] Write tests for CameraScreen renders camera and gallery options (red)
- [ ] Implement `CameraScreen` — expo-camera capture + expo-image-picker for gallery (green)
- [ ] Write tests for image upload progress and result display (red)
- [ ] Implement upload with progress bar, display findings card (green)
- [ ] Add "Add to chat" button — sends photo result context into conversation

### Review gate ⛔
Photo analysis tested with ≥3 real car images. Edge cases (bad file, timeout) handled. → Proceed to Phase 5.

---

## Phase 5: Maintenance Reminders ⬜

### Backend
- [ ] Write tests for `GET/POST/PUT/DELETE /maintenance` (red)
- [ ] Implement MaintenanceRecord model (id, vehicle_id, service_type, last_done_date, last_done_mileage, interval_miles, interval_months, next_due_date, next_due_mileage)
- [ ] Implement maintenance routes with auth middleware (green)
- [ ] Write tests for `GET /maintenance/due` — returns overdue and upcoming items (red)
- [ ] Implement due-soon logic: overdue if past date OR mileage exceeded (green)
- [ ] Implement `POST /maintenance/suggest` — GPT-4o suggests service schedule for vehicle profile

### Mobile
- [ ] Write tests for MaintenanceScreen renders service list (red)
- [ ] Implement `MaintenanceScreen` — list of services with due/overdue status badges (green)
- [ ] Write tests for `useMaintenance` hook — fetch, add, update records (red)
- [ ] Implement `useMaintenance` hook (green)
- [ ] Implement push notifications via expo-notifications for upcoming services
- [ ] Add mileage update prompt on app open ("Update your mileage?")

### Review gate ⛔
Reminder logic tested. Push notifications fire on simulator. → Proceed to Phase 6.

---

## Phase 6: Polish & Harden ⬜

- [ ] Error boundaries in React Native — no silent crashes
- [ ] Global error handler in Express — structured JSON errors, no stack traces in production
- [ ] Input sanitization on all backend routes (zod schemas)
- [ ] OpenAI API key never sent to mobile client — proxy all AI calls through backend
- [ ] Image compression before upload (mobile) — target < 1MB
- [ ] Offline graceful degradation — show cached data, queue actions
- [ ] Add request logging middleware (`morgan`) to backend
- [ ] Performance: lazy-load screens, memoize expensive components
- [ ] Accessibility: ARIA labels, dynamic font scaling, color contrast check
- [ ] Security audit: rate limiting, CORS lockdown, JWT expiry handling

### Review gate ⛔
Full manual QA pass. Zero console errors/warnings in production build. → Proceed to Phase 7.

---

## Phase 7: Ship ⬜

- [ ] Set up PostgreSQL on Railway / Supabase / Render
- [ ] Deploy backend to Railway or Render with env vars
- [ ] Set up TestFlight (iOS) and internal track (Android)
- [ ] Build Expo EAS production build: `eas build --platform all`
- [ ] Submit to TestFlight for internal testing
- [ ] Smoke test full user journey on real device
- [ ] Write onboarding copy for App Store description

---

## Parking Lot 🅿️

Ideas to revisit after v1:
- VIN decoder integration (NHTSA API)
- Voice input for hands-free diagnosis
- Repair cost estimator by ZIP code
- Community mechanic tips by make/model
- Scan tool Bluetooth OBD-II adapter integration

---

## Lessons Learned 📝

_Update this section as you discover conventions, gotchas, or better approaches._

- (none yet)
