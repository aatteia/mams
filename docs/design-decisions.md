# Design Decisions

This document records the key architectural and design decisions made during the build of the MAMS prototype, including the reasoning behind each choice. It is intended for developers continuing or productionising this work.

---

## Client-side mock data instead of a database

**Decision:** All data lives in `src/lib/mock-data.ts`. There are no API routes, no database queries, and no server actions.

**Reason:** The purpose of the prototype is to validate the UI and workflow, not the data layer. A database would add environment setup, migration management, and seeding complexity that would slow down iteration without adding demonstrable value at this stage. The mock data approach allows the entire prototype to run from `npm run dev` with no configuration.

**Production implication:** In a production system, `mock-data.ts` would be replaced by API calls to a backend service. The component interfaces (`Request`, `User`, `HistoryEvent` in `src/lib/types.ts`) are already shaped to what a real API would return — the data fetch location would change but the consuming components would not.

---

## base-ui instead of Radix UI for ShadCN components

**Decision:** ShadCN components are built on `@base-ui/react` (`@base-ui/react`), not the Radix UI primitives that ShadCN uses by default.

**Reason:** This was the project scaffold's choice. base-ui is the successor to Radix, maintained by the MUI team, and provides a more composable and style-agnostic primitive layer.

**Implications for developers:**
- There is no `asChild` prop. Polymorphic rendering is done by wrapping or styling directly.
- Data attributes differ from Radix: `data-popup-open` instead of `data-[state=open]`.
- `MenuPrimitive.GroupLabel` requires a `<Menu.Group>` ancestor or it throws. `DropdownMenuLabel` in this codebase renders a plain `<div>` for this reason.
- `onOpenChange` callbacks receive `(open: boolean, eventDetails)` — the extra parameter is safe to ignore.

---

## Role context as the only global state

**Decision:** The only piece of shared application state is `activeRole` and `activeUser`, held in `RoleContext` (`src/lib/role-context.tsx`). There is no Redux, Zustand, or other state library.

**Reason:** The prototype has one cross-cutting concern — the active user identity drives what data is shown and what actions are available. Everything else (filter state, form state, checklist state) is local to the component that owns it. Adding a global state library for a single concern would be over-engineering.

**Production implication:** In a production system, the role context would be replaced by a server-side session (NextAuth.js or similar) returning the authenticated user's role and identity. The consuming components would not change — only the source of `activeRole` and `activeUser` would move from a mock lookup to a session read.

---

## `useReducer` for multi-step form state

**Decision:** The four-step Relocate Account form manages state with a single `useReducer` in the page component. State is not persisted to `localStorage` or URL parameters.

**Reason:** `useReducer` with a typed action union makes each state transition explicit and auditable. The alternative — multiple `useState` calls or a form library — would either scatter state across hooks (harder to trace) or add a dependency (React Hook Form, Zod) not justified for one form in a prototype.

**Production implication:** A production form should persist draft state so users do not lose progress on navigation. Options include `localStorage` for simplicity, URL-encoded state for shareability, or a server-side draft model. React Hook Form with Zod validation would be appropriate for a production multi-step form.

---

## Tailwind CSS v4 with `@theme inline` tokens

**Decision:** Design tokens are defined using Tailwind v4's `@theme inline` directive in `globals.css`, not in `tailwind.config.ts` extend.

**Reason:** Tailwind v4 moves token definitions into CSS using the `@theme` directive. This is the idiomatic v4 approach and generates `--color-*` CSS custom properties that are usable both in Tailwind utility classes and directly in CSS.

**Theme system:** Five themes (Snow, Slate, Navy, High Contrast, Eucalyptus) are implemented as `html[data-theme="x"]` CSS selectors. Each selector overrides both `--color-*` tokens (for Tailwind utilities) and `--` shorthand aliases (for ShadCN component base styles). Both sets are required — a theme that only sets one will partially apply.

---

## Breadcrumbs derived from URL, not declared per page

**Decision:** Breadcrumbs are computed automatically from the URL pathname in `useBreadcrumbs()` inside the dashboard layout, using a static label map and a `SKIP_SEGMENTS` set to exclude path segments with no corresponding page.

**Reason:** Declaring breadcrumbs per page would require every page to pass data up to the layout (either via context or layout props), creating coupling. URL-derived breadcrumbs are zero-maintenance — adding a new route automatically gets a breadcrumb.

**Limitation:** Dynamic segment labels are resolved by pattern matching (`REQ-` prefix for request IDs). A more sophisticated implementation would look up the actual entity name (e.g. "REQ-2024-0841 — Relocate Account") by reading the page's data. This is not implemented in the prototype.

---

## Disabled primary action over validation-on-click

**Decision:** Gated primary actions (Approve, Submit request) use `disabled` at the DOM level rather than showing errors on click.

**Reason:** When a button's prerequisite condition is visible on the same screen (checklist completion, confirmation checkbox), making the button inert until the condition is met is clearer than allowing the click and then surfacing an error. The button's disabled state is always accompanied by a caption explaining what is required ("Complete all 5 items to enable approval", "Tick the confirmation above to enable submission").

**Exception:** Validation-on-click is still used for fields that are not visually complete/incomplete in an obvious way (text inputs, selects) — a blank required field is not as visually self-evident as an unchecked checkbox. Those fields validate on the Continue/Submit click.

---

## No mobile-first pass in the prototype

**Decision:** The prototype is designed and tested at desktop width. Responsive layout is present (collapsible sidebar via Sheet, responsive grid) but has not been formally tested or refined at small screen sizes.

**Reason:** The primary stakeholder audience for this prototype will view it on desktop. Spending time on mobile refinement before the design is validated would be premature. A mobile pass is listed in the backlog.

---

## Notifications derived from request history, not a separate data model

**Decision:** The notification bell derives its items by walking `REQUESTS` and extracting relevant `HistoryEvent` records based on the active role, rather than maintaining a separate notifications data structure.

**Reason:** In the prototype, request history events and notifications represent the same underlying facts. A separate notifications table would duplicate data. The derivation function in `NotificationBell` is a pure transformation — it has no side effects and is straightforward to replace with an API call in production.

**Production implication:** A production system would have a dedicated notifications table, populated by server-side event handlers when requests change status. The `NotificationBell` component would fetch from a `/api/notifications` endpoint rather than deriving from local mock data.
