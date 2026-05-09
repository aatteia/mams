# MAMS — Interview and Demo Guide

This document describes the MAMS system, who it is for, and how to use it. It is intended to support interviews, stakeholder walkthroughs, and presentations.

---

## What is MAMS?

MAMS — the Military Account Management System — is a Human Resources Information System designed for the Australian Defence Force. It manages the lifecycle of network account requests that arise from personnel movements and administrative events: transfers between units, new enlistments, application access grants, and account deregistrations.

In the Defence environment, when a service member transfers from one unit to another, their network account, system access, and identity records must be formally moved and validated. This is not a trivial operation — it involves confirming clearance levels, verifying network segment availability at the destination, checking for outstanding service desk tickets, and notifying multiple administrators. MAMS structures and tracks this process, replacing ad-hoc email chains and paper-based forms.

---

## The problem MAMS solves

Without a purpose-built system, account movements in Defence are typically managed through a mix of email, shared spreadsheets, and manual checklists. This creates several risks:

- **Lost requests.** An email chain is not a system of record. Requests can be missed, forgotten, or actioned by the wrong person.
- **Incomplete approvals.** Without a structured checklist, approvers may overlook required verification steps before granting access.
- **No audit trail.** Spreadsheets do not capture who approved what, when, and on what basis. This is a compliance and security risk.
- **No visibility.** Submitters have no way to know where their request is in the approval pipeline.

MAMS addresses each of these directly: every request is logged, assigned, tracked through a structured approval workflow, and given a complete audit history.

---

## Who uses MAMS?

MAMS is designed around three distinct roles, each with a different relationship to the request lifecycle.

### Standard User
A service member who submits requests for their own account or on behalf of their unit. They see only their own submissions, can track the current status of each request, and receive notifications when requests are approved or rejected.

**Example persona:** Corporal James Nguyen, 1st Signals Regiment. James has received a posting order transferring him to 6th Brigade. He submits a Relocate Account request, specifying his destination unit, effective date, and destination supervisor. He can track the request from his dashboard until it is approved.

### Approver
A senior NCO or officer who reviews requests assigned to them. They do not submit requests — their role is to work through a structured approval checklist for each assigned request, verify that all conditions are met, and record their decision. They also have view access to the full personnel directory.

**Example persona:** Warrant Officer Class 2 David Hartley, 6th Brigade. David receives a notification when a new request is assigned to him. He opens the request, reads the relocation details, and works through five checklist items — confirming the destination network segment, checking the applicant's clearance level, verifying no outstanding tickets, checking the effective date against the Defence calendar, and notifying the destination IT administrator. Only when all five are complete can he approve.

### Administrator
A staff officer with full system access. Administrators can submit requests on behalf of any personnel, view and manage the entire request queue, access the full personnel directory, and receive system-wide notifications. In a deployed system, administrators would also manage user accounts and system configuration.

**Example persona:** Major Sarah Chen, HQJOC. Sarah manages the system across multiple units. She submits a request on behalf of FLGOFF Papadopoulos ahead of her posting to Air Force Command HQ, monitors the queue for stalled requests, and reviews system-wide activity.

---

## How to navigate the prototype

The prototype is accessible at: **https://proof-of-concept-defence-hris.vercel.app**

### Starting the demo

1. Open the link. You will land on the login page.
2. Click **Continue as Administrator** to enter as Maj Sarah Chen (Admin role).
3. You can switch roles at any time using the dropdown in the top-right corner of the screen — click the name and role chip.

### Switching roles

The role switcher is available from every screen. Switch to:
- **Admin** — Maj Sarah Chen — to see the full system view
- **Approver** — WO2 David Hartley — to see the approval queue and assigned requests
- **Standard User** — Cpl James Nguyen — to see a personal view with restricted access

Notice how the dashboard statistics, quick actions, navigation items, and available request tables change with each role. The Personnel page also changes — Admin and Approver see the full directory; Standard User sees only their own profile.

---

## Walkthrough 1 — Submitting a relocation request (Admin role)

This journey demonstrates the request submission workflow.

1. From the dashboard as Admin, click **Relocate Account** in the Quick Actions section.
2. **Step 1 — Reason:** Select a reason from the dropdown (e.g. "Posting — permanent unit transfer") and add optional notes. Click **Continue**.
3. **Step 2 — Applicant:** Search for a service member by name or service number. Select one from the results. Click **Continue**.
4. **Step 3 — Move Details:** Select the origin unit and location, destination unit and location, effective date, and destination supervisor name. Click **Continue**.
5. **Step 4 — Confirm:** Review the full summary. Tick the accuracy confirmation checkbox — this enables the **Submit request** button. Click **Submit request**.
6. A confirmation toast appears. The request is now in the system.

**What to highlight:** The four-step structure ensures all required information is collected before submission. Inline validation on each step prevents incomplete forms from progressing. The gated submit button (disabled until the checkbox is ticked) mirrors the same pattern used in the approval workflow — a deliberate design consistency.

---

## Walkthrough 2 — Reviewing and approving a request (Approver role)

This journey demonstrates the approval workflow.

1. Switch role to **Approver** (WO2 David Hartley).
2. The dashboard now shows requests assigned to David. Click **Review Pending** in the Quick Actions section — the All Requests page opens with the Pending filter pre-applied.
3. Click on any pending request ID (e.g. REQ-2024-0841) to open the detail view.
4. Review the **Details** tab — the full request summary is displayed.
5. Navigate to the **Instructions** tab. A progress bar and a numbered checklist of five items are shown. The Approve button is disabled and the caption reads "Complete all 5 items to enable approval."
6. Work through the checklist — tick each item. The progress bar advances with each tick.
7. When all five items are checked, the Approve button becomes active and the caption changes to "All items complete — ready to approve."
8. Click **Approve**. A confirmation dialog appears. Click **Confirm approval**.
9. The status badge changes to Approved, a success toast fires, and the buttons are disabled (the request is actioned).
10. Navigate to the **History** tab. The approval event has been added to the timeline, showing the approver's name and a timestamp.

**What to highlight:** The approval checklist is a structured replacement for informal verification. The gate (all items must be checked before approval is possible) is a compliance control, not merely a UI preference. The history timeline provides a complete, tamper-evident record of every action taken on the request.

---

## Walkthrough 3 — The notification bell

1. From any role, notice the bell icon in the top bar. A blue dot indicates unread notifications.
2. Click the bell. A dropdown panel appears with notifications relevant to the active role:
   - As **Admin**: new request submissions across the system
   - As **Approver**: requests assigned to them for review
   - As **User**: status changes on their own requests
3. Each notification includes a description, a clickable request ID link, and a relative timestamp.
4. Once the panel is opened, the blue dot clears. Switch roles — the dot reappears, reflecting a fresh set of notifications for the new user.

---

## Walkthrough 4 — Appearance settings

1. Navigate to **Settings** from the bottom of the sidebar.
2. Under the **Appearance** section, try switching themes: Snow (default), Slate, Navy, High Contrast, Eucalyptus.
3. The entire interface updates immediately — sidebar colour, background, primary colour, and all component tokens change.
4. Enable **Follow system** to make the theme track the OS light/dark preference automatically.

This feature demonstrates attention to accessibility and user preference — High Contrast in particular is designed for users with visual impairments or bright outdoor environments.

---

## Key design principles

**Role-aware throughout.** Every screen adapts to the active user's role. Navigation items, dashboard statistics, personnel access, notification content, and available actions all change based on who is logged in. The system never shows a user something they are not permitted to see.

**Structured workflows over free-form process.** The four-step form and the five-item approval checklist replace unstructured email communication with a defined, auditable process. Neither step can be skipped — the system enforces correct procedure.

**Complete audit trail.** Every request carries a history timeline showing every event: who submitted it, when it was assigned, who reviewed it, and what decision was made. This is essential in a Defence context where accountability and traceability are regulatory requirements.

**Consistent interaction patterns.** Gated primary actions (Approve, Submit) are always disabled until their condition is met, with a caption explaining what is required. Filters pre-apply when navigating from context-specific links (Review Pending pre-filters to Pending status). Breadcrumbs are always navigable. These patterns reduce cognitive load for users who return to the system infrequently.

---

## What a production system would add

This prototype demonstrates the user interface and workflow. A production implementation would require:

- **Real authentication** — Defence PKI certificate-based login or SAML-integrated single sign-on against an existing identity provider
- **Database** — PostgreSQL or a Defence-approved data store; the TypeScript types defined in this prototype map directly to the required schema
- **Server-side API** — REST or GraphQL API for request CRUD operations, assignment logic, and notification dispatch
- **Email and notification integration** — integration with Defence messaging infrastructure for approval assignment and status change notifications
- **Role management** — administrator interface for assigning roles, managing user accounts, and configuring approval routing rules
- **Remaining request types** — New Account, Application Access, and Employee Registration forms, each with their own approval checklists
- **Security controls** — field-level access control, data classification handling, audit log immutability, and penetration testing against Defence security frameworks
- **Accessibility audit** — formal WCAG 2.1 AA compliance testing, including screen reader testing and keyboard-only navigation validation
- **Mobile optimisation** — formal testing and refinement at small screen sizes for use on tablets in the field
