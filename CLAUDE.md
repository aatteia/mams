# MAMS — Developer Reference

## What this project is

A proof-of-concept prototype for a Military Account Management System (MAMS). It demonstrates a role-aware HR request workflow for the Australian Defence Force. There is no database, no real authentication, and no server-side API. All data is client-side mock data defined in `src/lib/mock-data.ts`.

## Actual stack

| Layer | Choice | Notes |
|---|---|---|
| Framework | Next.js 15 (App Router) | Turbopack in dev (`next dev --turbopack`) |
| Language | TypeScript strict | No `any`, no untyped parameters |
| Styling | Tailwind CSS v4 | `@theme inline` tokens, `html[data-theme]` for theming |
| Components | ShadCN on `@base-ui/react` | **Not Radix UI** — see constraints below |
| State | React context + `useReducer` | No Redux, no Zustand |
| Data | Static mock data | `src/lib/mock-data.ts` — no DB calls anywhere |
| Fonts | Geist (Next.js font) | Loaded via `next/font/google` |
| Toast | Sonner | `<Toaster>` in dashboard layout |
| Deployment | Vercel | Auto-deploys from `main` branch |

## Key commands

```bash
npm run dev        # Start dev server (Turbopack, port 3000)
npm run build      # Production build — use this to verify before pushing
npx tsc --noEmit   # Type-check only, no output
```

Note: `db:push`, `db:studio`, and `stripe:listen` scripts exist in `package.json` from the project scaffold but are not relevant to this prototype — there is no database and no Stripe integration.

## Project structure

```
src/
  app/
    (auth)/login/            # Demo login — no real auth
    (dashboard)/layout.tsx   # Shell layout — wraps all dashboard routes
    (dashboard)/dashboard/   # Role-aware dashboard page
    (dashboard)/requests/    # All Requests list with filtering and sorting
    (dashboard)/requests/[id]/    # Request detail and approval
    (dashboard)/requests/new/relocate/  # Multi-step form (4 steps)
    (dashboard)/personnel/   # Directory (admin/approver) or own profile (user)
    (dashboard)/settings/    # Appearance, notifications, role info
    globals.css              # Design tokens and theme selectors
    layout.tsx               # Root layout — ThemeProvider wraps body
  components/
    layout/                  # AppSidebar, TopBar, RoleSwitcher, NotificationBell
    dashboard/               # StatCard, RequestTable, QuickActions, ActivityFeed
    approval/                # ChecklistItem, HistoryTimeline
    forms/                   # StepIndicator, ApplicantSearch
    ui/                      # ShadCN component primitives (button, card, etc.)
  lib/
    mock-data.ts             # Single source of truth for all prototype data
    types.ts                 # All shared TypeScript types
    role-context.tsx         # Active role + user — consumed across all pages
    theme-context.tsx        # Theme switching, localStorage persistence, follow-system
    date-utils.ts            # formatDistanceToNow, formatShortDate, formatDateTime
    utils.ts                 # cn() helper (clsx + tailwind-merge)
```

## Critical constraint — base-ui is not Radix

ShadCN components in this project are built on `@base-ui/react`, not Radix UI. This has several important implications:

- **No `asChild` prop.** Do not write `<Button asChild><Link>`. Style the `<Link>` directly with Tailwind classes instead.
- **`MenuPrimitive.GroupLabel` throws without a `<Menu.Group>` parent.** `DropdownMenuLabel` in this project renders a plain `<div>` for this reason — see `src/components/ui/dropdown-menu.tsx`.
- **`data-popup-open` not `data-state="open"`.** base-ui uses different data attributes than Radix. Use `data-popup-open:` Tailwind variants, not `data-[state=open]:`.
- **`onOpenChange` signature differs.** base-ui passes `(open: boolean, eventDetails: ChangeEventDetails)` — functions with fewer params are compatible due to TypeScript's covariant function parameter rule.

## Theme system

Themes are applied by setting `data-theme` on `<html>`. The default (Snow) uses no attribute.

Each theme selector in `globals.css` sets **both** `--color-*` (Tailwind v4 tokens used by utility classes) and `--` (ShadCN CSS variables used by component base styles). Both must be present for a theme to work correctly.

```css
html[data-theme="slate"] {
  --color-background: ...; /* Tailwind token */
  --background: ...;       /* ShadCN alias */
  /* ... all design tokens ... */
}
```

`suppressHydrationWarning` on `<html>` in `src/app/layout.tsx` prevents a React hydration mismatch when `ThemeProvider` sets `data-theme` client-side after SSR.

## Role system

`RoleProvider` in `src/lib/role-context.tsx` wraps the dashboard layout. It exposes `activeRole`, `activeUser`, and `setRole`. Three fixed personas map to the three roles:

```
admin    → Maj Sarah Chen    (u-001)
approver → WO2 David Hartley (u-002)
user     → Cpl James Nguyen  (u-003)
```

Role switching is demo-only. There is no authentication. Switching role re-renders all role-gated UI instantly.

## Breadcrumb system

Breadcrumbs are derived automatically from the URL pathname in `useBreadcrumbs()` inside `src/app/(dashboard)/layout.tsx`. The `SKIP_SEGMENTS` set excludes path segments that have no corresponding page (e.g. `new` in `/requests/new/relocate`). Non-last crumbs with an `href` render as `<Link>` elements.

## Conventions

- All pages under `(dashboard)/` are `"use client"` — the entire dashboard is client-rendered. There are no RSC data fetches.
- `useSearchParams()` in child pages is covered by the `<Suspense>` boundary wrapping the dashboard layout.
- Form state in the multi-step form (`/requests/new/relocate`) uses `useReducer`. State is not persisted — navigating away loses progress.
- `cn()` from `src/lib/utils.ts` is the only way to compose classNames. Do not use template literals for conditional classes.
- Never hardcode colour values in component className props. Always use design token utility classes (`text-foreground`, `bg-primary`, etc.).

## What is not in this project

- No database — ignore `prisma/`, `src/lib/db.ts`, `src/lib/env.ts`
- No authentication — the login page is a demo entry point only
- No API routes
- No Stripe
- No server actions
- No email sending
