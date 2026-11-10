# Login – Design

> **How it is built.** This document turns the requirements into a plan a
> builder (person or AI) can follow without asking the business again.
> [01-requirements.md](01-requirements.md) is the source of truth for
> *behaviour*; if the two disagree, fix the requirements first, then this file.
>
> - **Part A – Data design** (sections 1–2): **you own and review this.**
> - **Part B – App design** (sections 3–8): the AI drafts this; you check it.

---

# Part A – Data design (you review)

## 1. Tables

The app stores data in one table so far. Every row gets a number (`id`)
automatically. In Session 2 you add the leave request table here.

### `User` – one row per person who can log in

| Field | Type | Required? | What it holds |
|-------|------|-----------|---------------|
| `id` | number | auto | Unique number for the user |
| `email` | text | yes, unique | Used to log in, stored in lowercase, e.g. `employee-1@budiman.com` |
| `name` | text | yes | Full name shown on screen, with the division in brackets, e.g. "Employee 1 (Finance & Administration)" |
| `passwordHash` | text | yes | Scrambled password – never the real one |
| `role` | text | yes | `EMPLOYEE`, `SUPERVISOR`, `HR` or `ADMIN` |
| `identificationNo` | text | yes | Staff ID: `00-1` then 5 digits, e.g. `00-148291` |
| `designation` | text | yes | Job title, one of the list in rule 4, e.g. `Administrative Officer` |
| `division` | text | yes | Department, one of the list in rule 4, e.g. `Finance & Administration` |
| `createdAt` | date + time | auto | When the user was created |

> **Why is role plain text?** SQLite (our database) does not support a fixed
> list of choices ("enums"). So the allowed values are listed in
> `src/lib/constants.ts`, and the app checks them.

## 2. Business rules

1. **Emails.** Every email is a company address ending in `@budiman.com`. It is
   stored in lowercase and is unique, so login ignores capitals.
2. **Passwords.** Stored only as a hash (bcrypt). A wrong email and a wrong
   password give the **same** message, so nobody can find out which emails exist.
3. **Roles.** Each user has exactly one role. The role decides which pages they
   can open (§4).
4. **Divisions and designations** are chosen from fixed lists, kept in
   `src/lib/constants.ts`:
   - **Division:** Software Development, Quality Assurance, IT Infrastructure &
     Operations, Cybersecurity, Data & Analytics, Product & Design, Customer
     Support, Human Resources, Finance & Administration, Management.
   - **Designation:** Software Engineer, QA Engineer, System Administrator,
     IT Support Technician, Data Analyst, UI/UX Designer, Team Lead, HR Officer,
     Administrative Officer, Director. The designation is only a job title;
     the role decides what someone can do.
5. **Sessions.** A login lasts 7 days on the same browser, then the user must
   log in again.

---

# Part B – App design (AI drafts, you check)

## 3. Technology

Chosen so everyone can run the app on their own laptop with no extra accounts.

| Part | Choice | In plain words |
|------|--------|----------------|
| Web framework | **Next.js** (App Router) + **TypeScript** | Builds both the screens and the server logic in one project. |
| Database | **SQLite** via **Prisma 6** | The database is a single file; Prisma lets the code read and write it. |
| Login | Signed **session cookie** | After login the browser keeps a signed note saying who you are (7 days). JavaScript on the page can't read it (`httpOnly`). |
| Passwords | **bcryptjs** | Stores a scrambled version of each password. |
| Styling | **Tailwind CSS v4** | Styles are written as class names, e.g. `text-sm`. No config file needed. |
| Buttons and forms | **Server actions** | Pressing a button calls a function on the server, which checks the user before doing anything. There are no separate `/api` routes. |

## 4. Screens (routes)

After login, each user is sent to their role's home page. Opening a page for
another role sends you back to your own home page.

| Route | Who | What it shows |
|-------|-----|---------------|
| `/` | anyone | Sends you to your home page, or to `/login`. |
| `/login` | anyone | Email and password form. A logged-in user is sent to their home page. |
| `/employee` | Employee | A page with only a short hint ("This page is empty. Continue here with your own project…") until Session 2. |
| `/supervisor` | Supervisor | A page with only a short hint ("This page is empty. Continue here with your own project…") until Session 2. |
| `/hr` | HR | A page with only a short hint ("This page is empty. Continue here with your own project…") until Session 2. |
| `/admin` | Admin | A page with only a short hint ("This page is empty. Continue here with your own project…") until Session 2. |

A page that doesn't exist shows **Not found**, with a button back to your
home page.

**Layout:** no sidebar. A top bar across every page shows the app name
(Budiman HRMS) on the left. On the right it shows who is logged in (name and
role) and a **Log out** button. On a phone it shows the user's initials
instead of the name and role. The page below the top bar is empty apart from a short hint line.

**Feedback:** **Log out** opens a confirmation box first. Errors are shown in
red above the form.

## 5. Where the code lives

| Folder / file | What's in it |
|---------------|--------------|
| `prisma/schema.prisma` | The tables from Part A. |
| `prisma/migrations/` | Steps that create the tables. |
| `prisma/seed.ts` | The demo users (§6). |
| `src/lib/constants.ts` | Roles, divisions, designations, time zone, date helpers. |
| `src/lib/auth.ts` | Login cookie, `getCurrentUser()`, `requireRole()`, `roleHome()`. |
| `src/lib/db.ts` | The single database connection. |
| `src/app/actions.ts` | Server actions: `loginAction`, `logoutAction`. |
| `src/app/login/` | The login page. |
| `src/app/<role>/` | One folder per role's screens (pages with only a hint for now). |
| `src/components/` | Screen pieces: app frame (top bar), login form and log out button. |
| `src/components/ui/` | Small building blocks used everywhere: Button, Card, PageHeader, EmptyState, Avatar, Icons, form field styles, dialogs, and the overlay page scrollbar. Use these when you build the module. |
| `src/app/globals.css` | Brand colours (the `brand-…` and `navy` colours). Change them here. |

## 6. Demo data

`npm run db:seed` deletes all users and creates:

| Email (`@budiman.com`) | Name | Role | ID no. | Designation | Division |
|----------|------|------|--------|-------------|----------|
| `employee-1@` | Employee 1 (Finance & Administration) | Employee | 00-148291 | Administrative Officer | Finance & Administration |
| `employee-2@` | Employee 2 (Software Development) | Employee | 00-173604 | Software Engineer | Software Development |
| `supervisor-1@` | Supervisor 1 (Finance & Administration) | Supervisor | 00-159827 | Team Lead | Finance & Administration |
| `supervisor-2@` | Supervisor 2 (Software Development) | Supervisor | 00-126418 | Team Lead | Software Development |
| `hr-1@` | HR 1 (Human Resources) | HR | 00-191356 | HR Officer | Human Resources |
| `hr-2@` | HR 2 (Human Resources) | HR | 00-184072 | HR Officer | Human Resources |
| `admin@` | Admin (Management) | Admin | 00-105739 | Director | Management |

All use the password `password123` (workshop only). There are two users for
each of Employee, Supervisor and HR, in different divisions, so the module you
build can be tested with more than one person per role.

## 7. Security (in plain words)

- Every page checks who is logged in and their role (`requireRole`).
- Every server action checks again on the server – hiding a button is not enough.
- The login cookie is signed with `AUTH_SECRET` from `.env`, so it can't be faked.
- Passwords are stored only as a hash.

## 8. Commands

| Command | What it does |
|---------|--------------|
| `npm run dev` | Start the app at http://localhost:3000 |
| `npm run db:migrate` | Create or update the database tables |
| `npm run db:seed` | Reset the demo users |
| `npm run db:studio` | Open a browser view of the database |
| `npm run lint` | Check the code for common mistakes |
| `npm run build` | Check the whole app compiles |

---

# Leave Request – Design

> **How it is built.** This document turns the Leave Request requirements
> ([01-requirements.md](01-requirements.md), second half) into a plan a
> builder (person or AI) can follow without asking the business again.
> That document is the source of truth for *behaviour*; if the two disagree,
> fix the requirements first, then this file.
>
> - **Part A – Data design** (sections 1–4): approved 2026-11-10.
> - **Part B – App design** (sections 5–8): the AI drafts this; you check it.

---

# Part A – Data design (you review)

## 1. Tables

One new table. Every row gets a number (`id`) automatically. People are not
copied into this table — a row only stores their `id`, and the app looks up
the name when it shows the request.

### `LeaveRequest` – one row per leave request

| Field | Type | Required? | What it holds |
|-------|------|-----------|---------------|
| `id` | number | auto | Unique number for the request |
| `employeeId` | number | yes | The `User` who made the request |
| `supervisorId` | number | yes | The `User` chosen to sign it — a Supervisor from the employee's own division |
| `leaveType` | text | yes | `Annual`, `Sick`, `Emergency` or `Unpaid` |
| `startDate` | date | yes | First day of leave |
| `endDate` | date | yes | Last day of leave |
| `reason` | text | yes | Why the employee needs the leave, typed on the form |
| `status` | text | yes | One of the six labels in §2; starts at **Waiting for supervisor** |
| `remarks` | text | no | The reason typed by whoever rejected; stays empty until then |
| `decidedByHrId` | number | no | The HR `User` who pressed Approve or Reject; empty until HR decides |
| `createdAt` | date + time | auto | When the request was submitted |
| `updatedAt` | date + time | auto | When anything on the request last changed |

Three notes:

- **Days are not stored.** The day count is worked out for display only:
  calendar days, counting both the start and the end date (requirement A3).
- **Why is status plain text?** Same reason as `User.role` in §1 of the Login
  design: SQLite has no fixed list of choices ("enums"), so the allowed
  labels live in `src/lib/constants.ts` and the app checks them.
- **Nothing is edited or deleted after submission.** Only the buttons in the
  app change a row, and only ever `status`, `remarks` and `decidedByHrId`
  (requirements A11–A12).

## 2. Statuses

Exactly the six labels from requirement A4. Nobody picks a status by hand —
only the actions in the table below set one.

| Status | Where the request is | Set when |
|--------|----------------------|----------|
| **Waiting for supervisor** | sitting with the chosen supervisor | the employee submits the form |
| **Signed – waiting to send** | supervisor said yes; the employee still has to send it to HR | the supervisor signs |
| **Waiting for HR** | in HR's shared list, waiting for a decision | the employee clicks **Send to HR** |
| **Approved** | HR said yes; `decidedByHrId` is filled | an HR officer approves |
| **Rejected by supervisor** | supervisor said no; `remarks` holds the reason | the supervisor rejects |
| **Rejected by HR** | HR said no; `remarks` holds the reason; `decidedByHrId` is filled | an HR officer rejects |

> **Stretch only:** the cancel button (story U14) needs a seventh label,
> **Cancelled**, for a request still *Waiting for supervisor*. Requirement A4
> says six, so if the stretch is built, update A4 in the requirements first.

## 3. Business rules

1. **The form needs everything.** Leave type, start date, end date, reason
   and a chosen supervisor are all required; an empty form is refused.
2. **Dates.** Start must be on or before end, and the start cannot be in the
   past (today is allowed). Dates are compared in the company time zone
   (`Asia/Brunei`), same as the rest of the app (A9).
3. **The supervisor must be from your division.** The dropdown only lists
   users who are Supervisors in the employee's own division; the choice is
   made at submission and never changes afterwards (A5).
4. **Only the chosen supervisor decides at stage one.** Nobody else can sign
   or reject, and only while the status is *Waiting for supervisor*.
5. **A rejection always has a reason.** Whether it comes from the supervisor
   or from HR, an empty reason box is refused; the reason is stored in
   `remarks` (A7).
6. **Only the employee sends to HR**, and only when the request is
   *Signed – waiting to send*. Nobody else can press that button.
7. **HR decides from the shared list.** Either HR officer can approve or
   reject any request *Waiting for HR*. The first decision counts; if the
   other officer had the page open, their action is refused and they see the
   decided status (A10). Whichever officer decides is stored in
   `decidedByHrId` (A8).
8. **A finished request is closed.** Once a request is Approved or Rejected
   (by either side), no button on any page changes it again — the server
   refuses such an action even if someone sends one (AC-21).
9. **Cancel (stretch only).** Only the employee who made the request can
   cancel it, and only while it is still *Waiting for supervisor* (U14).
10. **Allowed values live in one place.** `leaveType` and `status` values are
    kept with the other lists in `src/lib/constants.ts`, like roles,
    divisions and designations already are.
11. **The seed does not create requests.** `npm run db:seed` still creates
    only the seven users; every request is made through the app itself (A12).

## 4. Who sees what

| Who | Sees | Can do |
|-----|------|--------|
| **Employee** | only their own requests, in **Current** (not yet decided) and **Past** (decided) groups, with status and remarks | submit a request; send their own signed request to HR; cancel while waiting (stretch) |
| **Supervisor** | requests where they are the chosen supervisor, at any stage — including decided ones they handled — except cancelled ones, which disappear from their list | sign or reject while *Waiting for supervisor* |
| **HR** | one shared list: every request *Waiting for HR*, plus the decided ones in **Past**. Requests still with the supervisor are not shown | approve or reject while *Waiting for HR* |
| **Admin** | every request from every division, in any status | nothing — the list has no buttons at all |

Two visibility rules from the requirements:

- The **reason for a rejection** (`remarks`) is shown to the requesting
  employee, their supervisor, every HR user and the Admin (A7).
- The **HR officer who decided** (`decidedByHrId`, shown as their name)
  appears on every list that shows the record (A8).

---

# Part B – App design (AI drafts, you check)

## 5. Screens (one per role action)

The four hint pages become real pages. Each action has its own screen, and a
screen only ever shows the buttons this user may press *right now* (Part A
§3–§4). The top bar and the pieces from `src/components/ui/` (Button, Card,
Field, PageHeader, EmptyState, DetailSection) are reused — no new look.

| Route | Who | What it shows | Actions on it |
|-------|-----|---------------|---------------|
| `/employee` | Employee | Their own requests in **Current** and **Past** groups: type, dates, days, status, remarks. A **Submit leave request** button at the top. | **Send to HR** on a *Signed – waiting to send* row; **Cancel request** on a *Waiting for supervisor* row (stretch). |
| `/employee/new` | Employee | The submit form: leave type, start date, end date, reason, and a supervisor dropdown with only supervisors from the employee's division. | **Submit**. |
| `/supervisor` | Supervisor | The requests chosen for them, Current and Past. | Open a request. |
| `/supervisor/[id]` | Supervisor | One request in full: employee, type, dates, days, reason, status, remarks. | **Sign**, or **Reject** with a reason box, while *Waiting for supervisor*. Nothing once decided. |
| `/hr` | HR | The one list both HR officers share: **Waiting for HR** on top, **Past** below. | Open a request. |
| `/hr/[id]` | HR | One request in full, plus the name of the HR officer who decided, once decided. | **Approve**, or **Reject** with a reason box, while *Waiting for HR*. Nothing once decided. |
| `/admin` | Admin | Every request from every division, with its status. A status filter (stretch). | None — there are no buttons here at all. |
| `/admin/[id]` | Admin | One request, read-only. | None. |

Details:

- An employee has no detail screen of their own: their list row already shows
  everything they may see.
- Opening a detail page of another role sends you back to your home page,
  just like the login pages do (`requireRole` covers these routes too).
- Errors show in red above the form or the reason box, with the exact words
  from the checklist: "End date cannot be before start date." and "A reason
  is required."

## 6. What each button does

One button = one server action (same pattern as `loginAction`). Before
touching the database, every action checks on the server: **who is pressing**
(their role, and that they are the right person for this row) and **whether
the row still has the status this button expects**. Hiding a button is never
enough — the server does the same check again.

| Button | Pressed by | Allowed only when | What it does |
|--------|-----------|-------------------|--------------|
| **Submit** | Employee | form is complete and valid; the chosen supervisor is a Supervisor in the employee's own division | creates the row at **Waiting for supervisor** |
| **Sign** | the chosen supervisor | status is *Waiting for supervisor* | → **Signed – waiting to send** |
| **Reject** (supervisor) | the chosen supervisor | status is *Waiting for supervisor*; the reason box is not empty | → **Rejected by supervisor**; stores the reason |
| **Send to HR** | the requesting employee | status is *Signed – waiting to send* | → **Waiting for HR** |
| **Approve** | any HR officer | status is *Waiting for HR* | → **Approved**; stores who decided |
| **Reject** (HR) | any HR officer | status is *Waiting for HR*; the reason box is not empty | → **Rejected by HR**; stores the reason and who decided |
| **Cancel request** (stretch) | the requesting employee | status is *Waiting for supervisor* | → **Cancelled**; the row leaves the supervisor's list |

**Two HR officers, one decision (A10):** an action updates the row *only if
it still has the status the button expects*. If the other officer decided
first, this update changes nothing: the action is refused and the page shows
the decided status.

## 7. Where the new code lives

Everything new sits next to the login code; nothing moves.

| Folder / file | What's new in it |
|---------------|------------------|
| `prisma/schema.prisma`, `prisma/migrations/` | the `LeaveRequest` table from Part A §1 |
| `src/lib/constants.ts` | `LEAVE_TYPES` and `STATUSES` lists, kept next to roles, divisions and designations |
| `src/lib/leave.ts` | plain helpers: day count (A3), date checks (A9), "may this user act on this row now?" (Part A §3) |
| `src/app/actions.ts` | the server actions for every button in §6, next to `loginAction` |
| `src/app/employee/page.tsx` | the hint page becomes the employee's list |
| `src/app/employee/new/page.tsx` | the submit form page |
| `src/app/supervisor/page.tsx` + `src/app/supervisor/[id]/page.tsx` | the supervisor's list and detail |
| `src/app/hr/page.tsx` + `src/app/hr/[id]/page.tsx` | HR's shared list and detail |
| `src/app/admin/page.tsx` + `src/app/admin/[id]/page.tsx` | the Admin's read-only list and detail |
| `src/components/LeaveRequestForm.tsx` | the submit form (like `LoginForm.tsx`) |
| `src/components/RequestTable.tsx` | shared list rows; each role page decides which buttons (if any) appear |
| `src/components/DecisionButtons.tsx` | Sign/Reject and Approve/Reject with the reason box, used on both detail screens |

## 8. Staying safe (in plain words)

On top of the login rules in §7:

- Every leave page and every leave action starts by asking who is logged in
  and whether their role may be here (`requireRole`), like the login pages.
- Each action checks the current user is **the** employee or **the** chosen
  supervisor of that record — the right role is not enough.
- The status check happens in the database update itself, so a page left
  open from yesterday can never act twice.
- The reason box is checked on the server too, so an empty reason can never
  reach the database even if someone sends one by hand.
- Admin has no server action at all — there is nothing an Admin can press.
