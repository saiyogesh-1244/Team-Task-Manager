TaskFlow - Team Task Manager
============================

OVERVIEW
--------
TaskFlow is a full-stack team task management web application. It allows teams to create
projects, assign tasks, track progress, and monitor overdue items through a shared dashboard.
It features role-based access control (Admin/Member), secure SSO authentication, and a
real-time kanban board interface.

FEATURES
--------
- Secure authentication via OpenID Connect (OIDC / SSO)
- Role-based access control: Admin and Member roles per project
- Project management: create, edit, and delete projects
- Task management: create, assign, update status, set due dates and priority levels
- Kanban board view: To Do / In Progress / Done columns
- Dashboard: stat cards (total projects, my tasks, in-progress count, overdue count)
- My Tasks panel: tasks assigned to the current user across all projects
- Overdue Tasks panel: tasks past their due date across all projects
- Member management: add/remove members, change roles (admin only)
- REST API with full input validation using Zod schemas
- OpenAPI-first contract with auto-generated React Query hooks

TECH STACK
----------
Frontend:
  - React 18 + Vite
  - TypeScript 5.9
  - TanStack React Query (data fetching and caching)
  - Wouter (client-side routing)
  - Tailwind CSS + shadcn/ui component library
  - React Hook Form + Zod (form validation)
  - date-fns (date formatting)

Backend:
  - Node.js 24 + Express 5
  - TypeScript 5.9
  - PostgreSQL + Drizzle ORM
  - OpenID Connect authentication (openid-client library)
  - Session management stored in PostgreSQL
  - Zod input validation on all API endpoints
  - Pino structured logging

API & Tooling:
  - OpenAPI 3.0 specification (contract-first development)
  - Orval codegen: generates React Query hooks and Zod schemas from OpenAPI spec
  - pnpm workspaces (monorepo)
  - esbuild (API server bundler)

DATABASE SCHEMA
---------------
- users: id, email, firstName, lastName, profileImageUrl, createdAt, updatedAt
- sessions: session storage for authenticated users (server-side)
- projects: id, name, description, ownerId, createdAt, updatedAt
- projectMembers: projectId, userId, role (admin | member), joinedAt
- tasks: id, projectId, title, description, status (todo | in_progress | done),
         priority (low | medium | high), assigneeId, dueDate, createdAt, updatedAt

API ENDPOINTS
-------------
Authentication:
  GET  /api/auth/user    - Get current authenticated user
  GET  /api/login        - Initiate OIDC login flow
  GET  /api/callback     - OIDC callback handler
  GET  /api/logout       - Logout and clear session

Projects:
  GET    /api/projects         - List all projects for current user
  POST   /api/projects         - Create a new project
  GET    /api/projects/:id     - Get a project by ID
  PATCH  /api/projects/:id     - Update project (admin only)
  DELETE /api/projects/:id     - Delete project (admin only)

Members:
  GET    /api/projects/:id/members             - List project members
  POST   /api/projects/:id/members             - Add a member (admin only)
  PATCH  /api/projects/:id/members/:userId     - Update member role (admin only)
  DELETE /api/projects/:id/members/:userId     - Remove a member (admin only)

Tasks:
  GET    /api/projects/:id/tasks               - List tasks (filter by status/assignee)
  POST   /api/projects/:id/tasks               - Create a task
  GET    /api/projects/:id/tasks/:taskId       - Get a single task
  PATCH  /api/projects/:id/tasks/:taskId       - Update a task
  DELETE /api/projects/:id/tasks/:taskId       - Delete a task

Dashboard:
  GET  /api/dashboard/summary         - Summary stats (projects, tasks by status, overdue)
  GET  /api/dashboard/my-tasks        - Tasks assigned to current user
  GET  /api/dashboard/overdue-tasks   - Overdue tasks across all user's projects

ROLE-BASED ACCESS CONTROL
--------------------------
- Any authenticated user can create a project (creator automatically becomes Admin)
- Admin: full access — update/delete project, add/remove members, change member roles
- Member: can view project, create and update tasks, view team members
- All endpoints require authentication; unauthenticated requests return HTTP 401

HOW TO RUN LOCALLY
------------------
Prerequisites: Node.js 24+, pnpm, PostgreSQL

1. Clone the repository

2. Install dependencies:
   pnpm install

3. Set environment variable:
   DATABASE_URL=postgresql://user:password@localhost:5432/taskflow

4. Push the database schema:
   pnpm --filter @workspace/db run push

5. Start the API server:
   pnpm --filter @workspace/api-server run dev

6. Start the frontend:
   pnpm --filter @workspace/task-manager run dev

7. Open http://localhost:80 in your browser

VALIDATION AND SECURITY
------------------------
- All API request bodies and URL parameters validated with Zod schemas
- Sessions stored server-side in PostgreSQL (not in readable cookies)
- PKCE flow used for OIDC authentication (prevents authorization code injection)
- CORS configured with credentials support for cross-origin requests
- Role-based authorization enforced on every mutating endpoint

PROJECT STRUCTURE
-----------------
artifacts/
  api-server/            Express 5 REST API server
    src/
      routes/            Route handlers (auth, projects, members, tasks, dashboard)
      middlewares/       Authentication middleware
      lib/               Auth helpers, logger, shared types
  task-manager/          React + Vite frontend application
    src/
      pages/             Dashboard, Projects, ProjectDetail, ProjectSettings
      components/        UI components and layout
lib/
  db/                    Drizzle ORM schema and database client
  api-spec/              OpenAPI 3.0 specification + Orval config
  api-client-react/      Generated React Query hooks (auto-generated, do not edit)
  api-zod/               Generated Zod validation schemas (auto-generated, do not edit)
  replit-auth-web/       useAuth() React hook for authentication state

DEVELOPMENT COMMANDS
--------------------
pnpm run typecheck                         - Full TypeScript check across all packages
pnpm run build                             - Build all packages
pnpm --filter @workspace/api-spec run codegen  - Regenerate API hooks from OpenAPI spec
pnpm --filter @workspace/db run push       - Apply DB schema changes (development only)
