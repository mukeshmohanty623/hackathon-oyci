## OYCI admin & staff specification (/admin)

This file focuses on internal admin and staff workflows, including event management, staffing, attendance, consents, integrations, and reporting.

---

### 1. Admin roles & access

- **SuperAdmin**
  - Manages global settings, integrations (Zoho, Xero), and all data.
- **Coordinator**
  - Manages events, staff assignments, and reports.
- **Staff**
  - Views “My events today”, records attendance, and adds quick registrations/drop-ins.
- **Finance/Payroll**
  - Reviews hours and runs payroll exports.
- **Reporting-only**
  - Can view but not change reports and data.

Role-based redirects after login:

- SuperAdmin/Coordinator → Admin dashboard.
- Staff → My events today.
- Finance/Payroll → Payroll & exports.
- Reporting-only → Reports home.

---

### 2. Dashboards & navigation

**Admin dashboard (SuperAdmin/Coordinator)**

- Summary cards:
  - Today’s sessions (count).
  - Young people attending today.
  - Under-capacity events (many spaces).
  - Understaffed events.
- Alerts:
  - Understaffed events.
  - Events with long waiting lists.
  - Expired consents for upcoming bookings.
  - Zoho sync errors.
- Today’s events table:
  - Time, event, location.
  - Booked count / capacity.
  - Staffed? (Yes/No).
  - Actions: View, Check-in.
- Quick links:
  - Create event.
  - View all events.
  - Run report.

**Staff dashboard (My events today)**

- For logged-in staff:
  - List of sessions they’re assigned to today.
  - For each:
    - Time, event name, location, number booked.
    - Button: Open check-in.

---

### 3. Event lifecycle (admin)

**Event list**

- Filters:
  - Date range.
  - Programme.
  - Location.
  - Status: Active / Archived.
- Table columns:
  - Event name.
  - Programme.
  - Upcoming sessions count.
  - Capacity (per session).
  - Booking rules (booking required / drop-in allowed).
  - Actions: Edit, View sessions, Duplicate.

**Create / edit event wizard**

- Step 1 – Basics:
  - Name.
  - Description (for guardians).
  - Programme (optional grouping).
  - Tags (holiday, after-school, one-off trip, etc.).
  - Age range (min / max).
  - Capacity per session.
  - Location (name, address, postcode).
- Step 2 – Schedule:
  - Select:
    - Single date/time.
    - Repeating pattern (e.g. weekly):
      - Start date, end date.
      - Days of week.
      - Time of day.
  - Preview list of `EventInstance`s that will be created.
- Step 3 – Booking rules:
  - Booking required? (yes/no).
  - Drop-in allowed? (yes/no).
  - Booking window:
    - Opens on [date/time].
    - Closes [hours/days] before start.
  - Max sessions per young person (for series).
- Step 4 – Review & publish:
  - Summary of all sessions to be created.
  - Option to:
    - Publish now.
    - Save as draft.

**Event instance detail**

- Header:
  - Event name.
  - Date/time.
  - Location.
- Tabs:
  - Overview:
    - Bookings vs capacity.
    - Staffing summary.
    - Internal notes for staff.
  - Participants:
    - List of bookings and attendance records.
  - Staff:
    - List of assignments with roles and times.
  - Reports:
    - Quick stats for this specific session (attendance, no-shows, drop-ins).

---

### 4. Staff management & Zoho People integration

**Staff list**

- Table:
  - Name.
  - Role.
  - Active/inactive.
  - Synced with Zoho? (yes/no indicator).
- Filters:
  - Role.
  - Active status.
- Actions:
  - View profile.
  - Deactivate / Reactivate.
  - Button: Sync from Zoho now (for authorised roles).

**Staff profile**

- Summary:
  - Name, contact details, role.
  - Skills.
  - Link to Zoho profile (based on `zoho_people_id`).
  - Last Zoho sync time.
- Schedule:
  - Calendar or list of upcoming assignments.
  - Links to event instances.
- Leave:
  - Read-only list of leave periods from Zoho (type, dates, status).
- Hours:
  - Total hours worked in recent periods (from `StaffAssignment`).
  - Links to payroll exports.

**Assign staff to event instance**

- Requirements:
  - Show required roles counts (Lead, Support, Volunteer).
- UI:
  - For each role slot:
    - Dropdown of available staff:
      - Only staff not on approved leave.
      - Avoid double-booking by showing conflicts.
    - Optional skills or preferences indicator.
- Save behaviour:
  - Creates/updates `StaffAssignment` records.

**Zoho leave impact**

- Scheduled sync:
  - Every 15–60 minutes pull staff and leave data from Zoho.
- For each approved leave period:
  - Find overlapping `StaffAssignment`s.
  - Mark them `unassigned_due_to_leave`.
  - Log an audit entry.
  - Notify SuperAdmin/Coordinators and show dashboard alerts.
- If leave is cancelled:
  - Do not auto-reassign.
  - Instead show suggestions for reassigning staff where appropriate.

**Error handling**

- If Zoho API fails:
  - Log error.
  - Show banner with last successful sync time.
  - Retry later with backoff.

---

### 5. Attendance & drop-ins (staff)

**Event check-in**

- Accessed from:
  - Admin dashboard.
  - Staff “My events today”.
- Layout:
  - Header with event name, date/time, location.
  - Staff roster for quick view.
- Sections:
  - Booked participants:
    - Search box (name/family).
    - List showing:
      - Child name and age.
      - Status: Booked / Arrived / No-show.
      - Action button: Check in / Mark no-show.
  - Drop-in participants:
    - Button: Add drop-in.
    - List of already added drop-ins with link to profile.
  - Summary:
    - Counts:
      - Booked.
      - Arrived.
      - Drop-ins.
      - No-shows.

**Drop-in registration**

- Step 1: search existing young person:
  - Search fields: name, school, family, postcode.
  - If found:
    - Create `AttendanceRecord` with:
      - `status = Arrived`.
      - `booking_id = null`.
      - `source = staff_checkin`.
- Step 2: if not found:
  - Minimal registration:
    - Child name.
    - Approximate age or DOB.
    - Emergency contact name & phone.
    - Optional school.
  - Create:
    - `YoungPerson` (and `Family` if needed).
    - Mark as `profile_incomplete` for follow-up.
    - `AttendanceRecord` as drop-in.

---

### 6. Participant & family admin views

**Global search**

- Search across:
  - Young people (by name, school, DOB).
  - Families (by guardian name, phone, postcode).

**Young person profile (admin)**

- Tabs:
  - Summary:
    - Name, DOB, school, family.
    - Consent records and expiry dates.
  - Participation:
    - List of attendance records and bookings.
    - Filters by date range, programme.
  - Notes:
    - Safeguarding notes with role-based permissions.
  - Admin:
    - Merge duplicates.
    - Mark inactive.

**Family profile (admin)**

- Shows:
  - Guardians and contact details.
  - Children with brief participation summaries.
  - Household-level notes.
  - Pattern summary:
    - “Children from this family have attended X sessions across Y programmes this year.”

---

### 7. Consents management (admin)

**Consent dashboard**

- Filters:
  - Expiring in X days.
  - Expired.
  - All.
- Table:
  - Child name.
  - Family.
  - Consent type.
  - Valid until.
  - Status (valid/soon to expire/expired).
- Bulk actions:
  - Send SMS/email reminders.
  - Export list for letter generation.

**Staff-assisted consent renewal**

- Flow:
  - Staff opens family at reception.
  - Selects child and consent type.
  - Shows current text and explains in person.
  - Records:
    - Guardian name.
    - Method (in-person, phone, paper).
    - Date.
  - Saves new `ConsentRecord`.

---

### 8. Integrations (admin view)

**Zoho People**

- Settings page:
  - API credentials and connection status.
  - Last sync time.
  - Manual `Sync now` button.
  - Logs of recent sync runs and any errors.

**Xero**

- Settings page:
  - API credentials (if using direct API).
  - Mapping between staff and Xero employees or pay items.
  - Confirmed payroll export formats.

---

### 9. Payroll & Xero exports

**Payroll overview (Finance/Payroll role)**

- Select:
  - Pay period (start/end dates).
  - Staff to include (default: all active).
- System calculates:
  - Total hours per staff from `StaffAssignment` (actual or planned times).
- Review table:
  - Staff name.
  - Total hours.
  - Number of sessions.
  - Status:
    - Pending.
    - Exported.
    - Failed.
- Actions:
  - Adjust hours or exclude staff.
  - Export as:
    - CSV for Xero.
    - Direct API call to Xero (if configured).

**After export**

- Create/update `PayrollExportRecord`:
  - Mark as `exported` with timestamp and batch ID.
  - Prevent accidental double export (warn if trying to re-export same period/staff).

**Error handling**

- CSV:
  - Show message and allow regeneration if file creation fails.
- API:
  - For each staff row:
    - If Xero returns an error:
      - Mark that row as `failed`.
      - Store error message.
  - Show summary: “X of Y staff exports failed – view details.”

---

### 10. Reporting (admin)

**Reports home**

- Categories:
  - Participation.
  - Staff & hours.
  - Equality & inclusion (optional).
  - Consent & safeguarding.
- Each report tile:
  - Title, short description.
  - `View report` button.

**Key report types (brief)**

- Participation summary:
  - Total bookings, unique young people, attendances, average sessions per young person.
- Participation by programme/event:
  - Compare engagement across programmes.
- Individual participation frequency:
  - Sessions attended and no-shows per young person.
- Engagement by school/area/age:
  - Understand reach into different communities.
- Staffing load:
  - Hours and sessions per staff, by role.
- Drop-in vs pre-booked:
  - Ratio of booked vs drop-in attendances.
- Consent status overview:
  - Counts of valid, expiring, and expired confidentiality consents.

All reports should support CSV/Excel export and include filter criteria and generation timestamp.

