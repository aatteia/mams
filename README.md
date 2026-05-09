# MAMS — Military Account Management System

A proof-of-concept prototype for a Defence Human Resources Information System (HRIS), demonstrating a role-aware request management workflow for military network account operations.

**Live demo:** https://proof-of-concept-defence-hris.vercel.app

---

## What this is

MAMS is a prototype for managing IT-adjacent HR requests within the Australian Defence Force — specifically the lifecycle of network account actions that accompany personnel movements: transfers between units, new enlistments, application access grants, and account registrations.

The prototype demonstrates three distinct user roles, two complete end-to-end workflows, and a full approval pipeline — all running client-side with no database or authentication dependency.

---

## Demo roles

The prototype includes a role switcher in the top-right corner. Three personas are available:

| Role | Persona | What they can do |
|---|---|---|
| **Admin** | Maj Sarah Chen | Submit requests for any personnel, view all queues, access full personnel directory |
| **Approver** | WO2 David Hartley | Review and action requests assigned to them, complete approval checklists |
| **Standard User** | Cpl James Nguyen | Submit and track their own requests, view their own profile |

Switch between roles at any time using the dropdown in the top bar — no logout required.

---

## Key journeys

**Submit a relocation request (Admin or User)**
1. From the dashboard, select **New Request → Relocate Account**
2. Complete the four-step form: Reason → Applicant → Move Details → Confirm
3. Submit — the request appears in the All Requests queue

**Review and approve a request (Approver)**
1. Switch role to Approver
2. From the dashboard or All Requests page, open any pending request
3. Navigate to the **Instructions** tab
4. Complete all checklist items — the Approve button enables when all are checked
5. Approve or reject — the status updates and a history event is recorded

---

## Local development

```bash
# Install dependencies
npm install

# Start the development server
npm run dev
```

The app is available at `http://localhost:3000`. No environment variables are required to run the prototype — all data is client-side mock data.

```bash
# Type-check without building
npx tsc --noEmit

# Production build (verifies no type or compilation errors)
npm run build
```

---

## Tech stack

| Layer | Choice |
|---|---|
| Framework | Next.js 15 (App Router, Turbopack) |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS v4 |
| Components | ShadCN on base-ui (`@base-ui/react`) |
| State | React context + `useReducer` — no external state library |
| Data | Client-side mock data (`src/lib/mock-data.ts`) — no database |
| Deployment | Vercel |

---

## Project structure

```
src/
  app/
    (auth)/login/          # Demo login page
    (dashboard)/
      layout.tsx           # Shell — sidebar, top bar, role provider
      dashboard/           # Role-aware dashboard
      requests/            # All Requests list
      requests/[id]/       # Request detail and approval screen
      requests/new/relocate/ # Multi-step relocation form
      personnel/           # Personnel directory (admin/approver) or own profile (user)
      settings/            # Appearance, notifications, access settings
  components/
    layout/                # AppSidebar, TopBar, RoleSwitcher, NotificationBell
    dashboard/             # StatCard, RequestTable, QuickActions, ActivityFeed
    approval/              # ChecklistItem, HistoryTimeline
    forms/                 # StepIndicator, ApplicantSearch
    ui/                    # ShadCN-derived component primitives
  lib/
    mock-data.ts           # All users, requests, history, checklists
    types.ts               # Shared TypeScript types
    role-context.tsx       # Active role and user context
    theme-context.tsx      # Theme switching and persistence
    date-utils.ts          # Relative and formatted date helpers
```

---

## Deploying

The repository is connected to Vercel. Every push to `main` triggers an automatic production deployment.

To deploy manually:
```bash
npx vercel --prod
```

---

## Prototype scope

This is a demonstration prototype. It has no real authentication, no database, and no server-side logic. All state resets on page refresh. See [`docs/prototype-scope.md`](docs/prototype-scope.md) for a complete list of what is and is not implemented.
