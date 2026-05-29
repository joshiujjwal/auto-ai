# AGENTS.md — AutoAI

Setup and conventions for AI coding agents (Codex, Copilot Workspace, etc.).

---

## Setup

```bash
# Backend
cd backend
npm install
cp .env.example .env
# Fill in OPENAI_API_KEY, DATABASE_URL, JWT_SECRET
npm run db:migrate
npm run db:seed

# Mobile
cd mobile
npm install
cp .env.example .env
# Fill in EXPO_PUBLIC_API_URL=http://localhost:3000
```

---

## Running Tests

```bash
# Backend (always run before making changes)
cd backend && npm test

# Mobile
cd mobile && npm test

# Both — run in separate terminals
```

All tests must pass before and after your changes. If tests were already failing when you started, note it explicitly.

---

## TDD Workflow (Required)

1. **Red**: Write a failing test that describes the behavior you want
2. **Green**: Write the minimum code to make it pass
3. **Refactor**: Clean up without breaking tests
4. Commit each phase separately if the change is substantial

Never write implementation code without a corresponding test.

---

## Code Style

### TypeScript / JavaScript
- Backend: ES Modules (`import`/`export`), Node.js 20+
- Mobile: TypeScript strict mode
- No `any` types in TypeScript — use proper types or `unknown`
- Async/await everywhere — no `.then()` chains
- Named exports preferred over default exports (except React components)

### Naming
- Files: `camelCase.js` / `camelCase.ts`
- React components: `PascalCase.tsx`
- Constants: `SCREAMING_SNAKE_CASE`
- Database columns: `snake_case`
- API routes: `kebab-case` (e.g. `/obd/diagnose`, not `/obdDiagnose`)

### Backend Patterns
- Controllers are thin — delegate all logic to services
- Services are pure functions where possible (easier to test)
- All DB queries go through `services/db.js` — never raw `pg` in controllers
- Validate request body with `zod` at the route level before hitting controller

### React Native Patterns
- Screens own their data fetching via hooks (`useChat`, `useOBD`, etc.)
- Components are presentational — receive data and callbacks as props
- No business logic in components
- Style with `StyleSheet.create()` — no inline style objects
- Use `useMemo` / `useCallback` for expensive operations and stable references

---

## PR Instructions

Every PR must include:

1. **What changed** — one paragraph, plain English
2. **Evidence** — one of:
   - Test output screenshot or paste
   - `npm test` passing output
   - Screenshot of the feature working on simulator/device
3. **Test coverage** — note any new tests added or existing tests updated
4. **Reviewer notes** — anything unusual about the approach

**PR size**: Max ~400 lines changed. If larger, split into smaller PRs.

Do not merge PRs with:
- Failing tests
- `TODO` comments without a linked issue
- `console.log` statements left in production paths
- Hardcoded API keys or secrets

---

## File Ownership

| Area | Location | Notes |
|------|----------|-------|
| AI calls | `backend/src/services/openai.js` | All GPT-4o interactions |
| Prompt templates | `backend/src/utils/promptBuilders.js` | System prompts, vehicle context injection |
| OBD static data | `backend/src/utils/obdDatabase.js` | Local code lookup table |
| Auth logic | `backend/src/middleware/auth.js` | JWT verify only |
| DB queries | `backend/src/services/db.js` | pg pool wrapper |
| API client | `mobile/src/services/api.js` | Axios instance with base URL + auth header |
| Navigation | `mobile/src/App.tsx` | Stack navigator root |

---

## Out of Scope (Do Not Touch Without Explicit Request)

- Do not change the database schema without updating the migration files
- Do not add new npm packages without confirming they work with Expo SDK version
- Do not refactor working code unless explicitly asked
- Do not remove or skip existing tests
- Do not send images directly from mobile to OpenAI — always proxy through backend
