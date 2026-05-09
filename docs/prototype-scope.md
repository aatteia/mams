# Prototype Scope

This document describes what is and is not implemented in the MAMS prototype. It is intended for stakeholders reviewing the prototype and for developers planning further work.

---

## What this prototype demonstrates

MAMS is a simulation of a Defence HR request management system. It is fully interactive and navigable but contains no real data, no real authentication, and no server-side processing. Its purpose is to validate the user interface, workflow logic, and role-based interaction model before investing in a production implementation.

---

## Implemented features

### Authentication and roles
- Demo login page with one-click role selection (no credentials required)
- Three distinct roles with separate views and permissions: Admin, Approver, Standard User
- Role switcher in the top bar — switch roles at any time without logging out
- All pages and components are role-aware — content, navigation, and available actions adapt per role

### Dashboard
- Role-specific summary statistics (Pending Approvals, My Requests, Team Requests)
- Quick action cards appropriate to each role
- Full request table with sortable columns (Request ID, Type, Applicant, Status, Priority, Submitted)
- Recent Activity sidebar with timestamped events and clickable request links
- Breadcrumb navigation throughout

### Request management — All Requests (`/requests`)
- Filterable list: status (All / Pending / In Progress / Approved / Rejected), request type, priority (Urgent only)
- Search by request ID or applicant name
- URL-driven filter state — links from the dashboard pre-apply filters (e.g. Review Pending opens with Pending selected)
- Sortable columns with reset to default
- Scope toggle for approvers: Assigned to Me / All Requests

### Relocate Account form (`/requests/new/relocate`)
- Four-step form: Reason → Applicant → Move Details → Confirm
- Inline validation with specific error messages on each step
- Applicant search by name or service number
- Confirmation checkbox gates the Submit button (disabled until ticked)
- Step progress indicator

### Request detail and approval (`/requests/[id]`)
- Three-tab layout: Details, Instructions, History
- Details tab: full read-only summary of the request
- Instructions tab: numbered approval checklist — Approve button disabled until all items are checked
- History tab: chronological audit trail of all events
- Approve and reject actions with confirmation dialogs
- Status updates and toast notifications on action

### Personnel directory (`/personnel`)
- Admin and Approver: full searchable directory with unit and role filters; expandable rows show each person's request history
- Standard User: own profile only, with request history

### Settings (`/settings`)
- Profile display (read-only)
- Appearance: five themes (Snow, Slate, Navy, High Contrast, Eucalyptus) with live switching and system preference follow
- Notification preferences (toggle per notification type, role-gated)
- Access and Role: displays current role capabilities

### Notifications
- Bell icon in top bar with unread count
- Role-relevant notifications derived from request history (new submissions for admin, assignments for approver, status changes for user)
- Clears on open; resets on role switch

---

## What is not implemented

| Feature | Notes |
|---|---|
| Real authentication | Login is a demo entry point — no credentials are validated |
| Data persistence | All state resets on page refresh — no database |
| New Account form | Request type exists in mock data but has no submission form |
| Application Access form | Request type exists in mock data but has no submission form |
| Employee Registration form | Request type exists in mock data but has no submission form |
| Email notifications | Notification preferences are UI only — no emails are sent |
| Search in top bar | Search input is present in the UI but is not wired to a results page |
| Pagination | The request table shows all records — no pagination for large datasets |
| File attachments | No document upload capability |
| Comments on requests | No freetext comment thread on request detail |
| User management | No interface for creating or modifying user accounts |
| Reporting or exports | No data export, no reporting dashboard |
| Audit log export | History timeline is display only — no downloadable audit trail |
| Real-time updates | No WebSocket or polling — page must be refreshed to see changes made in another session |

---

## Known prototype limitations

- **Form progress resets on navigation.** If a user navigates away from the multi-step form mid-way, their progress is lost. This is intentional for the prototype — a production system would persist draft state.
- **Approval actions are session-scoped.** Approving or rejecting a request updates the local UI state only. Refreshing the page resets the request to its original mock status.
- **Mock data is fixed.** Submitting a new request does not add it to the All Requests list. The eight mock requests are always present.
- **Three fixed personas.** Role switching always maps to the same three people. A production system would allow any authenticated user to log in.
- **No mobile optimisation pass.** The layout is responsive but has not been formally tested or refined for small screen sizes.

---

## Prototype personas

| Name | Rank | Role | Unit |
|---|---|---|---|
| Maj Sarah Chen | Major | Admin | HQJOC, Duntroon ACT |
| WO2 David Hartley | Warrant Officer Class 2 | Approver | 6th Brigade, Enoggera QLD |
| Capt Aisha Okafor | Captain | Approver | Defence Intelligence, Duntroon ACT |
| Cpl James Nguyen | Corporal | Standard User | 1st Signals Regiment, Enoggera QLD |
| Lt Priya Sharma | Lieutenant | Standard User | 2nd Cavalry Regiment, Darwin NT |
| Sgt Michael Torres | Sergeant | Standard User | 16th Air Land Regiment, Sydney NSW |

The role switcher cycles between the Admin, first Approver, and first Standard User personas.
