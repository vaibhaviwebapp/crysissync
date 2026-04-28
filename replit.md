# CrisisSync AI — Emergency Response System for Hospitality

## Overview
A single-page web MVP for a hotel-grade crisis response platform. Three role-based portals (Guest, Staff/Responder, Admin) share a real-time picture of incidents, evacuation routes, AI threat detections, and audit-grade chained logs.

## Architecture
- **Monorepo:** pnpm workspace.
- **API (`artifacts/api-server`)** — Express 5 + Socket.IO 4, drizzle-orm/pg on Replit Postgres, JWT auth (bcrypt), tamper-evident hash-chained audit log, in-memory building map (3 floors × 6×4 grid) with Dijkstra evacuation routing, simulated AI module (fire/crowd detection + per-floor risk score), drill mode that tags incidents/alerts as `DRILL`.
  - Mounts `/api/*` and `/socket.io` (declared in `.replit-artifact/artifact.toml`).
  - Real-time events: `incident:new`, `incident:updated`, `alert:new`, `task:new`, `task:updated`, `log:new`, `trigger_siren` (`{status: "ON"|"OFF", type, timestamp}`).
- **Web (`artifacts/crisissync`)** — React 19 + Vite + Tailwind v4 + shadcn UI, wouter routing, TanStack Query, generated Orval client (`@workspace/api-client-react`), socket.io-client, recharts for analytics.
  - Theming: deep navy/control-tower aesthetic with signal amber, alert red, signal green.
  - `lib/socket.tsx` global SocketProvider invalidates queries and triggers Web Audio siren + speech synthesis on alerts.
  - Offline mode: panic alerts queued in localStorage, simulated SMS log to front desk; auto-flushed on `online` event.
  - Voice alerts gated by user toggle (uses `SpeechSynthesisUtterance`).

## Seed users (password shown)
- `admin@crisissync.demo` / `admin1234` — full command center
- `staff@crisissync.demo` / `staff1234` — incident triage + tasks
- `responder@crisissync.demo` / `respond1234` — same as staff
- `guest@crisissync.demo` / `guest1234` — Room F2-R201, Floor 2

## Routes
- Guest: `/guest` (panic), `/guest/evacuate` (Dijkstra route + floor maps)
- Staff/Responder: `/staff` (active incidents + broadcast), `/staff/tasks`, `/staff/floors`
- Admin: `/admin` (KPIs, building map, AI risk), `/admin/incidents`, `/admin/tasks`, `/admin/analytics`, `/admin/ai`, `/admin/drill`, `/admin/logs`

## Important conventions
- All Orval hooks return `T` directly — invalidate via `getXxxQueryKey()`. Mutations: `mutate({ data: {...} })`.
- `lib/api-zod` re-exports types under `Types` namespace to avoid Orval duplicate-export errors.
- `req.user` augmented via `declare global { namespace Express { interface Request { user?: JwtPayload } } }` (Express 5 ESM).
- `AiDetection` extends `Record<string, unknown>` for hash-chain `details` arg.
- After schema changes always run `pnpm -w run typecheck:libs` to rebuild lib dist .d.ts.

## Building
- 3 floors, ground floor has lobby + 2 exits, stairs preferred (Dijkstra weight 2) over elevators (weight 8). Hazards (active incident locations) excluded from path.
