# Extending the Prototype

This document describes how to add new request types, personas, and pages to the prototype. It is intended for developers continuing this work.

---

## Adding a new request type

The prototype currently has one fully-implemented form (Relocate Account). Three other request types exist in the mock data (`new-account`, `app-access`, `employee-registration`) but have no submission forms. The steps below apply to any new type.

### 1. Add the type to `src/lib/types.ts`

```typescript
export type RequestType =
  | "relocate-account"
  | "new-account"
  | "app-access"
  | "employee-registration"
  | "your-new-type"; // add here
```

### 2. Add a label in `src/lib/mock-data.ts`

```typescript
export const REQUEST_TYPE_LABELS: Record<Request["type"], string> = {
  "relocate-account": "Relocate Account",
  "new-account": "New Account",
  "your-new-type": "Your New Type Label",
  // ...
};
```

### 3. Add an approval checklist in `src/lib/mock-data.ts`

```typescript
export const APPROVAL_CHECKLISTS: ApprovalChecklist[] = [
  // existing checklists...
  {
    requestType: "your-new-type",
    items: [
      {
        id: "cl-nt-1",
        label: "First checklist item label",
        detail: "Detailed instruction for the approver.",
      },
      // add as many items as needed
    ],
  },
];
```

### 4. Create the form page

Create `src/app/(dashboard)/requests/new/your-type/page.tsx`. Follow the structure of the Relocate Account form:

- Use `useReducer` with a typed state and action union
- Define a `FormState` interface for your form's fields
- Implement step validation in a `validateStep()` function
- Use the existing `StepIndicator` component for the progress bar
- Use the existing `Button`, `Input`, `Select`, `Textarea`, `Card` components
- On submit, show a success toast and redirect to `/dashboard`

### 5. Add a navigation entry in `src/components/layout/app-sidebar.tsx`

If the new type warrants its own nav item, add it to the `NAV_ITEMS` array with appropriate role gating.

### 6. Add a Quick Action card in `src/components/dashboard/quick-actions.tsx`

```typescript
{
  label: "Your New Action",
  description: "Short description of what it does",
  href: "/requests/new/your-type",
  icon: YourIcon,
  roles: ["admin", "user"], // which roles see this card
},
```

### 7. Update the breadcrumb label map in `src/app/(dashboard)/layout.tsx`

```typescript
const LABELS: Record<string, string> = {
  // existing labels...
  "your-type": "Your New Type Label",
};
```

---

## Adding a new mock request

Open `src/lib/mock-data.ts` and add a new entry to the `REQUESTS` array. Follow the existing pattern:

```typescript
{
  id: "REQ-2024-0842",
  type: "new-account",
  status: "pending",
  submittedBy: USERS[0],       // who submitted it
  submittedAt: "2026-05-09T10:00:00.000Z",
  applicant: USERS[6],         // who the request is about
  summary: "Short summary of the request.",
  assignedApproverId: "u-002", // which approver it goes to
  priority: "routine",
  history: makeHistory(USERS[0], USERS[1], "pending", "2026-05-09T10:00:00.000Z"),
},
```

The `makeHistory()` helper generates the standard Submitted → Assigned → Under Review event sequence. For a terminal status (approved or rejected), pass the status as the third argument and a fourth history event will be generated automatically.

---

## Adding a new persona

Personas are defined in `USERS` in `src/lib/mock-data.ts`. Each user must have a unique `id`, `name`, `serviceNumber`, `rank`, `unit`, `location`, `email`, `role`, and `initials`.

```typescript
{
  id: "u-011",
  name: "LTCOL Rebecca Park",
  serviceNumber: "7654321",
  rank: "Lieutenant Colonel",
  unit: "Army Headquarters",
  location: "Victoria Barracks, Melbourne VIC",
  email: "r.park@defence.gov.au",
  role: "admin",
  initials: "RP",
},
```

To make this persona accessible via the role switcher, update `src/lib/role-context.tsx`:

```typescript
const ROLE_TO_USER: Record<Role, User> = {
  admin: USERS[10],    // change to your new user index
  approver: USERS[1],
  user: USERS[2],
};
```

---

## Adding a new page

1. Create the page file at `src/app/(dashboard)/your-page/page.tsx`
2. Add `"use client"` at the top — all dashboard pages are client components
3. Add a nav item in `src/components/layout/app-sidebar.tsx` with the appropriate `href` and role gating
4. Add a label for the path segment in `useBreadcrumbs()` in `src/app/(dashboard)/layout.tsx`
5. If the page URL segment should not appear as a breadcrumb intermediate, add it to `SKIP_SEGMENTS`

---

## Adding a new theme

Themes are defined in two places:

**`src/lib/theme-context.tsx`** — add to `THEME_DEFINITIONS`:

```typescript
{
  id: "your-theme" as const,
  label: "Your Theme Name",
  description: "Short description of the theme's character.",
  preview: {
    sidebar: "#hex",   // sidebar background colour
    bg: "#hex",        // main content background
    primary: "#hex",   // primary accent colour
  },
},
```

**`src/app/globals.css`** — add a theme selector block. Copy the structure of an existing theme and replace all token values. Every block must set both `--color-*` (Tailwind v4 tokens) and `--` (ShadCN aliases) for all design tokens. See the `slate` theme for a complete example.

---

## Connecting a real backend

When moving from prototype to production, the data layer is the primary thing to replace. The component interfaces are already shaped correctly.

**Replace mock data lookups with API calls:**

```typescript
// Before (mock)
const request = getRequestById(id);

// After (production)
const request = await fetch(`/api/requests/${id}`).then(r => r.json());
```

**Key files to update:**

| File | Change |
|---|---|
| `src/lib/mock-data.ts` | Replace with API client functions |
| `src/lib/role-context.tsx` | Replace mock persona lookup with session read (NextAuth, etc.) |
| `src/app/(dashboard)/layout.tsx` | Add auth guard — redirect unauthenticated users to login |
| `src/app/(auth)/login/page.tsx` | Replace demo buttons with real credential form or OAuth flow |

**Typing remains stable.** The TypeScript types in `src/lib/types.ts` define the shape of all entities. As long as the API returns data conforming to those types, consuming components require no changes.
