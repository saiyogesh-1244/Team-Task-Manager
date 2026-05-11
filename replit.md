# TaskFlow — Team Task Manager

A full-stack task management app for teams: create projects, assign tasks, track status, and view overdue items on a shared dashboard.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/task-manager run dev` — run the frontend (port auto-assigned)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string, `SESSION_SECRET` — session signing key

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5 (artifact: `artifacts/api-server`, base path: `/api`)
- Frontend: React + Vite (artifact: `artifacts/task-manager`, base path: `/`)
- DB: PostgreSQL + Drizzle ORM
- Auth: Replit Auth (OIDC), sessions stored in DB (`sessions` table)
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec at `lib/api-spec/openapi.yaml`)
- Build: esbuild (CJS bundle for API)

## Where things live

- `lib/db/src/schema/` — Drizzle schema (auth.ts, projects.ts, members.ts, tasks.ts)
- `lib/api-spec/openapi.yaml` — OpenAPI spec (source of truth for API contracts)
- `lib/api-client-react/src/generated/api.ts` — generated React Query hooks (do not edit manually)
- `lib/api-zod/src/generated/api.ts` — generated Zod schemas (do not edit manually)
- `artifacts/api-server/src/routes/` — Express route handlers (auth, projects, members, tasks, dashboard)
- `artifacts/task-manager/src/pages/` — React pages (dashboard, projects, project-detail, project-settings)
- `lib/replit-auth-web/src/use-auth.ts` — `useAuth()` hook (not composite — consumed from src)

## Architecture decisions

- Contract-first API: OpenAPI spec → Orval codegen → typed React Query hooks + Zod schemas.
- Sessions stored in DB (not cookie-signed), managed via `lib/db` `sessionsTable`.
- `lib/replit-auth-web` exports from `src` directly (not composite) so Vite consumers resolve `import.meta.env`.
- Role-based access: `projectMembers.role` is `admin | member`; enforced per-route in the API.
- Orval config uses `mode: "single"` for Zod output to avoid duplicate export conflicts.

## Product

- **Dashboard**: stats (total projects, my tasks, in-progress, overdue), my task list, overdue task list.
- **Projects**: list all projects you belong to; create new projects.
- **Project Detail**: Kanban-style board with To Do / In Progress / Done columns; create, edit, move, and delete tasks.
- **Project Settings** (admin only): rename/re-describe project, manage members (add/remove/change role), delete project.

## User preferences

- No emojis in UI or code unless explicitly requested.
- Keep API contracts in OpenAPI first, then regenerate with Orval — never edit generated files.

## Gotchas

- Always run `pnpm --filter @workspace/api-spec run codegen` after changing `openapi.yaml`.
- Always run `pnpm --filter @workspace/db run push` after changing schema files.
- `lib/replit-auth-web` must NOT be added to root `tsconfig.json` references (uses `import.meta.env`).
- The shared proxy routes `/api` to port 8080 and `/` to the task-manager Vite port.
- Never call service ports directly (e.g. `localhost:8080`); use `localhost:80/api/...` for curl.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.
