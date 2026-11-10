# Login – Tasks

> **In what order, and how to check.** Build one task at a time, in order.
> After each task: the AI builds it → a reviewer agent tests it → **you check it
> in the browser**.
>
> If a check fails, **fix the spec first** (requirements or design), then ask
> the AI to rebuild. The requirements and design are the source of truth.
>
> These tasks are **already done** in the starter app – they show what a task
> list looks like. In Session 2, add the Leave Request tasks after Task 6.

**Core and Stretch.** **Core** tasks must be finished first: the app is not
usable without them. **Stretch** tasks are nice extras. Do them only if time is
left, and only if you ask for them. All the login tasks are Core, because login
has to work before anything else.

**Before you start:** `npm install`, then `cp .env.example .env`
(Windows: `copy .env.example .env`).

| # | Task | Core / Stretch | Checks (from requirements §4) |
|---|------|----------------|-------------------------------|
| 1 | Set up the app | Core | – |
| 2 | Database table and demo users | Core | – |
| 3 | Log in and log out | Core | AC-01, AC-02, AC-03, AC-06 |
| 4 | Role protection and home pages | Core | AC-04, AC-05, AC-07 |
| 5 | Top bar | Core | AC-08, AC-09 |
| 6 | Final check and README | Core | All |

---

## Task 1 – Set up the app *(Core)*

- **Goal:** A Next.js + TypeScript app with Tailwind CSS that starts.
- **Files:** `package.json`, `next.config.ts`, `tsconfig.json`,
  `src/app/layout.tsx`, `src/app/page.tsx`, `src/app/globals.css`
  (starts with `@import "tailwindcss";` – no Tailwind config file needed).
- **Check:** `npm run dev` starts, and http://localhost:3000 shows a page.

## Task 2 – Database table and demo users *(Core)*

- **Goal:** Prisma with SQLite, the `User` table from design Part A, and the
  demo users from design §6 with hashed passwords.
- **Files:** `prisma/schema.prisma`, `prisma/migrations/`, `prisma/seed.ts`,
  `src/lib/db.ts`, `src/lib/constants.ts`, `.env.example`, the `db:seed`
  script in `package.json`.
- **Check:** `npm run db:migrate` creates `prisma/dev.db`. `npm run db:seed`
  prints the 7 demo users, and `npm run db:studio` shows them.

## Task 3 – Log in and log out *(Core)*

- **Goal:** Login page, signed session cookie, and logout with a confirmation
  box. After login, go to the role's home page.
- **Files:** `src/lib/auth.ts` (`getCurrentUser`, `startSession`, `endSession`,
  `roleHome`), `src/app/actions.ts` (`loginAction`, `logoutAction`),
  `src/app/login/page.tsx`, `src/components/LoginForm.tsx`,
  `src/components/LogoutButton.tsx`, `src/components/ui/` (Button, Field,
  dialogs).
- **Check:** AC-01, AC-02, AC-03, AC-06.

## Task 4 – Role protection and home pages *(Core)*

- **Goal:** A `requireRole(...)` helper used at the top of every page, so each
  role sees only its own screens, and a home page for each role with only a short hint line.
- **Files:** `src/lib/auth.ts`, `src/app/employee/page.tsx`,
  `src/app/supervisor/page.tsx`, `src/app/hr/page.tsx`,
  `src/app/admin/page.tsx`.
- **Check:** AC-04, AC-05, AC-07.

## Task 5 – Top bar *(Core)*

- **Goal:** The top bar from design §4 (app name, who is logged in, **Log
  out**), and the **Not found** page.
- **Files:** `src/components/AppShell.tsx`, `src/components/LogoutButton.tsx`,
  `src/components/ui/`, `src/app/not-found.tsx`.
- **Check:** AC-08, AC-09.

## Task 6 – Final check and README *(Core)*

- **Goal:** Run `npm run db:seed`, then walk through the whole acceptance
  checklist as each role. Make sure `README.md` explains how to install, run
  and test.
- **Check:** Every AC box ticked; `npm run lint` and `npm run build` pass.

---

# Leave Request – Tasks

> **In what order, and how to check.** Same rules as the login tasks: build
> one task at a time, in order; after each task the AI builds it → a reviewer
> agent tests it → **you check it in the browser**. If a check fails, **fix
> the spec first** ([01-requirements.md](01-requirements.md) or
> [02-design.md](02-design.md)), then ask the AI to rebuild.
>
> The seed still creates only the seven users, so the checks create their own
> requests — **run them in order** (requirements §4). Start each walkthrough
> with a clean slate: `npm run db:seed`.
>
> **Core first, then Stretch.** Tasks 7–15 are Core: the module is not done
> without them. Tasks 16–17 are Stretch: attempt only when every Core check
> passes and time is left.

| # | Task | Core / Stretch | Checks (from requirements §4) |
|---|------|----------------|-------------------------------|
| 7 | Leave table and allowed values | Core | – |
| 8 | Submit a request | Core | AC-10, AC-11, AC-12 |
| 9 | My requests list | Core | AC-16, AC-23 |
| 10 | Supervisor signs or rejects | Core | AC-13, AC-14, AC-15 |
| 11 | Send a signed request to HR | Core | AC-17 |
| 12 | HR decides from the shared list | Core | AC-18, AC-19, AC-20 |
| 13 | Finished is closed; Admin sees all | Core | AC-21, AC-22 |
| 14 | Confirm first, success after | Core | AC-24 |
| 15 | Final check | Core | All |
| 16 | Filter by status | Stretch | AC-25 |
| 17 | Cancel a waiting request | Stretch | AC-26 |

## Task 7 – Leave table and allowed values *(Core)*

- **Goal:** The `LeaveRequest` table from design Part A §1 exists, the leave
  types and status labels live in `src/lib/constants.ts`, and the plain
  helpers (day count, date checks, "may this user act now?") exist in
  `src/lib/leave.ts`. No screens yet — nothing to click.
- **Files:** `prisma/schema.prisma`, `prisma/migrations/`,
  `src/lib/constants.ts`, `src/lib/leave.ts` (new).
- **Check:** `npm run db:migrate` runs without error and `npm run db:studio`
  shows an empty `LeaveRequest` table next to `User`.

## Task 8 – Submit a request *(Core)*

- **Goal:** On `/employee`, a **Submit leave request** button opens the form
  at `/employee/new` (design Part B §5): leave type, start date, end date,
  reason, and a supervisor dropdown listing only supervisors from the
  employee's own division. **Submit** creates the row at *Waiting for
  supervisor*; empty fields are refused and bad dates show the red errors.
- **Files:** `src/app/employee/page.tsx`, `src/app/employee/new/page.tsx`,
  `src/components/LeaveRequestForm.tsx`, `src/app/actions.ts`
  (`submitLeaveRequest`), `src/lib/leave.ts`.
- **Check:** AC-10, AC-11, AC-12.

## Task 9 – My requests list *(Core)*

- **Goal:** `/employee` shows only your own requests in **Current** and
  **Past** groups, each row with type, dates, days, status and remarks.
  Nobody can reach another employee's request.
- **Files:** `src/app/employee/page.tsx`, `src/components/RequestTable.tsx`
  (new).
- **Check:** AC-16, AC-23.

## Task 10 – Supervisor signs or rejects *(Core)*

- **Goal:** `/supervisor` lists only the requests chosen for the logged-in
  supervisor. Opening one shows the full record with **Sign** and **Reject**;
  an empty reason is refused with the red error, and the reason the employee
  later sees is the one typed here.
- **Files:** `src/app/supervisor/page.tsx`, `src/app/supervisor/[id]/page.tsx`,
  `src/components/DecisionButtons.tsx` (new), `src/app/actions.ts` (`sign`,
  `rejectBySupervisor`), `src/lib/leave.ts`.
- **Check:** AC-13, AC-14, AC-15.

## Task 11 – Send a signed request to HR *(Core)*

- **Goal:** A **Send to HR** button appears on the employee's own row only
  when it is *Signed – waiting to send*, and moves it to *Waiting for HR*. No
  other user and no other status can press it.
- **Files:** `src/app/employee/page.tsx`, `src/components/RequestTable.tsx`,
  `src/app/actions.ts` (`sendToHr`).
- **Check:** AC-17.

## Task 12 – HR decides from the shared list *(Core)*

- **Goal:** `/hr` shows one list both HR officers share — **Waiting for HR**
  on top, **Past** below. Opening a request shows the full record with
  **Approve** and **Reject** (empty reason refused); the deciding officer's
  name is stored and shown.
- **Files:** `src/app/hr/page.tsx`, `src/app/hr/[id]/page.tsx`,
  `src/components/DecisionButtons.tsx`, `src/app/actions.ts` (`approve`,
  `rejectByHr`), `src/lib/leave.ts`.
- **Check:** AC-18, AC-19, AC-20.

## Task 13 – Finished is closed; Admin sees all *(Core)*

- **Goal:** A decided request shows **no action button on any page**, and
  the server refuses a late action anyway (two quick HR officers: first
  decision wins). `/admin` lists every request from both divisions with its
  status, and `/admin/[id]` shows one read-only.
- **Files:** the four list pages, the three `[id]` detail pages,
  `src/app/actions.ts` (the status guard in every action),
  `src/lib/leave.ts`.
- **Check:** AC-21, AC-22.

## Task 14 – Confirm first, success after *(Core)*

- **Goal:** Every action (**Submit**, **Sign**, **Reject**, **Send to HR**,
  **Approve**) asks in a confirmation box first and shows a success message
  after — reusing `ConfirmDialog` and `SuccessModal` from `src/components/ui/`.
  Pressing **Cancel** in the box changes nothing.
- **Files:** `src/components/LeaveRequestForm.tsx`,
  `src/components/DecisionButtons.tsx`, `src/components/RequestTable.tsx`,
  `src/components/ui/`.
- **Check:** AC-24.

## Task 15 – Final check *(Core)*

- **Goal:** From a clean seed, walk the whole Leave Request acceptance
  checklist in order as each role, then the whole login checklist too (the
  new pages must not have broken it).
- **Check:** AC-10 … AC-24 every box ticked, plus AC-01 … AC-09 still
  passing; `npm run lint` and `npm run build` pass.

## Task 16 – Filter by status *(Stretch)*

- **Goal:** A status filter (**All** plus each status) on the HR and Admin
  lists; choosing one shows only those rows, **All** brings everything back.
- **Files:** `src/app/hr/page.tsx`, `src/app/admin/page.tsx`,
  `src/components/RequestTable.tsx`.
- **Check:** AC-25.

## Task 17 – Cancel a waiting request *(Stretch)*

- **Goal:** The employee can cancel their own request while it is still
  *Waiting for supervisor* (with confirmation): it moves to **Cancelled**,
  appears in the employee's **Past**, and disappears from the supervisor's
  list. Add the label to `constants.ts` — and update A4 in the requirements
  first, since it currently lists six statuses.
- **Files:** `docs/specs/01-requirements.md` (A4), `src/lib/constants.ts`,
  `src/app/employee/page.tsx`, `src/app/actions.ts` (`cancel`).
- **Check:** AC-26.
