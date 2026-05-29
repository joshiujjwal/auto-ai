# GitHub Copilot Instructions — AutoAI

## Project Overview

AutoAI is a React Native mobile app + Node.js/Express backend that acts as an AI-powered car mechanic. It uses GPT-4o (including vision) for chat diagnosis, OBD code explanation, and photo analysis of car damage.

---

## Stack

| Layer | Technology |
|-------|-----------|
| Mobile | React Native (Expo), TypeScript |
| Backend | Node.js 20, Express, ES Modules |
| AI | OpenAI GPT-4o via `openai` npm package |
| DB | PostgreSQL via `pg` |
| Auth | JWT (jsonwebtoken + bcrypt) |
| Validation | Zod |
| Testing | Jest + Supertest (backend), Jest + RNTL (mobile) |

---

## Coding Conventions

### General
- Async/await everywhere — never `.then()` chains
- Always validate inputs with Zod before processing
- Error responses always: `{ error: string, code: string }` — never raw errors
- No hardcoded strings — use constants for OBD severity levels, service types, etc.

### Backend
- All OpenAI API calls go through `backend/src/services/openai.js` only
- Controllers are thin — call a service, return the result
- All DB access through `backend/src/services/db.js` (pg pool)
- Protect routes with `middleware/auth.js` — never inline JWT verification
- Use `multer` with memory storage for image uploads — delete immediately after use

### React Native / Mobile
- TypeScript strict mode — no `any`
- Screens use custom hooks for data (`useChat`, `useVehicle`, `useMaintenance`, `useOBD`)
- Components are presentational — props only, no direct API calls in components
- `StyleSheet.create()` for all styles — no inline style objects
- AsyncStorage keys always prefixed with `@autoai/`
- Use Expo SDK equivalents instead of bare React Native APIs

---

## Testing Conventions

- Write the test BEFORE the implementation (red/green TDD)
- Test file lives next to source OR in `tests/` folder with matching path
- Test names: `describe('POST /chat', () => { it('returns AI response with vehicle context') })`
- Mock OpenAI calls in tests — never make real API calls in test suite
- Use `supertest` for backend HTTP tests — not fetch/axios
- Use `@testing-library/react-native` for component tests

---

## AI/OpenAI Patterns

- System prompt always includes vehicle context: year, make, model, trim, mileage
- Conversation history trimmed to 20 turns before sending to API
- Images encoded as base64 data URLs for GPT-4o vision calls
- Log token usage (prompt_tokens, completion_tokens) in development
- Handle `openai.APIError` and return 503 with retry guidance to client

---

## Boundaries

- **Do not** make OpenAI API calls directly from mobile — always proxy through backend
- **Do not** store uploaded images permanently — delete after GPT-4o responds
- **Do not** remove or modify existing tests
- **Do not** refactor code outside the scope of the current task
- **Do not** add dependencies without checking Expo SDK compatibility
- **Do not** put secrets or API keys in mobile code or version control
