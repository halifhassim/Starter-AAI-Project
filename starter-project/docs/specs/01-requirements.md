# Login – Requirements

> **What and why.** This document says what the login must do and why. It is
> the source of truth for behaviour: when something needs to change, update this
> document **first**. New to specs? Read [README.md](README.md) first.

## 1. The problem

Budiman Company is building an HR Management System. Before anyone can use
it, the system has to know **who is using it** and **what they are allowed to
do**.

This first part of the system provides:

1. A **login** with each person's company email and password.
2. A **home page for each role**, so everyone lands in the right place.
3. **Protection**, so nobody can open another role's pages.

## 2. Who uses it (roles)

| Role | Home page | What they will do (built in Session 2) |
|------|-----------|-----------------------------------------|
| **Employee** | `/employee` | Apply for leave and track their requests. |
| **Supervisor** | `/supervisor` | Decide on requests sent to them. |
| **HR** | `/hr` | Decide on requests sent to HR. |
| **Admin** | `/admin` | See every leave record. |

Each user has exactly one role.

## 3. User stories

Format: *As a [role], I can [do something] so that [benefit].*

- **U1. Log in** *(Core)*. As any user, I can log in with my company email and
  password so that I see the screens for my role.
- **U2. Land on my home page** *(Core)*. As any user, after logging in I go
  straight to my role's home page, so I don't have to look for it.
- **U3. Stay logged in** *(Core)*. As any user, I stay logged in for up to 7
  days on the same browser, so I don't have to log in every time.
- **U4. Log out** *(Core)*. As any user, I can log out when I'm done, so the
  next person on this computer can't use my account.
- **U5. Only my own screens** *(Core)*. As any user, I can only open the pages
  for my own role, so nobody can act as someone else.

## 4. Acceptance checklist

The login is done when every box is ticked. Use the demo accounts (password
`password123`): `employee-1` and `employee-2` (Employees), `supervisor-1` and
`supervisor-2` (Supervisors), `hr-1` and `hr-2` (HR), `admin` (Admin). Each
logs in with that name followed by `@budiman.com`, e.g.
`employee-1@budiman.com`; the checks below use the short name.
Run `npm run db:seed` first to start from clean data.

| ✓ | ID | Check | How to test |
|---|----|-------|-------------|
| [ ] | AC-01 | A user can log in and lands on their role's home page. | Log in as `employee-1` → you are on `/employee`, a page with only a short hint line. Repeat for `supervisor-1` (`/supervisor`), `hr-1` (`/hr`), `admin` (`/admin`). |
| [ ] | AC-02 | A wrong password or an unknown email shows an error. | Log in as `employee-1` with password `wrong` → "Invalid email or password." Try `nobody@budiman.com` → the same message. |
| [ ] | AC-03 | Empty fields are not sent. | Click **Log in** with the boxes empty → the browser asks you to fill them in. |
| [ ] | AC-04 | A logged-out user is sent to the login page. | Log out, then type `/employee` in the address bar → you are on `/login`. |
| [ ] | AC-05 | A logged-in user who opens the login page goes to their home page. | Logged in as `hr-1`, type `/login` → you are on `/hr`. |
| [ ] | AC-06 | Logging out asks first, then returns to the login page. | Click **Log out** in the top bar → a confirmation box → confirm → you are on `/login`. |
| [ ] | AC-07 | A user cannot open another role's pages. | As `employee-1`, type `/hr` or `/admin` → you are sent back to `/employee`. As `admin`, type `/employee` → you are sent back to `/admin`. |
| [ ] | AC-08 | Every page shows who is logged in. | The top bar shows the user's name and role. On a phone, it shows the user's initials instead. |
| [ ] | AC-09 | An unknown page shows "Not found". | Type `/something-else` → "Not found", with a button back to your home page. |
## 5. Not included (for now)

These are **out of scope** for the login:

- Signing up, or resetting a forgotten password.
- Adding, editing or deleting user accounts (only the demo data creates users).
- The Leave Request module itself – that is what you build in Session 2.

---

# Leave Request – Requirements

> **What and why.** This document says what the Leave Request module must do
> and why. It is the source of truth for behaviour: when something needs to
> change, update this document **first**. It is written **before** the module
> is built (Session 2), following the brief "Agentic AI – Workshop Project
> Brief". The brief left some decisions open on purpose; they are answered
> with assumptions in §6 – change any of them here first, then build.

## 1. The problem

Budiman Company still processes leave on paper: the employee fills in a form,
the supervisor signs it, and HR enters it into a spreadsheet by hand. That
causes three problems:

1. Forms get lost or delayed between the employee, the supervisor and HR.
2. Employees can't see where their request is: waiting for the supervisor,
   with HR, or decided.
3. HR has no single list of requests waiting for a decision.

**Goal:** one request, one system, visible from submission to decision.
Every person sees exactly the requests they need to act on, and nothing gets
lost. A request moves: **employee submits → supervisor signs or rejects →
employee sends to HR → HR approves or rejects → the record stays visible to
Admin**.

## 2. Who uses it (roles)

| Role | Page | What they do in this module |
|------|------|-----------------------------|
| **Employee** | `/employee` | Submits a request to a supervisor in their own division, tracks its status and remarks, and sends a signed request to HR. |
| **Supervisor** | `/supervisor` | Sees the requests sent to them and signs or rejects each one. A rejection needs a reason. |
| **HR** | `/hr` | Sees every request waiting for HR in one shared list and approves or rejects it. A rejection needs a reason, and the officer who decides is recorded. |
| **Admin** | `/admin` | Sees every leave record. View only: changes nothing. |

## 3. User stories

Format: *As a [role], I can [do something] so that [benefit].*

- **U6. Submit a leave request** *(Core)*. As an Employee, I can fill in a
  leave form (type, start date, end date, reason) and send it to a supervisor
  in my own division, so my request officially starts moving.
- **U7. Track my requests** *(Core)*. As an Employee, I can see my current and
  past requests with their status and any remarks, so I always know where my
  request is.
- **U8. Sign or reject what comes to me** *(Core)*. As a Supervisor, I can see
  the requests from my division's employees and sign or reject each one (a
  rejection needs a reason), so I replace the signature on the paper form.
- **U9. Send a signed request to HR** *(Core)*. As an Employee, I can send a
  request once my supervisor has signed it, so HR receives it without me
  carrying a paper copy.
- **U10. Decide from one shared list** *(Core)*. As an HR officer, I can see
  every request waiting for HR and approve or reject it (a rejection needs a
  reason; the officer who decides is recorded), so nothing waits unnoticed.
- **U11. See every record** *(Core)*. As an Admin, I can see all leave records
  in the system without being able to change them, so I have one trustworthy
  view of all leave.
- **U12. Confirm before, told after** *(Core)*. As any user, every action
  (submit, sign, reject, send, approve) asks me to confirm first and shows a
  success message after, so I never change a request by accident.
- **U13. Filter long lists** *(Stretch)*. As an HR officer or Admin, I can
  filter my list by status, so I find what I need as the list grows.
- **U14. Cancel while waiting** *(Stretch)*. As an Employee, I can cancel a
  request that is still waiting for my supervisor, so a mistake doesn't have
  to be signed and corrected.

## 4. Acceptance checklist

The module is done when every box is ticked. Run `npm run db:seed` first to
start from clean data; the seed creates the seven users only, so the checks
below create their own requests as they go — **run them in order**. Demo
accounts (password `password123`, `@budiman.com`): `employee-1` and
`supervisor-1` are both in **Finance & Administration**; `employee-2` and
`supervisor-2` are both in **Software Development**; `hr-1`, `hr-2` are HR;
`admin` is Admin. Leave type here is *Annual*, any near dates.

| ✓ | ID | Check | How to test |
|---|----|-------|-------------|
| [ ] | AC-10 | The form needs every field and refuses bad dates. | Log in as `employee-1` → `/employee` → click **Submit leave request** with the fields empty → the browser asks you to fill them in. Fill in an end date before the start date → a red error appears ("End date cannot be before start date."). Fill in a start date in the past → a red error appears. |
| [ ] | AC-11 | The supervisor list shows only supervisors from my division. | As `employee-1`, open the form → the **Supervisor** dropdown lists only `Supervisor 1 (Finance & Administration)`, not Supervisor 2. Repeat as `employee-2` → the dropdown lists only `Supervisor 2 (Software Development)`. |
| [ ] | AC-12 | An employee can submit a request. | As `employee-1`, submit Annual leave, start tomorrow, end the day after, reason "family event", supervisor `supervisor-1` → success message; the request appears on `/employee` with status **Waiting for supervisor**. |
| [ ] | AC-13 | A supervisor sees only their own division's requests. | Log in as `supervisor-1` → `/supervisor` shows the request from `employee-1`, and nothing from `employee-2`. (First submit a request as `employee-2` to `supervisor-2` and confirm `supervisor-1` never sees it.) |
| [ ] | AC-14 | Signing moves the request to "Signed – waiting to send". | As `supervisor-1`, open `employee-1`'s request → click **Sign** → status becomes **Signed – waiting to send**. |
| [ ] | AC-15 | Rejecting needs a reason, and the employee sees it. | As `supervisor-2`, click **Reject** on `employee-2`'s request with the reason box empty → red error "A reason is required." Enter the reason and reject → status **Rejected by supervisor**; as `employee-2`, the same request shows the reason. |
| [ ] | AC-16 | The employee list shows current and past requests with status and remarks. | As `employee-1`, `/employee` shows a **Current** group (not yet decided) and a **Past** group (approved/rejected), each row showing type, dates, days, status and any remarks. |
| [ ] | AC-17 | Only a signed request can be sent to HR. | As `employee-1`, the request from AC-14 shows a **Send to HR** button; a request still **Waiting for supervisor** does not. Click **Send to HR** → status becomes **Waiting for HR**. |
| [ ] | AC-18 | Both HR officers share one waiting list. | Log in as `hr-1` → `/hr` shows `employee-1`'s request under **Waiting for HR**. Log out, log in as `hr-2` → the same request is there too. |
| [ ] | AC-19 | HR approval records who decided. | As `hr-1`, click **Approve** on the request → status **Approved**; the record shows it was decided by `HR 1`. |
| [ ] | AC-20 | HR rejection needs a reason and records who decided. | `hr-2` submits nothing: instead, as `employee-2`, submit a second request, have `supervisor-2` sign it, send it to HR. As `hr-2`, click **Reject** with the reason empty → red error "A reason is required." Enter a reason → status **Rejected by HR**, reason shown to `employee-2`, record shows decided by `HR 2`. |
| [ ] | AC-21 | A decided request can't be acted on again. | Open the **Approved** record from AC-19 as `hr-2`, as `supervisor-1`, and as `admin` → no **Sign**, **Reject**, **Send to HR** or **Approve** button appears anywhere. |
| [ ] | AC-22 | Admin sees every request, and changes nothing. | Log in as `admin` → `/admin` lists all requests from both divisions with their statuses. The list has no action buttons at all; reopen any request as its owner → its status is unchanged. |
| [ ] | AC-23 | An employee cannot see another employee's requests. | As `employee-2`, the list on `/employee` shows only your own requests — nothing from `employee-1`, and there is no link or button that opens another employee's request. |
| [ ] | AC-24 | Every action asks first, then confirms. | Try each action once (**Submit**, **Sign**, **Reject**, **Send to HR**, **Approve**): a confirmation box appears before the change. Press **Cancel** → nothing changes (recheck the status). Press **Confirm** → a success message appears after. |
| [ ] | AC-25 *(Stretch)* | HR and Admin can filter by status. | As `hr-1` (or `admin`), choose status filter **Approved** → only approved rows remain; choose **All** → everything is back. |
| [ ] | AC-26 *(Stretch)* | An employee can cancel a request still waiting for the supervisor. | As `employee-1`, submit a new request; before it is signed, click **Cancel request** (with confirmation) → the request shows as cancelled and disappears from `supervisor-1`'s list. |

## 5. Not included (for now)

These are **out of scope** for the Leave Request module:

- Editing a request after submitting it, or changing the chosen supervisor.
- Leave balances (remaining days per employee) and quotas.
- Working-day or public-holiday calendars (days are calendar days — A3).
- Email or notification reminders to supervisors and HR.
- Attachments, e.g. a medical certificate.
- Re-assigning a request when the chosen supervisor is away.
- Bulk actions, reports, CSV export, or a printable PDF of the record.

## 6. Assumptions

The brief left these decisions open on purpose. Each answer below is an
assumption you can override — if you change one, update this section **first**,
then the design and the build.

- **A1. Leave types.** A fixed list of four: *Annual*, *Sick*, *Emergency*,
  *Unpaid* (kept with the other allowed-value lists in `src/lib/constants.ts`).
- **A2. Form fields.** Leave type, start date, end date, reason (a required
  text box), and the chosen supervisor. Nothing else — no attachments.
- **A3. Day count.** Calendar days, counting both the start and the end date,
  shown on every row. No one checks the count against any allowance.
- **A4. Statuses.** Exactly six labels, in this order of the workflow:
  **Waiting for supervisor** → **Signed – waiting to send** → **Waiting for
  HR** → **Approved**, with **Rejected by supervisor** and **Rejected by HR**
  as early endings. Only the actions in §3 change a status.
- **A5. Choosing a supervisor.** The dropdown lists the users whose role is
  Supervisor and whose division is exactly the employee's own division;
  exactly one must be chosen at submission.
- **A6. Who decides what.** The requesting employee sends to HR; only the
  chosen supervisor signs or rejects; either HR officer approves or rejects.
- **A7. Remarks.** The reason for a rejection is shown to the requesting
  employee, their supervisor, every HR user, and the Admin.
- **A8. The HR decider.** The name of the HR user who pressed Approve or
  Reject is stored on the record and shown in every list that shows the record.
- **A9. Dates.** Compared in the company time zone (`Asia/Brunei`, see
  `src/lib/constants.ts`); a request must have start ≤ end and cannot start in
  the past (today is allowed).
- **A10. Two HR officers at once.** The first to decide wins; if the second
  one had the page open, their action is refused and they see the decided
  status.
- **A11. Where finished requests go.** Decided requests (approved or rejected)
  move to the **Past** group on every role's page and stay there; nothing is
  archived or deleted.
- **A12. Demo data.** The seed file still creates only the seven users; every
  leave request used in testing is created through the app itself.
