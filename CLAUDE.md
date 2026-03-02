# Project Workspace — workflow (MERN Linear Clone)

## Key Commands
- `npm run install-all` — Install root + backend + frontend dependencies
- `npm start` — Start both backend and frontend concurrently (runs `setup.sh` first via `prestart`)
- `npm run backend` — Start backend only (Express on port 8080)
- `npm run frontend` — Start frontend only (React on port 8000)
- `npm run seed` — Seed the database with sample data
- `npm run test:task1` — Run backend test suite for task 1 (replace 1 with 1–8)
- `npm run build --prefix frontend 2>&1` — Build the frontend
- `cd backend && npm run format:check` — Check backend formatting
- `cd frontend && npm run format:check` — Check frontend formatting

## Architecture Overview

## 1. Architecture

### 1.1 Frontend Architecture

**Folder Structure:**
```
frontend/src/
├── components/
│   ├── admin/
│   │   ├── LogDetailsModal.js
│   │   ├── LogFilters.js
│   │   ├── LogsTable.js
│   │   └── StatusBadge.js
│   ├── issues/
│   │   ├── index.js
│   │   └── IssueFilterDropdown.js
│   ├── ui/
│   │   ├── ActivityDot.js
│   │   ├── Avatar.js
│   │   ├── Badge.js
│   │   ├── Button.js
│   │   ├── Card.js
│   │   ├── CollapsibleSection.js
│   │   ├── ConfirmDialog.js
│   │   ├── DataTable.js
│   │   ├── DetailPanel.js
│   │   ├── Divider.js
│   │   ├── DropdownMenu.js
│   │   ├── DropdownMenuItem.js
│   │   ├── EditableTextarea.js
│   │   ├── EditableTitle.js
│   │   ├── EmptyState.js
│   │   ├── FieldTrigger.js
│   │   ├── IconBadge.js
│   │   ├── IconButton.js
│   │   ├── Input.js
│   │   ├── Label.js
│   │   ├── LoadingScreen.js
│   │   ├── PropertyField.js
│   │   ├── SectionTitle.js
│   │   ├── Spinner.js
│   │   ├── TabNavigation.js
│   │   ├── TeamDisplay.js
│   │   ├── Textarea.js
│   │   └── index.js
│   ├── ActivityList.js
│   ├── ActivityRow.js
│   ├── Breadcrumb.js
│   ├── CommentInput.js
│   ├── CommentsSection.js
│   ├── CreateIssueModal.js
│   ├── ErrorBoundary.js
│   ├── Header.js
│   ├── IssueActivityTimeline.js
│   ├── IssueCard.js
│   ├── IssueProperties.js
│   ├── IssuesBoard.js
│   ├── IssueSidebar.js
│   ├── Layout.js
│   ├── MembersTable.js
│   ├── ProjectActivity.js
│   ├── ProjectModal.js
│   ├── ProjectProperties.js
│   ├── ProjectSidebar.js
│   ├── Sidebar.js
│   ├── SubIssuesSection.js
│   └── UpdateCard.js
├── constants/
│   ├── index.js
│   ├── issueStatus.js
│   ├── priority.js
│   ├── projectStatus.js
│   └── updateStatus.js
├── context/
│   ├── AuthContext.js
│   ├── SidebarContext.js
│   └── TeamsContext.js
├── hooks/
│   ├── index.js
│   ├── useAdminLogs.js
│   ├── useDebounce.js
│   ├── useIssue.js
│   ├── useIssueFilters.js
│   ├── useProjects.js
│   ├── useTeamDetail.js
│   └── useUsers.js
├── icons/
│   └── index.js
├── pages/
│   ├── AdminLogsPage.js
│   ├── IssueDetailPage.js
│   ├── IssuesPage.js
│   ├── LoginPage.js
│   ├── MembersPage.js
│   ├── MyIssuesPage.js
│   ├── ProjectDetailPage.js
│   ├── ProjectsPage.js
│   ├── TeamDetailPage.js
│   └── TeamsPage.js
├── services/
│   ├── adminApi.js
│   └── api.js
├── utils/
│   ├── activityNormalizers.js
│   ├── cn.js
│   ├── formatters.js
│   ├── projectActivityUtils.js
│   ├── statusMapping.js
│   └── utils.js
├── App.js
├── index.css
└── index.js
```

**Component Hierarchy:**
```
App
├── ErrorBoundary
│   └── AuthProvider
│       └── TeamsProvider
│           └── SidebarProvider
│               └── Router
│                   ├── Toaster (react-hot-toast, bottom-right)
│                   └── Suspense (LoadingScreen fallback)
│                       └── Routes
│                           ├── LoginPage (public, no Layout)
│                           └── PrivateRoute → Layout
│                               ├── Sidebar (teams nav, projects, issues)
│                               └── [Page content]
│                                   ├── IssuesPage
│                                   │   ├── Header
│                                   │   ├── IssuesBoard
│                                   │   ├── CreateIssueModal
│                                   │   ├── TabNavigation
│                                   │   └── IssueFilterDropdown
│                                   ├── IssueDetailPage
│                                   │   ├── Header + Breadcrumb
│                                   │   ├── IssueSidebar
│                                   │   ├── IssueProperties
│                                   │   ├── SubIssuesSection
│                                   │   ├── IssueActivityTimeline
│                                   │   ├── CommentsSection + CommentInput
│                                   │   └── ConfirmDialog
│                                   ├── ProjectsPage
│                                   │   ├── Header
│                                   │   └── ProjectModal
│                                   ├── ProjectDetailPage
│                                   │   ├── Header + Breadcrumb
│                                   │   ├── ProjectSidebar
│                                   │   ├── ProjectProperties
│                                   │   ├── ProjectActivity
│                                   │   ├── UpdateCard
│                                   │   └── TabNavigation
│                                   ├── MyIssuesPage
│                                   ├── AdminLogsPage
│                                   │   ├── LogFilters
│                                   │   ├── LogsTable
│                                   │   └── LogDetailsModal
│                                   ├── TeamsPage
│                                   ├── TeamDetailPage → MembersTable
│                                   └── MembersPage → MembersTable
```

**Routing:**

| Path | Component | Auth Required | Description |
|------|-----------|---------------|-------------|
| `/login` | `LoginPage` | No | Login / register entry point |
| `/` | `IssuesPage` | Yes | Default issues board view |
| `/my-issues/:issuesFilter?` | `MyIssuesPage` | Yes | Current user's assigned issues |
| `/projects/all` | `ProjectsPage` | Yes | All projects list |
| `/team/:teamKey/projects/all` | `ProjectsPage` | Yes | Team-scoped projects |
| `/projects/:projectIdentifier/:tab?` | `ProjectDetailPage` | Yes | Project detail with tabs (issues, updates, activities) |
| `/team/:teamKey/:issuesFilter` | `IssuesPage` | Yes | Team-scoped issues board |
| `/issue/:identifier` | `IssueDetailPage` | Yes | Issue detail view |
| `/admin/logs` | `AdminLogsPage` | Yes | API audit logs (admin) |
| `/admin/teams` | `TeamsPage` | Yes | Teams management (admin) |
| `/admin/team/:teamKey/members` | `TeamDetailPage` | Yes | Team members view |
| `/admin/members` | `MembersPage` | Yes | All members list |

All routes except `/login` are wrapped in `PrivateRoute` (redirects to `/login` when unauthenticated). All private pages are wrapped in `Layout` (Sidebar + main content).

**State Management:**
- `AuthContext`: Stores `{ user, token, loading, login(), register(), logout() }`. JWT stored in `localStorage`. On mount, fetches current user via `GET /api/auth/me`. Auto-handles 401 (session expired) via `api.setOnUnauthorized()`.
- `TeamsContext`: Stores `{ teams, loading }`. Fetches all teams on mount when user is authenticated.
- `SidebarContext`: Stores `{ isCollapsed, isMobile, isDrawerOpen, toggleSidebar(), openDrawer(), closeDrawer() }`. Responsive sidebar state management.

**Styling:**
- Tailwind CSS with custom dark theme configuration in `tailwind.config.js`.
- Custom colors: `background` (#0d0d0d), `text-primary` (#e5e5e6), `accent` (#5e6ad2), `status-*`, `priority-*`.
- Class merging via `clsx` + `tailwind-merge` (utility in `src/utils/cn.js`).
- Custom layers in `index.css` for `page-container`, `modal-backdrop`, `dropdown-panel`.
- Font: Inter (system font stack fallback).
- Dark mode is the only theme (hardcoded dark backgrounds).

**API Service:**
- `frontend/src/services/api.js` — Singleton `ApiService` class wrapping `fetch`. Auto-detects base URL by replacing port 8000 with 8080. Attaches Bearer token from localStorage. Namespaced methods: `api.auth.*`, `api.teams.*`, `api.users.*`, `api.projects.*`, `api.issues.*`, `api.comments.*`, `api.issueActivities.*`.

---

### 1.2 Backend Architecture

**Folder Structure:**
```
backend/src/
├── app.js                         (main Express entry point)
├── config/
│   └── database.js                (Mongoose connection)
├── controllers/
│   ├── apiLogController.js
│   ├── authController.js
│   ├── issueController.js
│   ├── projectController.js
│   ├── teamController.js
│   └── userController.js
├── middleware/
│   ├── adminAuth.js               (role=admin guard)
│   ├── apiLogger.js               (placeholder, calls next())
│   ├── auth.js                    (JWT authentication)
│   └── errorHandler.js            (centralized error handling)
├── models/
│   ├── ApiLog.js
│   ├── Comment.js
│   ├── Issue.js
│   ├── IssueActivity.js
│   ├── Project.js
│   ├── ProjectActivity.js
│   ├── ProjectUpdate.js
│   ├── Team.js
│   └── User.js
├── routes/
│   ├── apiLogRoutes.js
│   ├── authRoutes.js
│   ├── issueRoutes.js
│   ├── projectRoutes.js
│   ├── teamRoutes.js
│   └── userRoutes.js
├── services/
│   ├── admin/
│   │   ├── apiLogService.js
│   │   └── index.js
│   ├── auth/
│   │   ├── authService.js
│   │   └── index.js
│   ├── issue/
│   │   ├── commentService.js
│   │   ├── index.js
│   │   ├── issueActivityService.js
│   │   ├── issueHierarchy.js
│   │   └── issueService.js
│   ├── project/
│   │   ├── index.js
│   │   ├── projectActivityService.js
│   │   ├── projectService.js
│   │   ├── projectStatsService.js
│   │   └── projectUpdateService.js
│   ├── team/
│   │   ├── index.js
│   │   └── teamService.js
│   └── user/
│       ├── index.js
│       └── userService.js
└── utils/
    ├── apiLoggerUtils.js
    ├── appError.js                (AppError, BadRequestError, etc.)
    ├── auth.js                    (generateToken helper)
    ├── issuePopulates.js
    ├── projectUtils.js
    ├── sampleApiLogs.js
    ├── seed.js
    └── seeders/
        ├── cleanup.js
        ├── index.js
        ├── data/
        │   ├── commentsData.js
        │   ├── issuesData.js
        │   ├── projectActivitiesData.js
        │   ├── projectUpdatesData.js
        │   ├── projectsData.js
        │   ├── subIssuesData.js
        │   ├── teamsData.js
        │   └── usersData.js
        └── runners/
            ├── commentSeeder.js
            ├── issueSeeder.js
            ├── projectActivitySeeder.js
            ├── projectSeeder.js
            ├── projectUpdateSeeder.js
            ├── teamSeeder.js
            └── userSeeder.js
```

**Database Schemas:**

---
**Model: User** (`users` collection)

| Field | Type | Required | Unique | Default | Notes |
|-------|------|----------|--------|---------|-------|
| `email` | String | Yes | Yes | — | lowercase, trimmed, validated format |
| `password` | String | Yes | No | — | bcrypt hashed, minlength 6 |
| `name` | String | Yes | No | — | trimmed |
| `avatar` | String | No | No | `null` | |
| `role` | String | No | No | `"member"` | enum: `["admin", "member"]` |

Methods: `findByEmail(email)` (static), `comparePassword(candidatePassword)` (instance), `toPublicProfile()` (instance, strips password).

---
**Model: Team** (`teams` collection)

| Field | Type | Required | Unique | Default | Notes |
|-------|------|----------|--------|---------|-------|
| `name` | String | Yes | No | — | trimmed |
| `key` | String | Yes | Yes | — | uppercase, trimmed |
| `icon` | String | No | No | `"📦"` | emoji |
| `color` | String | No | No | `"bg-gray-600"` | Tailwind class |
| `description` | String | No | No | `""` | |
| `members` | [ObjectId] | No | No | `[]` | ref: `User` |

---
**Model: Project** (`projects` collection)

| Field | Type | Required | Unique | Default | Notes |
|-------|------|----------|--------|---------|-------|
| `name` | String | Yes | No | — | trimmed |
| `identifier` | String | Yes | Yes | — | trimmed, auto-generated |
| `description` | String | No | No | `""` | |
| `summary` | String | No | No | `""` | |
| `status` | String | No | No | `"backlog"` | enum: `["backlog","planned","in_progress","completed","cancelled"]` |
| `priority` | String | No | No | `"no_priority"` | enum: `["no_priority","urgent","high","medium","low"]` |
| `team` | ObjectId | Yes | No | — | ref: `Team` |
| `lead` | ObjectId | No | No | `null` | ref: `User` |
| `members` | [ObjectId] | No | No | `[]` | ref: `User` |
| `startDate` | Date | No | No | `null` | |
| `targetDate` | Date | No | No | `null` | |
| `completedDate` | Date | No | No | `null` | |
| `color` | String | No | No | `null` | |
| `icon` | String | No | No | `null` | |
| `creator` | ObjectId | Yes | No | — | ref: `User` |

Indexes: `{ team: 1, status: 1 }`, `{ creator: 1, status: 1 }`.

---
**Model: Issue** (`issues` collection)

| Field | Type | Required | Unique | Default | Notes |
|-------|------|----------|--------|---------|-------|
| `identifier` | String | Yes | Yes | — | e.g. `ENG-42` |
| `title` | String | Yes | No | — | trimmed |
| `description` | String | No | No | `""` | |
| `status` | String | No | No | `"todo"` | enum: `["backlog","todo","in_progress","in_review","done","cancelled","duplicate"]` |
| `priority` | String | No | No | `"no_priority"` | enum: `["no_priority","urgent","high","medium","low"]` |
| `team` | ObjectId | Yes | No | — | ref: `Team` |
| `project` | ObjectId | No | No | `null` | ref: `Project` |
| `assignee` | ObjectId | No | No | `null` | ref: `User` |
| `creator` | ObjectId | Yes | No | — | ref: `User` |
| `parent` | ObjectId | No | No | `null` | ref: `Issue` (sub-issue hierarchy) |
| `labels` | [String] | No | No | `[]` | |
| `estimate` | Number | No | No | `null` | |
| `subscribers` | [ObjectId] | No | No | `[]` | ref: `User` |

Indexes: `{ team: 1, status: 1 }`, `{ project: 1, status: 1 }`, `{ parent: 1 }`, `{ subscribers: 1 }`.

---
**Model: Comment** (`comments` collection)

| Field | Type | Required | Unique | Default | Notes |
|-------|------|----------|--------|---------|-------|
| `issue` | ObjectId | Yes | No | — | ref: `Issue` |
| `user` | ObjectId | Yes | No | — | ref: `User` |
| `content` | String | Yes | No | — | trimmed |
| `isEdited` | Boolean | No | No | `false` | |

Indexes: `{ issue: 1, createdAt: -1 }`.

---
**Model: IssueActivity** (`activities` collection)

| Field | Type | Required | Unique | Default | Notes |
|-------|------|----------|--------|---------|-------|
| `issue` | ObjectId | Yes | No | — | ref: `Issue` |
| `user` | ObjectId | Yes | No | — | ref: `User` |
| `action` | String | Yes | No | — | enum: `["created","updated_status","updated_priority","updated_assignee","updated_title","updated_description","added_comment","updated_comment","deleted_comment","added_label","removed_label","updated_project","updated_parent"]` |
| `changes` | Object | No | No | — | `{ field, oldValue, newValue }` |

Indexes: `{ issue: 1, createdAt: -1 }`.

---
**Model: ProjectActivity** (`projectactivities` collection)

| Field | Type | Required | Unique | Default | Notes |
|-------|------|----------|--------|---------|-------|
| `project` | ObjectId | Yes | No | — | ref: `Project` |
| `user` | ObjectId | Yes | No | — | ref: `User` |
| `action` | String | Yes | No | — | enum: `["updated_status","updated_priority","set_target_date","cleared_target_date","set_start_date","cleared_start_date","updated_lead","cleared_lead","updated_team","updated_members","updated_name","updated_summary","posted_update"]` |
| `changes` | Object | No | No | — | `{ field, oldValue, newValue }` |

Indexes: `{ project: 1, createdAt: -1 }`, `{ user: 1 }`.

---
**Model: ProjectUpdate** (`projectupdates` collection)

| Field | Type | Required | Unique | Default | Notes |
|-------|------|----------|--------|---------|-------|
| `project` | ObjectId | Yes | No | — | ref: `Project` |
| `author` | ObjectId | Yes | No | — | ref: `User` |
| `content` | String | Yes | No | — | trimmed |
| `status` | String | Yes | No | — | enum: `["on_track","at_risk","off_track"]` |

Indexes: `{ project: 1, createdAt: -1 }`, `{ author: 1 }`.

---
**Model: ApiLog** (`apilogs` collection)

| Field | Type | Required | Unique | Default | Notes |
|-------|------|----------|--------|---------|-------|
| `timestamp` | Date | No | No | `Date.now` | indexed |
| `method` | String | Yes | No | — | enum: `["GET","POST","PUT","PATCH","DELETE","OPTIONS","HEAD"]` |
| `path` | String | Yes | No | — | |
| `statusCode` | Number | Yes | No | — | indexed |
| `responseTime` | Number | Yes | No | — | ms |
| `userId` | ObjectId | No | No | `null` | ref: `User` |
| `userEmail` | String | No | No | `null` | |
| `ipAddress` | String | No | No | `null` | |
| `userAgent` | String | No | No | `null` | |
| `requestHeaders` | Mixed | No | No | `{}` | |
| `requestBody` | Mixed | No | No | `null` | |
| `queryParams` | Mixed | No | No | `{}` | |
| `responseBody` | Mixed | No | No | `null` | |
| `errorMessage` | String | No | No | `null` | |
| `errorStack` | String | No | No | `null` | |
| `isSlow` | Boolean | No | No | `false` | indexed |
| `isError` | Boolean | No | No | `false` | indexed |

Indexes: `{ isSlow: 1, isError: 1 }`, `{ timestamp: -1, statusCode: 1 }`.

---

**API Routes:**

| Method | Path | Controller | Auth | Description |
|--------|------|-----------|------|-------------|
| `POST` | `/api/auth/register` | `authController.register` | None | Register new user |
| `POST` | `/api/auth/login` | `authController.login` | None | Login, returns JWT |
| `GET` | `/api/auth/me` | `authController.getCurrentUser` | JWT | Get current user profile |
| `GET` | `/api/users` | `userController.getAllUsers` | JWT | List all users |
| `GET` | `/api/teams` | `teamController.getAllTeams` | JWT | List all teams |
| `GET` | `/api/teams/:identifier` | `teamController.getTeamByIdentifier` | JWT | Get team by key |
| `GET` | `/api/teams/:identifier/members` | `teamController.getTeamMembers` | JWT | Get team members |
| `GET` | `/api/issues/my-issues` | `issueController.getMyIssues` | JWT | Current user's issues |
| `GET` | `/api/issues` | `issueController.getIssues` | JWT | List issues (filterable by team, status, priority, assignee, creator, project) |
| `POST` | `/api/issues` | `issueController.createIssue` | JWT | Create issue |
| `GET` | `/api/issues/:identifier` | `issueController.getIssueByIdentifier` | JWT | Get issue by identifier |
| `PUT` | `/api/issues/:identifier` | `issueController.updateIssue` | JWT | Update issue fields |
| `GET` | `/api/issues/:identifier/valid-parents` | `issueController.getValidParents` | JWT | Get valid parent issues |
| `GET` | `/api/issues/:identifier/activities` | `issueController.getIssueActivities` | JWT | Get issue activity timeline |
| `GET` | `/api/issues/:identifier/comments` | `issueController.getCommentsByIssue` | JWT | List issue comments |
| `POST` | `/api/issues/:identifier/comments` | `issueController.createComment` | JWT | Add comment to issue |
| `PUT` | `/api/issues/:identifier/comments/:id` | `issueController.updateComment` | None* | Edit a comment |
| `DELETE` | `/api/issues/:identifier/comments/:id` | `issueController.deleteComment` | None* | Delete a comment |
| `GET` | `/api/projects` | `projectController.listProjects` | JWT | List projects (filterable by team, status, creator) |
| `POST` | `/api/projects` | `projectController.createProject` | JWT | Create project |
| `GET` | `/api/projects/:identifier` | `projectController.getProjectByIdentifier` | JWT | Get project detail |
| `PUT` | `/api/projects/:identifier` | `projectController.updateProject` | JWT | Update project |
| `GET` | `/api/projects/:identifier/issues` | `projectController.getProjectIssues` | JWT | List project's issues |
| `GET` | `/api/projects/:identifier/updates` | `projectController.getProjectUpdates` | JWT | List project updates |
| `POST` | `/api/projects/:identifier/updates` | `projectController.createProjectUpdate` | JWT | Post project status update |
| `GET` | `/api/projects/:identifier/activities` | `projectController.getProjectActivities` | JWT | Project activity log |
| `GET` | `/api/admin/logs` | `apiLogController.getAdminLogs` | JWT | List API logs (admin) |
| `GET` | `/api/admin/logs/:id` | `apiLogController.getAdminLogById` | JWT | Get single API log |

*Comment update/delete routes currently lack `authenticate` middleware in routing.

**Middleware Stack (applied to `/api/*` via `app.js`):**
1. `express.json()` — JSON body parser.
2. `cors()` — CORS, allows all origins.
3. Per-route `authenticate` — JWT verification (`Authorization: Bearer <token>`), attaches full `req.user` (User document) to request. Returns `401` for missing/invalid/expired tokens.
4. `adminAuth` — Role check (`req.user.role === 'admin'`), returns `403` if not admin. Used on admin routes.
5. `errorHandler` — Centralized error handler (last middleware). Handles `AppError`, Mongoose `ValidationError`, duplicate key (`11000`), `CastError`, and generic 500.

**Error Classes** (`backend/src/utils/appError.js`):
- `AppError(statusCode, message)` — base class
- `BadRequestError(message)` — 400
- `UnauthorizedError(message)` — 401
- `ForbiddenError(message)` — 403
- `NotFoundError(message)` — 404

**Seed Data:**
- 17 users (1 admin: `alex@workflow.dev`, 16 members). Default password: `Password@123` (bcrypt hashed, salt 12).
- 4 teams: Engineering (ENG), Design (DES), Marketing (MKT), Product (PRD).
- Multiple projects, issues (with sub-issues), comments, project activities, and project updates.
- Seed clears all collections before inserting (clean state).
- Run via `node src/utils/seed.js` or `npm run seed`.

---

### 1.3 Testing Strategy

**Backend Tests:**
- **Framework:** Mocha + Chai + chai-http
- **Location:** `backend/test/task1/` through `backend/test/task8/` — each contains `app.spec.js`
- **Reporter:** `mocha-multi-reporters` (spec + mocha-junit-reporter for XML output to `output/`)
- **Database:** Uses `workflow_db_test` (via `NODE_ENV=test`)
- **Pattern:** Each test file connects to the test DB, cleans collections, seeds fixtures, then runs HTTP assertions against `app` (imported from `src/app.js`). Uses `generateToken()` to create test JWT tokens.
- **Environment:** `NODE_ENV=test`, `PORT=8081`
- **Run:** `npm run test:task1` through `npm run test:task8` (from root or backend)

**Frontend Tests:**
- No frontend tests currently exist.
- `jest.config.js` at root is configured for jsdom environment with `identity-obj-proxy` for CSS modules and `babel-jest` transform.

---

## File Manifest

### Backend — Key Files

| Path | Purpose |
|------|---------|
| `backend/src/app.js` | Express app entry point, route registration, middleware, DB connection, server listen |
| `backend/src/config/database.js` | Mongoose connection (workflow_db / workflow_db_test) |
| `backend/src/middleware/auth.js` | JWT authentication middleware (`authenticate`) |
| `backend/src/middleware/adminAuth.js` | Admin role guard (`adminAuth`) |
| `backend/src/middleware/errorHandler.js` | Centralized error handler |
| `backend/src/middleware/apiLogger.js` | API logging middleware (placeholder) |
| `backend/src/utils/appError.js` | Custom error classes |
| `backend/src/utils/auth.js` | JWT token generation (`generateToken`) |
| `backend/src/utils/seed.js` | Database seeder entry point |
| `backend/src/utils/seeders/` | Seed data definitions and runner scripts |

### Frontend — Key Files

| Path | Purpose |
|------|---------|
| `frontend/src/App.js` | Root component, providers, routing, lazy-loaded pages |
| `frontend/src/index.js` | React DOM render entry |
| `frontend/src/index.css` | Tailwind base + custom styles |
| `frontend/src/context/AuthContext.js` | Auth state, login/register/logout, token management |
| `frontend/src/context/TeamsContext.js` | Teams data provider |
| `frontend/src/context/SidebarContext.js` | Sidebar UI state |
| `frontend/src/services/api.js` | API service singleton (fetch wrapper) |
| `frontend/src/components/Layout.js` | Sidebar + main content wrapper |
| `frontend/src/components/ErrorBoundary.js` | React error boundary |
| `frontend/src/utils/cn.js` | Tailwind class merge utility (`clsx` + `tailwind-merge`) |
| `frontend/tailwind.config.js` | Tailwind theme configuration |

---

## Conventions
- Functional React components only (no class components)
- PascalCase for component/page files, camelCase for utility/service/hook files
- ES modules throughout (`import`/`export`) — both backend and frontend
- Backend uses Babel (`@babel/node`) for ES module support with Node.js
- Controller → Service pattern in backend (no separate repository layer)
- Services organized by domain under `backend/src/services/`
- `express-async-errors` for automatic async error forwarding to errorHandler
- Tailwind CSS for all frontend styling (no CSS modules, no styled-components)
- `clsx` + `tailwind-merge` via `cn()` utility for conditional class composition
- `react-hot-toast` for toast notifications
- `lucide-react` for icons
- `recharts` for charts
- `react-datepicker` for date inputs
- React lazy loading for all page components
- JWT auth: `process.env.JWT_SECRET || 'workflow-secret-key-change-in-production'`
- API base URL auto-detected by replacing frontend port (8000) with backend port (8080)

## Testing Patterns
- Backend: Mocha + Chai + chai-http behavioral tests in `backend/test/task*/app.spec.js`
- Tests import `app` from `src/app.js` and use `chai.request(app)` for HTTP assertions
- Test DB cleanup: `Model.deleteMany({})` for all relevant models before each test suite
- Test tokens generated via `generateToken(userId)` utility
- JUnit XML output to `output/` directory via `mocha-junit-reporter`

## Do NOT
- Do NOT use `require`/`module.exports` — the project uses ES modules everywhere
- Do NOT add CSS files — use Tailwind classes exclusively
- Do NOT use axios — the project uses a custom `ApiService` wrapper around native `fetch`
- Do NOT use Redux, Zustand, or other state libraries — use React context + hooks
- Do NOT import from `backend/` in frontend or vice versa
- Do NOT hardcode database names — use `NODE_ENV` to switch between `workflow_db` and `workflow_db_test`
- Do NOT skip the `errorHandler` middleware — throw `AppError` subclasses from services/controllers

## Port Assignments
- Frontend: 8000
- Backend: 8080
- Backend (test): 8081

## Platform Configuration

### hackerrank.yml
- `run`: `npm start` (concurrently runs backend + frontend)
- `install`: `npm run prestart` (runs `setup.sh`)
- `readonly_paths`: `backend/test/*`, `backend/src/config/database.js`, `backend/src/utils/seeders/*`, `backend/src/utils/seed.js`, `setup.sh`
- `default_open_files`: `backend/src/app.js`, `frontend/src/App.js`

### Environment Variables
- `MONGODB_URI` — MongoDB connection string (defaults to `mongodb://127.0.0.1:27017/workflow_db`)
- `JWT_SECRET` — JWT signing secret (defaults to `workflow-secret-key-change-in-production`)
- `PORT` — Backend port (defaults to 8080)
- `NODE_ENV` — `test` switches to `workflow_db_test` database

### .gitignore
Ignores: `node_modules/`, `coverage/`, `.env`, `dist/`, `build/`, `.DS_Store`, `logs/`, `.cursor/`, `.claude/`, `wip/`, `docs/`
