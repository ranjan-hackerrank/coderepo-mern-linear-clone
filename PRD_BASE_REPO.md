# Workflow — Product Requirements Document (Detailed)

**Document purpose:** This PRD describes the Workflow issue-tracking product in full detail so that every feature, flow, validation, and user-facing message is explicit. No implementation details are included; only product behaviour and UX rules.

**Product name:** Workflow
**Version / release:** 1.0
**Last updated:** 2026-03-02

---

## Part A — Definitions and Reference

All terms used in this document are defined below. These definitions are authoritative for the product.

### A.1 Glossary

- **User / Account** — Registered member of a workspace with email-based login. Attrs: id, email, name, avatar, role, created_at.
- **Admin** — A User with `role: admin`. Has access to admin-only pages (Teams management, Members management, API Logs). Can perform all actions a Member can.
- **Member** — A User with `role: member` (default). Can create/update issues, projects, comments. Cannot access admin pages.
- **Team** — Organizational unit grouping Users and Issues/Projects. Attrs: id, name, key (uppercase identifier, e.g. "ENG"), icon (emoji), color (CSS class), description, members[]. A Team is the primary container for Issues; every Issue belongs to exactly one Team.
- **Project** — A container for grouping related Issues within or across Teams. Attrs: id, name, identifier (auto-generated slug), description, summary, status, priority, team, lead, members[], startDate, targetDate, completedDate, color, icon, creator, timestamps. Projects have their own status lifecycle and progress updates.
- **Issue** — A unit of work (task, bug, feature request). Attrs: id, identifier (e.g. "ENG-42"), title, description, status, priority, team, project (optional), assignee (optional), creator, parent (optional — for sub-issues), labels[], estimate, subscribers[], timestamps. Issues are the core entity of the product.
- **Sub-issue** — An Issue with a `parent` reference to another Issue. Sub-issues form a tree hierarchy with a maximum depth of 5 levels. Sub-issues must belong to the same team as their parent.
- **Comment** — User-authored text attached to an Issue. Attrs: id, issue, user, content, isEdited, timestamps.
- **Issue Activity** — An audit-trail record of changes to an Issue. Attrs: id, issue, user, action, changes (field/oldValue/newValue), timestamps. Auto-created on issue creation, status/priority/assignee/project/parent changes, and comment actions.
- **Project Activity** — An audit-trail record of changes to a Project. Attrs: id, project, user, action, changes (field/oldValue/newValue), timestamps. Auto-created on project field updates.
- **Project Update** — A status report posted by a User on a Project. Attrs: id, project, author, content, status (on_track/at_risk/off_track), timestamps. Used for tracking project health over time.
- **API Log** — A record of an API request/response for audit/debugging. Attrs: id, timestamp, method, path, statusCode, responseTime, userId, userEmail, ipAddress, userAgent, request/response details, isSlow, isError.
- **Session** — Authentication session for logged-in users. JWT token stored in localStorage. Token expires after 7 days. On expiry, user sees "Session expired. Please login again." and is redirected to Login.
- **Identifier** — Auto-generated human-readable key for Issues (`{TEAM_KEY}-{sequence}`, e.g. "ENG-42") and Projects (generated from project name).

### A.2 Actor Model and Roles

Roles: **Admin**, **Member**. Each account has exactly one role. New accounts default to `member`. Admin status is set during seeding or manual DB update (no role-change UI in v1.0).

| Capability | Admin | Member |
|------------|-------|--------|
| Browse/view issues, projects, teams | Yes | Yes |
| Create/edit/delete issues | Yes | Yes |
| Create/edit projects | Yes | Yes |
| Post project updates | Yes | Yes |
| Add/edit/delete comments | Yes | Yes |
| Subscribe to issues | Yes | Yes |
| View My Issues | Yes | Yes |
| View admin Teams page | Yes | No |
| View admin Members page | Yes | No |
| View admin API Logs page | Yes | No |

Access control on admin pages: Member attempting to view API Logs sees "Access Denied — You don't have permission to access this page. Admin privileges are required."

### A.3 In Scope for This Version

- Issue tracking with board (Kanban columns) and list views, status/priority management, assignee, labels, estimates.
- Sub-issue hierarchy (parent-child) up to 5 levels deep with circular-reference prevention.
- Team-scoped issue boards with filtering by status category (All, Active, Backlog) and advanced filters (status, priority, assignee, creator, project).
- Project management: creation, status/priority/lead/team/dates/members management, progress updates (on_track/at_risk/off_track), activity timeline.
- Comment system on issues with create, edit, delete.
- Activity timeline on issues showing all changes (status, priority, assignee, project, parent, comments).
- My Issues view with tabs for Assigned, Created, and Subscribed issues.
- Issue subscription (follow/unfollow to track changes).
- Authentication: Login and Registration flows.
- Admin: Teams list, Team members view, All members list, API Logs viewer with filters and detail modal.
- Dark theme UI throughout.
- Responsive design (desktop and mobile).

### A.4 Out of Scope for This Version

- Forgot password / password reset flow (no email integration).
- User profile editing (name, avatar, email changes).
- Role management UI (admin/member role changes).
- Team creation/editing/deletion via UI.
- Issue deletion confirmation with cascading sub-issue deletion (exists but no undo).
- Notifications system (in-app or email).
- Real-time updates (WebSocket/SSE).
- Search across all entities (global search).
- Drag-and-drop on Kanban board.
- File attachments on issues or comments.
- Time tracking / estimation reporting.
- Webhooks or integrations.
- Multi-workspace / multi-tenant support.
- Internationalization / localization beyond English.

---

## Part B — Entities, States, and Business Rules

### B.1 User / Account

| Attribute | Notes |
|-----------|-------|
| id | Internal ObjectId |
| email | Unique, required, validated format, lowercase, trimmed |
| password | Required, min 6 chars, bcrypt hashed (never exposed) |
| name | Required, trimmed |
| avatar | Optional, default null |
| role | `"admin"` or `"member"`, default `"member"` |
| created_at / updated_at | Timestamps |

**Create:** Self-registration via Login page (Register tab). **Delete:** Out of scope for v1.0. **Update:** Out of scope for v1.0 (no profile edit page).

**Validation:**
- Email required; format must match email regex; on invalid format: registration fails with server error.
- Duplicate email on registration: blocks with `"User already exists"`.
- All fields (email, password, name) required on registration; missing any: `"All fields are required"`.
- Password: minimum 6 characters (server-enforced via Mongoose schema).

**Session and access:**
- Logged-in session via JWT stored in `localStorage`. Token expires after 7 days.
- On 401 response from any API call: toast `"Session expired. Please login again."` and auto-logout (clear token, redirect to Login).
- Unauthenticated access to any private route redirects to `/login`.

### B.2 Team

| Attribute | Notes |
|-----------|-------|
| id | Internal ObjectId |
| name | Required, trimmed |
| key | Required, unique, uppercase, trimmed (e.g. "ENG") |
| icon | Emoji string, default "📦" |
| color | Tailwind CSS class string, default "bg-gray-600" |
| description | Optional, default "" |
| members | Array of User references |
| created_at / updated_at | Timestamps |

**Create/update/delete:** Out of scope for v1.0 (managed via seed data only).

**Display rules:** Teams appear in the sidebar navigation. Each team shows its icon, name, and key. Clicking a team navigates to its issues board.

### B.3 Issue

| Attribute | Notes |
|-----------|-------|
| identifier | Auto-generated: `{TEAM_KEY}-{sequence}`, unique |
| title | Required, trimmed |
| description | Optional, default "" |
| status | Required, default "todo" |
| priority | Required, default "no_priority" |
| team | Required, ref Team |
| project | Optional, ref Project |
| assignee | Optional, ref User |
| creator | Required, ref User (set automatically) |
| parent | Optional, ref Issue (for sub-issues) |
| labels | Array of strings |
| estimate | Optional number |
| subscribers | Array of User references |
| created_at / updated_at | Timestamps |

**Status values (exhaustive):**
| Status | Label | Meaning |
|--------|-------|---------|
| `backlog` | Backlog | Not yet prioritized |
| `todo` | Todo | Ready to be worked on |
| `in_progress` | In Progress | Actively being worked on |
| `in_review` | In Review | Work completed, under review |
| `done` | Done | Completed |
| `cancelled` | Cancelled | Will not be done |
| `duplicate` | Duplicate | Duplicate of another issue |

**Status categories for filtering:**
- **All:** All statuses shown.
- **Active:** `in_progress`, `in_review`, `done` (non-backlog, non-cancelled, non-duplicate).
- **Backlog:** `backlog` only.

No explicit status transition restrictions; any status can be set to any other status.

**Priority values (exhaustive):**
| Priority | Label | Visual |
|----------|-------|--------|
| `no_priority` | No priority | Gray dash |
| `urgent` | Urgent | Red alert circle |
| `high` | High | Orange bar chart |
| `medium` | Medium | Yellow bar chart |
| `low` | Low | Gray bar chart |

**Create validation:**
- Title and teamId are required; missing either: `"Title and team are required"`.
- Identifier is auto-generated from team key + sequential count.
- If parent is specified: parent must exist (`"Parent issue not found"`), parent must be in the same team (`"Parent must be in the same team"`), parent depth must not exceed MAX_DEPTH of 5 (`"Sub-issues cannot be nested more than 5 levels deep"`).

**Update validation:**
- Issue must exist; missing: `"Issue not found"`.
- Parent change: cannot set self as parent (`"Issue cannot be its own parent"`), cannot set a descendant as parent (`"Cannot set parent to a descendant issue (would create circular reference)"`), combined depth must not exceed 5 levels.

**Activity tracking:** On create, an `"created"` activity is logged. On update of status, priority, assignee, project, or parent fields, corresponding `"updated_{field}"` activities are logged.

**Delete:** Issue deletion permanently removes the issue and all its sub-issues, comments, and activities. Confirmation dialog: `"This will permanently delete this issue and all its sub-issues, including their comments and activities. This action cannot be undone."` with "Delete" confirmation button.

### B.4 Comment

| Attribute | Notes |
|-----------|-------|
| id | Internal ObjectId |
| issue | Required, ref Issue |
| user | Required, ref User (set automatically) |
| content | Required, trimmed, non-empty |
| isEdited | Boolean, default false |
| created_at / updated_at | Timestamps |

**Create:** Content is required; empty content: `"Content is required"`. An `"added_comment"` activity is logged on the issue.

**Update:** Content is required; empty content: `"Content is required"`. Comment must exist: `"Comment not found"`.

**Delete:** Comment must exist: `"Comment not found"`. Permanently removes the comment.

### B.5 Project

| Attribute | Notes |
|-----------|-------|
| name | Required, trimmed |
| identifier | Auto-generated from name, unique |
| description | Optional, default "" |
| summary | Optional, default "" |
| status | Default "backlog" |
| priority | Default "no_priority" |
| team | Required, ref Team |
| lead | Optional, ref User |
| members | Array of User references |
| startDate | Optional date |
| targetDate | Optional date |
| completedDate | Optional date |
| color | Optional |
| icon | Optional emoji |
| creator | Required, ref User |
| created_at / updated_at | Timestamps |

**Status values (exhaustive):**
| Status | Label |
|--------|-------|
| `backlog` | Backlog |
| `planned` | Planned |
| `in_progress` | In Progress |
| `completed` | Completed |
| `cancelled` | Canceled |

No explicit status transition restrictions; any status can be set to any other status.

**Priority values:** Same as Issue priorities (no_priority, urgent, high, medium, low).

**Create validation:**
- Name and teamId are required; missing either: `"Name and team are required"`.
- Identifier auto-generated from name.

**Update tracking:** All field changes are tracked as Project Activities with specific action types:
| Change | Activity action |
|--------|----------------|
| Status changed | `updated_status` |
| Priority changed | `updated_priority` |
| Target date set | `set_target_date` |
| Target date cleared | `cleared_target_date` |
| Start date set | `set_start_date` |
| Start date cleared | `cleared_start_date` |
| Lead set | `updated_lead` |
| Lead cleared | `cleared_lead` |
| Team changed | `updated_team` |
| Members changed | `updated_members` |
| Name changed | `updated_name` |
| Summary changed | `updated_summary` |
| Update posted | `posted_update` |

**Metrics:** Each project exposes computed metrics: `totalIssues` (count of all issues in project) and `doneIssues` (count of issues with status `"done"`).

### B.6 Project Update

| Attribute | Notes |
|-----------|-------|
| project | Required, ref Project |
| author | Required, ref User |
| content | Required, trimmed, non-empty |
| status | Required, one of: `on_track`, `at_risk`, `off_track` |
| created_at / updated_at | Timestamps |

**Create validation:**
- Content is required and must not be empty: `"Content and status are required"`.
- Status must be valid: `"Invalid status value, must be one of: on_track, at_risk, off_track"`.
- Project must exist: `"Project not found"`.

**Health indicator values:**
| Status | Label | Color |
|--------|-------|-------|
| `on_track` | On track | Green |
| `at_risk` | At risk | Yellow |
| `off_track` | Off track | Red |

**Display:** Latest update determines the project health indicator shown in the projects list. Updates are displayed in reverse chronological order on the project Updates tab, grouped with associated activities.

### B.7 API Log

| Attribute | Notes |
|-----------|-------|
| timestamp | When the API call occurred |
| method | HTTP method (GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD) |
| path | API endpoint path |
| statusCode | HTTP response status code |
| responseTime | Response time in ms |
| userId / userEmail | Requesting user (if authenticated) |
| ipAddress / userAgent | Client info |
| requestHeaders / requestBody / queryParams / responseBody | Request/response payloads |
| errorMessage / errorStack | Error details (if applicable) |
| isSlow / isError | Boolean flags for filtering |

**Access:** Admin role only. Members see "Access Denied" message.

**Display:** Paginated table (50 per page) with filters. Clicking a log row opens a detail modal showing full request/response information.

---

## Part C — Page and Flow Specifications

Every page is specified below with: when it is used, who can access it, every visible element, every user action and its outcome, and every message shown.

### C.1 Login Page

**When used:** User opens the app unauthenticated, or is redirected from a private route.
**Access:** Public. If already logged in, auto-redirects to `/` (Issues page).
**Page title:** "Workflow"

**Elements on the page:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| App logo + "Workflow" header | branding | req'd | Centered above form |
| Login / Register toggle tabs | tab buttons | req'd | Switches between login and register forms |
| Name input | text input | register only | Required for registration; shown only in Register mode |
| Email input | email input | req'd | Placeholder "alice@workflow.dev" |
| Password input | masked text | req'd | Placeholder "••••••••"; uses WebkitTextSecurity disc |
| Sign in / Create account button | primary | req'd | Text changes based on active tab |
| Loading spinner | inline | conditional | Shown during API call |

**User actions and outcomes:**
- User enters valid email and password in Login mode and clicks "Sign in" → authenticated → toast `"Login successful!"` → navigate to `/`.
- User enters invalid credentials in Login mode → toast error with `"Invalid credentials"`.
- User leaves email or password empty → server returns `"Email and password are required"` → toast error.
- User switches to Register tab, fills Name + Email + Password, clicks "Create account" → account created → toast `"Registration successful!"` → navigate to `/`.
- User tries to register with empty name → toast `"Name is required"`.
- User tries to register with existing email → toast `"User already exists"`.
- User tries to register with missing fields → toast `"All fields are required"`.
- Already logged-in user visits `/login` → auto-redirect to `/`.

**No other outcomes.**

### C.2 Issues Page (Team Issues Board)

**When used:** Default landing page after login; user selects a team from sidebar; user navigates to `/team/:teamKey/:filter`.
**Access:** Authenticated (any role).
**Page title:** Team name shown in header breadcrumb.

**Elements on the page:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| Header with team breadcrumb | header | req'd | Shows team icon + name |
| "Add Issue" button | action button | req'd | Opens Create Issue modal |
| Tab navigation (All / Active / Backlog) | tabs | req'd | Filters issues by status category |
| Advanced filter dropdown | filter panel | optional | Filter by status, priority, assignee, creator, project |
| View mode toggle (Board / List) | button group | req'd | Switches between Kanban columns and list view |
| Issues board / list | main content | req'd | Kanban columns grouped by status, or flat list |
| Issue cards | cards in board | req'd | Show identifier, title, priority icon, assignee avatar |

**User actions and outcomes:**
- User clicks "Add Issue" → Create Issue modal opens with team pre-selected and default status "todo".
- User clicks "Add Issue" with no team selected → toast `"Please select a team first"`.
- User clicks a tab (All/Active/Backlog) → issues filtered; URL updates to `/team/{key}/{filter}`.
- User toggles view mode → board switches between column and list layout; URL updates `?view=list` or removes param.
- User clicks an issue card → navigates to `/issue/{identifier}`.
- User applies advanced filters → issues filtered; active filter count shown on dropdown.
- User clicks "+" button on a status column → Create Issue modal opens with that status pre-selected.
- No team in URL → auto-redirects to first team's "all" view.

**No other outcomes.**

### C.3 Issue Detail Page

**When used:** User clicks an issue from any list/board.
**Access:** Authenticated (any role).
**Page title:** Issue identifier shown in header breadcrumb (e.g. "ENG-42").

**Elements on the page:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| Header with team → issue breadcrumb | header | req'd | Clickable team name navigates to team board |
| Options menu (Subscribe / Delete) | dropdown | req'd | Three-dot menu in header |
| Right sidebar toggle button | icon button | req'd | Opens/closes properties sidebar |
| Editable title | inline edit | req'd | Click to edit; saves on blur |
| Parent issue link | breadcrumb | conditional | Shown if issue has a parent; links to parent issue |
| Editable description | inline textarea | req'd | Placeholder "Add description..."; saves on blur |
| Issue properties | property fields | req'd | Status, Priority, Assignee, Project, Parent — inline horizontal display |
| Sub-issues section | collapsible list | conditional | Shows child issues with status icons |
| Activity timeline | activity list | req'd | Shows all tracked changes with user, action, and timestamp |
| Comments section | comment list | req'd | Existing comments with edit/delete options |
| Comment input | textarea + submit | req'd | "Add a comment..." input at bottom |
| Detail sidebar (right panel) | side panel | optional | Full properties panel with all fields |
| Delete confirmation dialog | modal | conditional | Shown on delete action |

**User actions and outcomes:**
- User edits title → saves on blur; toast `"Issue updated"`.
- User edits description → saves on blur; toast `"Issue updated"`.
- User changes status/priority/assignee/project/parent via property fields → API update; toast `"Issue updated"`; activity logged.
- User clicks "Subscribe" in options menu → toggles subscription; toast `"Subscribed to issue"` or `"Unsubscribed from issue"`.
- User clicks "Delete issue" in options menu → confirmation dialog appears with message `"This will permanently delete this issue and all its sub-issues, including their comments and activities. This action cannot be undone."` → on confirm: issue deleted; toast `"Issue deleted"`; navigate to team board.
- User types comment and submits → comment added; toast `"Comment added"`; activity timeline and comments refresh.
- User edits a comment → comment updated inline.
- User deletes a comment → comment removed from list.
- User clicks parent issue link → navigates to parent issue detail page.
- User clicks a sub-issue → navigates to that sub-issue's detail page.
- Update fails (e.g. validation error) → toast with error message (e.g. `"Sub-issues cannot be nested more than 5 levels deep"`).

**No other outcomes.**

### C.4 Create Issue Modal

**When used:** User clicks "Add Issue" on Issues page or Project issues tab, or clicks "+" on a Kanban column.
**Access:** Authenticated (any role).

**Elements on the modal:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| Title input | text input | req'd | Issue title |
| Description input | textarea | optional | |
| Status selector | dropdown | req'd | Pre-selected based on context (default "todo") |
| Priority selector | dropdown | req'd | Default "no_priority" |
| Team selector | dropdown | req'd | Pre-selected if opened from team context |
| Project selector | dropdown | optional | |
| Assignee selector | dropdown | optional | Shows team members |
| Parent issue selector | dropdown | optional | Shows valid parent candidates |
| Labels input | tag input | optional | |
| Create button | primary action | req'd | |
| Cancel / close | secondary | req'd | |

**User actions and outcomes:**
- User fills title, selects team, clicks Create → issue created; toast success; modal closes; issues list refreshes.
- User omits title → server validation: `"Title and team are required"`.
- User selects invalid parent → server validation error shown via toast.

**No other outcomes.**

### C.5 My Issues Page

**When used:** User clicks "My Issues" in sidebar navigation.
**Access:** Authenticated (any role).
**Page title:** "My Issues"

**Elements on the page:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| Header "My Issues" | header | req'd | |
| Tab navigation (Assigned / Created / Subscribed) | tabs | req'd | Filters issues by relationship to current user |
| View mode toggle (Board / List) | button group | req'd | |
| Issues board / list | main content | req'd | Issues matching selected tab filter |

**User actions and outcomes:**
- User clicks "Assigned" tab → shows issues where user is assignee.
- User clicks "Created" tab → shows issues where user is creator.
- User clicks "Subscribed" tab → shows issues where user is in subscribers list.
- User clicks an issue → navigates to Issue Detail page.
- Default tab is "Assigned"; if no filter in URL, auto-redirects to `/my-issues/assigned`.

**No other outcomes.**

### C.6 Projects Page

**When used:** User clicks "Projects" in sidebar navigation or navigates to `/projects/all`.
**Access:** Authenticated (any role).
**Page title:** "Projects"

**Elements on the page:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| Header "Projects" | header | req'd | |
| "Add Project" button | action button | req'd | Opens Create/Edit Project modal |
| "All projects" tab | tab | req'd | Currently only tab |
| Projects table | data table | req'd | Columns: Name (with icon), Team (if all teams view), Health indicator, Priority, Lead avatar, Start date, Target date, Status icon |
| Empty state | message | conditional | `"No projects match your filters. Create a new project to get started."` |

**User actions and outcomes:**
- User clicks "Add Project" → Project modal opens in create mode.
- User clicks a project row → navigates to `/projects/{identifier}`.
- If viewing all projects and no projects exist → empty state message shown.
- Projects show health indicator (on_track/at_risk/off_track/no updates) based on latest Project Update.

**No other outcomes.**

### C.7 Project Detail Page

**When used:** User clicks a project from Projects page.
**Access:** Authenticated (any role).
**Page title:** Project name shown in header breadcrumb.

**Elements on the page:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| Header with team → project breadcrumb | header | req'd | Clickable team navigates to team projects |
| Right sidebar toggle | icon button | req'd | Opens/closes properties sidebar |
| Tab navigation (Overview / Updates / Issues) | tabs | req'd | Three content views |
| Editable project name | inline edit | req'd | On Overview tab |
| Editable project summary | inline textarea | req'd | On Overview tab |
| Properties section | property fields | req'd | Status, Priority, Lead, Start date, Target date, Members |
| Latest update card | card | conditional | On Overview tab; shows most recent Project Update |
| "Write new update" button | action button | req'd | On Overview tab; switches to Updates tab |
| Updates list | timeline | req'd | On Updates tab; reverse chronological |
| Update composer | card with textarea + status selector | req'd | On Updates tab; post new update |
| Project issues board | issues list | req'd | On Issues tab; list view with "Add Issue" button |
| Detail sidebar (right panel) | side panel | optional | Full properties + recent activities |
| Empty state for updates | message | conditional | `"No updates yet. Write the first update to track progress."` |
| Empty state for no updates on Updates tab | message | conditional | `"No updates yet"` |

**User actions and outcomes:**
- User edits project name → saves; toast `"Project updated"`; if name changes, identifier regenerates and URL updates.
- User edits project summary → saves; toast `"Project updated"`.
- User changes any property (status, priority, lead, dates, team, members) → API update; toast `"Project updated"`; activity logged.
- User clicks "Write new update" → switches to Updates tab with focus on update composer.
- User writes update content, selects status (on_track/at_risk/off_track), clicks post → update created; toast `"Update created"`; updates list refreshes.
- User tries to post empty update → toast `"Update content is required"`.
- User clicks "Add Issue" on Issues tab → Create Issue modal opens with project pre-selected.
- User clicks an issue in the issues list → navigates to Issue Detail page.
- Update fails → toast `"Failed to update project"`.

**No other outcomes.**

### C.8 Teams Page (Admin)

**When used:** Admin navigates to `/admin/teams` via sidebar.
**Access:** Authenticated Admin only (no explicit block for Members in route, but shown in admin sidebar section).
**Page title:** "Teams"

**Elements on the page:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| Header "Teams" | header | req'd | |
| Teams table | data table | req'd | Columns: Name (with icon + key), Members count, Created date |

**User actions and outcomes:**
- User clicks a team row → navigates to `/admin/team/{key}/members` showing team members.
- Table shows all teams with their member counts.

**No other outcomes.**

### C.9 Team Detail Page (Admin)

**When used:** Admin clicks a team from Teams page.
**Access:** Authenticated (shows via admin navigation).
**Page title:** Team name.

**Elements on the page:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| Header with team name | header | req'd | |
| Members table | data table | req'd | Shows team members with name, email, avatar, role |

**No other outcomes.**

### C.10 Members Page (Admin)

**When used:** Admin navigates to `/admin/members` via sidebar.
**Access:** Authenticated (shown in admin sidebar).
**Page title:** "Members"

**Elements on the page:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| Header "Members" | header | req'd | |
| Members table | data table | req'd | All users in workspace with name, email, avatar, role, team count |

**No other outcomes.**

### C.11 API Logs Page (Admin)

**When used:** Admin navigates to `/admin/logs` via sidebar.
**Access:** Authenticated Admin only. Member sees: "Access Denied — You don't have permission to access this page. Admin privileges are required."
**Page title:** "API Logs"

**Elements on the page:**
| Element | Type | Required | Rules / behaviour |
|---------|------|----------|-------------------|
| Header "API Logs" | header | req'd | |
| Log filters bar | filter controls | req'd | Filter by method, status code, path, time range, slow/error flags |
| Logs table | data table | req'd | Columns: timestamp, method, path, status code, response time, user |
| Pagination controls | pager | conditional | Page N / Total; prev/next buttons; "Showing X of Y" |
| Log detail modal | modal overlay | conditional | Full request/response details on row click |
| Empty state | message | conditional | `"No logs found"` |
| Error state | error banner | conditional | `"Error loading logs: {error}"` with Retry button |

**User actions and outcomes:**
- Admin views table → logs displayed with 50 per page.
- Admin applies filters → table updates; page resets to 1.
- Admin clears filters → all logs shown.
- Admin clicks a log row → detail modal opens with full request/response payload.
- Admin clicks prev/next page → pagination updates.
- Error loading logs → red error banner with "Retry" button.

**No other outcomes.**

### C.12 Global Navigation (Sidebar)

**When used:** All authenticated pages (wrapped in Layout).
**Access:** Authenticated (any role).

**Elements:**
| Element | Type | Rules / behaviour |
|---------|------|-------------------|
| App logo "Workflow" | branding | Top of sidebar |
| Teams list | nav links | Each team with icon + name; clicking navigates to `/team/{key}/all` |
| "My Issues" link | nav link | Navigates to `/my-issues/assigned` |
| "Projects" link under each team | nav link | Navigates to `/team/{key}/projects/all` |
| All Projects link | nav link | Navigates to `/projects/all` |
| Admin section (Teams, Members, API Logs) | nav links | Visible to all authenticated users (admin restriction on API Logs page only) |
| Sidebar collapse toggle | button | Collapses/expands sidebar |
| Mobile drawer | overlay | Sidebar as drawer on mobile (<640px); closes on outside click |

---

## Part D — User Flows (Step-by-Step)

### D.1 Registration and First Login
1. User → Login page.
2. User → clicks "Register" tab.
3. User → fills Name, Email, Password → clicks "Create account".
4. Application → validates inputs → creates account → generates JWT → stores in localStorage.
5. Application → toast `"Registration successful!"` → navigates to `/` (Issues page).
6. Application → sidebar loads teams → auto-redirects to first team's issues board.

### D.2 Create and Manage an Issue (Canonical Flow)
1. User → Issues page for a team.
2. User → clicks "Add Issue".
3. User → fills title, selects status/priority/assignee → clicks Create.
4. Application → creates issue → shows in board → toast success.
5. User → clicks issue card → Issue Detail page.
6. User → edits title, changes status to "in_progress" → toast `"Issue updated"`.
7. User → adds a comment → toast `"Comment added"`.
8. User → views activity timeline showing creation, status change, and comment events.

### D.3 Create Sub-issue
1. User → Issue Detail page for a parent issue.
2. User → clicks "Create sub-issue" in Sub-issues section.
3. Application → opens Create Issue modal with parent pre-set and team locked to parent's team.
4. User → fills title → clicks Create.
5. Application → validates depth (max 5) → creates sub-issue → sub-issue appears in parent's sub-issues list.

### D.4 Project Lifecycle
1. User → Projects page → clicks "Add Project".
2. User → fills name, selects team, sets lead/dates/priority → clicks Create.
3. Application → creates project → appears in projects list.
4. User → clicks project → Project Detail page.
5. User → Overview tab: edits summary, changes status to "in_progress".
6. User → Updates tab: writes progress update with "on_track" status → posts.
7. Application → project health indicator turns green in projects list.
8. User → Issues tab: adds issues to project.

### D.5 Login with Existing Account
1. User → Login page (Login tab active by default).
2. User → enters email and password → clicks "Sign in".
3. Application → validates credentials → returns JWT.
4. Application → toast `"Login successful!"` → navigates to `/`.

### D.6 Session Expiry
1. User → performing any action requiring authentication.
2. Application → API returns 401 (token expired).
3. Application → toast `"Session expired. Please login again."`.
4. Application → clears localStorage token → redirects to Login page.

---

## Part E — Edge Cases and Product Rules

- **Invalid credentials on login:** Show toast `"Invalid credentials"`.
- **Duplicate email on registration:** Show toast `"User already exists"`.
- **Missing required fields on registration:** Show toast `"All fields are required"`.
- **Empty name on registration:** Show toast `"Name is required"`.
- **Issue not found:** Show `"Issue not found"` and navigate to home.
- **Project not found:** Show toast `"Project not found"` and navigate to `/projects/all`.
- **Team not found:** Show `"Team not found"`.
- **Comment empty on submit:** Server returns `"Content is required"`.
- **Sub-issue depth exceeds 5:** Show `"Sub-issues cannot be nested more than 5 levels deep"`.
- **Circular parent reference:** Show `"Cannot set parent to a descendant issue (would create circular reference)"`.
- **Self as parent:** Show `"Issue cannot be its own parent"`.
- **Parent in different team:** Show `"Parent must be in the same team"`.
- **Empty update content:** Show toast `"Update content is required"`.
- **Invalid update status:** Show `"Invalid status value, must be one of: on_track, at_risk, off_track"`.
- **Non-admin accessing API Logs:** Show "Access Denied" page with message `"You don't have permission to access this page. Admin privileges are required."`.
- **Session expired during any action:** Toast `"Session expired. Please login again."` and redirect to Login.
- **API error on any update:** Toast `"Failed to update issue"` / `"Failed to update project"` / `"Failed to add comment"` respectively.
- **No issues for selected team:** Board shows empty columns.
- **No projects matching filters:** Show `"No projects match your filters. Create a new project to get started."`.
- **No API logs found:** Show `"No logs found"`.
- **Sidebar on mobile:** Sidebar closes automatically when viewport width drops below 640px. Outside click closes the sidebar drawer.

---

## Part F — Non-Functional Requirements (Product and Experience)

- **Performance (product expectation):** Issue board loads and renders within 1 second for up to 100 issues per team. API responses for listing endpoints should return within 500ms under normal load. Project updates and activity timelines load independently and do not block the main page render (lazy loading via separate API calls).
- **Security (product expectation):** Passwords are never displayed or returned in API responses. JWT tokens expire after 7 days. All authenticated endpoints verify the Bearer token. 401 responses trigger automatic logout with user-facing message. Password inputs use visual masking (WebkitTextSecurity disc).
- **Accessibility (product expectation):** All buttons and interactive elements have accessible labels (aria-label). Icons have title attributes. Tab navigation is keyboard-accessible. Loading states show descriptive messages ("Loading...", "Loading page..."). Error states provide clear feedback.
- **Responsive design (product expectation):** Full desktop experience with sidebar navigation. Mobile experience (<640px) with collapsible sidebar drawer. Issues board view supports both column (Kanban) and list layouts. Tables remain usable with horizontal scroll on narrow viewports. Modal dialogs are centered and responsive. Touch targets are appropriately sized.
- **Theme:** Dark theme is the only supported theme. Background: #0d0d0d, text: #e5e5e6, accent: #5e6ad2. All pages use consistent dark styling. No light mode toggle in v1.0.
- **Toast notifications:** All success/error feedback via `react-hot-toast` positioned bottom-right. Dark-themed toasts matching app styling.

---

## Part G — Resolved Open Points (This Version)

- **Role management UI:** Out of scope. Roles are set during seeding or manual DB update. No UI to change user roles.
- **Team CRUD:** Out of scope. Teams are managed via seed data only.
- **Forgot password flow:** Out of scope. No email integration in v1.0.
- **Real-time updates:** Out of scope. No WebSocket/SSE. Users must refresh to see changes from other users.
- **Search:** No global search across entities. Issue filtering is done via status/priority/assignee filters within a team context.
- **Drag-and-drop:** Not available on Kanban board. Status changes are done via property dropdowns.
- **Sub-issue max depth:** Set to 5 levels. Enforced in both create and parent-change validation.
- **JWT expiry:** 7 days. On expiry, user is auto-logged out with message.
- **API Logs data:** Uses sample/mock data in v1.0 (not live API logging middleware).
- **Comment edit/delete access control:** Comment update and delete routes currently lack authentication middleware. Any client with the comment ID can edit/delete. To be fixed in future version.

---

## Part H — References and Document Control

**References:** Product inspired by Linear (linear.app) issue tracking UX patterns. Dark theme and minimal design language follow Linear's design system. Kanban board, issue identifiers (TEAM-N), sub-issue hierarchy, and project updates are modeled after Linear's feature set.

**Change log:**
- 2026-03-02 — Document created from existing codebase analysis. All features, entities, flows, validations, and edge cases documented for v1.0.

**Approval:** This document requires product owner approval before any UX deliverables proceed. Any change to behaviour must be reflected here.
