## OYCI user-side specification (/users)

This file focuses only on the public/guardian/youth side of the system. Admin, staff, integrations, and reporting are documented separately.

---

### 1. User goals & constraints

- **Book activities for children** quickly, on mobile, without needing to understand complex systems.
- **See and manage family participation**: who is booked into what, and when.
- **Handle consents and confidentiality** in plain language without long forms.
- **Allow for low digital confidence**:
  - Mobile-first, very simple language.
  - Minimal required fields.
  - Clear routes for staff-assisted bookings.
- **Support spontaneous participation**:
  - Users should feel they can turn up and still be welcomed, not blocked.

---

### 2. Key user personas

- **Guardian with smartphone & basic digital skills**
  - Can register, log in, and manage bookings themselves.
  - Prefers SMS reminders.

- **Guardian with low digital confidence**
  - Might only partially complete registration.
  - Often phones OYCI or turns up in person.
  - Needs clear messages that staff can handle bookings on their behalf.

- **Young person attending independently (no direct login)**
  - Does **not** create their own account.
  - Arrives in person; staff create a minimal profile and later link it to a guardian account when possible.
  - All online access and bookings are owned by a guardian account.

---

### 3. High-level user journeys

1. **New guardian registers, adds children, and books an activity.**
2. **Existing guardian logs in, views upcoming sessions, and changes a booking.**
3. **Guardian receives consent renewal prompt and completes it.**
4. **Family or young person turns up without booking; staff handle drop-in but future bookings are visible to guardians.**

---

### 4. Information architecture (user-side)

- **Top-level sections (when signed in)**:
  - `Home` (Dashboard)
  - `Activities` (Events list)
  - `Family` (Guardians & children)
  - `Account` (Profile & settings)

---

### 5. Screens & flows

#### 5.1 Entry, sign-in, and sign-up

**Screen: Welcome / splash**

- Shows:
  - OYCI logo and line of text: “Book activities and track your family’s participation.”
- Actions:
  - `Sign in`
  - `Create an account`
- Support message:
  - “If you cannot use this website, call us and we will book for you.”

**Screen: Sign in**

- Fields:
  - `Email`
  - `Password`
- Secondary flows:
  - `Forgot password?`
- Error states:
  - Invalid details message in plain language: “We couldn’t find that login. Please check your details or call us if you need help.”

**Screen: Create guardian account – step 1 (Guardian details)**

- Required fields:
  - `First name`
  - `Last name`
  - `Postcode`
  - `Email address`
  - `Password`
  - `Confirm password`
- UX details:
  - Keep the form on a single scroll on mobile.
  - Use large, clear error messages near each field.

**Screen: Create guardian account – step 2 (First child)**

- Fields:
  - `Child’s first name` (required)
  - `Child’s last name` (optional if same as guardian)
  - `Preferred name` (optional)
  - `Date of birth` (required; simple picker)
  - `School` (searchable dropdown with “Not in school / Other”)
  - `Year group` (optional dropdown)
  - `Any access or support needs?` (short text, optional)
- Actions:
  - Primary: `Save and continue`
  - Secondary link: `Skip this for now – staff can help later` (but warn that some activities may not be bookable without a child profile).

---

#### 5.2 Confidentiality & consents

**Screen: Confidentiality & consent**

- Content:
  - Short paragraph in plain language:
    - “We want to keep you and your information safe. Please read and agree so we can support your family.”
  - “Show full details” accordion with full legal text for those who want it.
- Controls:
  - Checkbox: “I understand and agree to the confidentiality agreement.”
  - Additional optional checkboxes:
    - “I am happy for photos of my child to be used in OYCI materials.”
    - “I agree to receive messages about new activities.”
  - Auto-filled fields:
    - `Your name` (guardian name, editable)
    - `Today’s date`
- Behaviour:
  - User cannot proceed to bookings without agreeing to core confidentiality.
  - Choices are recorded per child via `ConsentRecord`.

**Edge cases**

- If consent expires later:
  - Guardians are prompted with a similar screen but also see:
    - “Here is what has changed since you last agreed” with bullet-point highlights.

---

#### 5.3 Home / dashboard

**Screen: Home (Guardian dashboard)**

- Sections:
  - Greeting: “Welcome, [Guardian first name]”.
  - **Family summary**:
    - For each child:
      - Name, age.
      - Consent status:
        - “Consent valid until [date]” (green).
        - “Needs renewal” (amber).
      - Shortcut link: `View child`.
    - Button: `Add child`.
  - **Upcoming activities**:
    - List of upcoming sessions, grouped by date:
      - Time, event name, which child(ren), location.
      - Status badge: Booked, Waiting list, Cancelled.
      - Actions: `View details`, `Change / cancel`.
  - **Important banner (if needed)**:
    - Example: “We need to renew confidentiality for [Child]. This will take about 1 minute.” → `Renew now`.

---

#### 5.4 Family management

**Screen: Family overview**

- Shows:
  - Guardians in the family:
    - Name, contact info, which one is primary.
  - Children:
    - List with:
      - Name, age, school, consent status.
      - Links: `View details`, `Edit`, `Remove` (if allowed).
- Actions:
  - `Add child`
  - `Invite another parent/carer` (optional advanced feature).

**Screen: Add / edit child**

- Fields (add):
  - `First name` (required)
  - `Last name` (optional)
  - `Preferred name` (optional)
  - `Date of birth` (required)
  - `School` (optional)
  - `Year group` (optional)
  - `Access / support needs` (optional)
  - `Dietary requirements` (optional)
- Behaviour:
  - On save:
    - If confidentiality consent missing or expired:
      - Redirect to consent screen for this child.
    - Otherwise:
      - Return to family overview.

---

#### 5.5 Activities (events) discovery

**Screen: Activities list**

- Filters (at top or in slide-out panel):
  - `Date`:
    - “This week”, “This month”, “Choose dates…”.
  - `Type`:
    - Checkboxes (Holiday, After-school, One-off trip, etc.).
  - `Location`:
    - Postcode/area dropdown or “All locations”.
  - `Booking type`:
    - All / Booking required / Drop-in allowed.
- Event cards:
  - Title.
  - Short description (one line).
  - Dates:
    - Single: “Tue 15 June, 4–6pm”.
    - Series: “Every Tue for 6 weeks (from 15 June)”.
  - Badges:
    - `Booking required` / `Drop-in welcome`.
    - `Spaces left: X` / `Full – Waiting list`.
  - Primary action: `View details`.

**Empty & error states**

- If no events match filters:
  - Text: “We couldn’t find any activities that match. Try changing the filters or call us and we can help.”

---

#### 5.6 Activity details & booking

**Screen: Activity details**

- Header:
  - Event name.
  - Badge for type (e.g. Holiday programme).
- Sections:
  - About:
    - Longer description in simple language.
  - When & where:
    - List of upcoming dates with times.
    - Location name and address.
  - Who in your family can attend:
    - Clear text that all ages of young people supported by OYCI are welcome, unless the event is explicitly restricted for another reason (e.g. school group only).
  - Access & support:
    - Bullet points (e.g. wheelchair access, snacks provided).
- Actions:
  - `Book for my child(ren)` (if booking allowed and spaces available).
  - `Join waiting list` (if full and waiting list enabled).
  - Message if drop-in allowed:
    - “You can also just turn up and give your name at the door.”

**Flow: Start booking**

- If guardian presses `Book for my child(ren)`:
  - System checks:
    - If guardian has at least one eligible child with valid confidentiality consent.
  - If no eligible child:
    - Show gentle message:
      - “We need some details first before we can book this activity.”
      - Buttons:
        - `Add / update child details`.
        - `Call us for help`.

---

#### 5.7 Booking wizard

**Step 1: Select child(ren)**

- Shows cards (checkboxes) for each eligible child:
  - Name, age, any flag (e.g. “Has access needs note”).
  - Optional per-child note input:
    - Placeholder: “Anything staff should know for this activity?”
- Validation:
  - At least one child must be selected.

**Step 2: Select dates (for multi-session events)**

- Options:
  - Radio buttons:
    - `Book all dates in this block`.
    - `Choose specific dates`.
- If choose specific dates:
  - List of all `EventInstance`s in the series with:
    - Date/time.
    - Spaces left.
    - Checkbox per date.
- Validation:
  - At least one date must be selected.

**Step 3: Review & confirm**

- Summarise:
  - Event name.
  - Selected dates.
  - Child(ren) attending.
  - Any notes to staff.
  - Reminder text:
    - “When you arrive, give your name at reception. If you cannot attend, please cancel your place so someone else can come.”
- Buttons:
  - Primary: `Confirm booking`.
  - Secondary: `Back`.

**Step 4: Confirmation**

- Show:
  - “You’re booked” message.
  - For each date:
    - Date/time, location, which child(ren).
  - Reminders:
    - “We will send a reminder before each session.”
  - Actions:
    - `Add to calendar`.
    - `View all upcoming activities`.

**Notifications**

- Email and/or SMS:
  - Plain summary of bookings plus simple cancellation info.

---

#### 5.8 Managing bookings

**Screen: Upcoming activities**

- Tabs:
  - `Upcoming`
  - `Past`
- For each upcoming session:
  - Date/time.
  - Event name.
  - Children attending.
  - Status: Booked / Waiting list / Cancelled.
  - Actions:
    - `View details`.
    - `Change / cancel`.

**Change booking**

- Options:
  - Remove a child from one or more dates.
  - Cancel all remaining dates for that child.
- Rules:
  - Show any cut-off, e.g. “Please cancel at least 24 hours before if possible.”

**Cancel booking**

- Confirmation modal:
  - “Are you sure you want to cancel this booking?”
  - Explain impact: “We will free this place for another young person.”

---

#### 5.9 Consent renewal (user-side)

**Trigger**

- When a user logs in and:
  - A child’s confidentiality consent is expired, or
  - Will expire within a configurable window (e.g. 30 days).

**Banner behaviour**

- At top of dashboard:
  - “We need to renew confidentiality for [Child Name]. This will take about 1 minute.”
  - Button: `Renew now`.
- System behaviour:
  - Allows viewing existing bookings.
  - Blocks new bookings for that child until renewal done.

**Renewal screen**

- Shows:
  - Current confidentiality text.
  - Section: “What has changed since last time” with bullets (if version differs).
- Controls:
  - Checkbox: “I understand and agree to this confidentiality agreement.”
  - Name and date (pre-filled).
- After submit:
  - New `ConsentRecord` created.
  - Banner disappears.

---

#### 5.10 Spontaneous / drop-in participation

- In activities list and details:
  - For `drop_in_allowed` events:
    - Label: “Drop-in welcome – booking optional”.
    - Text:
      - “You can just turn up. Booking helps us plan, but you won’t be turned away because you didn’t book.”
- For guardians:
  - Messaging should reassure that they can always come and talk to staff even without using the system.

---

### 6. Accessibility & usability principles (user-side)

- **Plain language**:
  - Avoid jargon (“programme”, “session”) unless explained.
  - Write labels in child/guardian-friendly terms.
- **Short forms**:
  - Split longer processes into 2–3 short steps.
  - Only ask for extra information when truly needed.
- **Mobile-first**:
  - All designs optimised for small screens.
  - Large touch targets and font sizes.
- **Staff-assisted paths everywhere**:
  - At every key point (register, add child, book, renew consent), give a clear “Call us / come to reception” option so people are not blocked by the technology.
