# Coaching Dashboard — Page & Functionality Spec

**Platform name:** Coaching Dashboard
**Prototype:** `index.html` (alpha-ui-mockup) — 33 screens across 4 roles
**Scope note:** This documents what each page is *responsible for* and the functionality each user needs. **Routing and other live-integration features are NOT being built into the real app yet** — they're approved in the mock first (per current decision). This spec is the review surface for that approval.

**Roles:** Coach · Guide · Admin/DRI (Coaching DR, Campus DR, Subject DR) · Student (Timeback-only, minimal surface).

---

## 0. Global shell (every authenticated page)

| Element | Responsibility | User needs |
|---|---|---|
| **Left sidebar** | Labeled, always-visible nav, grouped by section; role-specific item set; active item highlighted; badges for counts (e.g. Awaiting notes · 3). | Always know where they are and where everything lives. |
| **Breadcrumb** (topbar left) | `Section / Page` location indicator; home crumb is clickable. | Orientation; one-click back to role home. |
| **Search** (topbar) | Global lookup of students, coaches, sessions. | Jump straight to any entity. |
| **Notifications bell** | Unread feed: flags, overdue notes, new bookings, QC complete, guide notes. Mark-all-read. | Catch events without hunting. |
| **Up-next banner** | Surfaces the next session ~8 min out with a one-click "Open session". Dismissible. | Never miss the next call. |
| **User chip** (sidebar foot) | Identity + role; opens Settings. | Confirm who they're acting as; reach settings. |
| **View-as-coach** (admin only, topbar) | Admin renders any coach's surface read-only. | Spot-check a coach without logging in as them. |

---

## 1. COACH

The coach's day is a **queue of sessions**; the home is a worklist, the session detail is a forward-moving pipeline.

### 1.1 Dashboard `/coaching`
**Purpose:** Command center — what needs attention today, in order.
**Sections:** Up-next banner · welcome header (today's count, next time) · 4 stat cards (This week + sparkline, Awaiting notes, No-show rate, Students coached) · Today's sessions list · embedded week calendar.
**User can:** open any session · join the next call · jump to full schedule · jump to awaiting notes · see week-at-a-glance.
**Reads:** today's `CoachSession`s, week aggregates, awaiting-notes count.
**Links to:** session-detail, schedule, notes.

### 1.2 My schedule `/coaching/schedule`
**Purpose:** Full calendar of the coach's sessions.
**Sections:** week grid (8 AM–2 PM gutter), color-coded events by call type, "now" line, prev/today/next, Week·Day·List toggle.
**User can:** navigate weeks · switch view density · click an event → session detail.
**Reads:** `CoachSession`s for the visible range (from OpenCal bookings).
**Backend:** pulls bookings via OpenCal API.

### 1.3 Calls history `/coaching/history`
**Purpose:** Past sessions, searchable record of performance.
**Sections:** filter bar (subject, status, +date range) · session cards (date/time, student, tags, status).
**User can:** filter by subject/status/date · open any past session (incl. no-show/cancelled states).
**Reads:** completed/no-show/cancelled `CoachSession`s.

### 1.4 Awaiting notes `/coaching/awaiting`
**Purpose:** The documentation worklist — sessions that still need write-up.
**Sections:** urgency-sorted cards (overdue → yesterday → today) with overdue severity styling.
**User can:** open a card → lands **directly on the After tab** of that session (not Before).
**Reads:** sessions where `status=completed` but notes not submitted.

### 1.5 Performance `/coaching/performance`
**Purpose:** Personal trend review (replaces opening 5 PDF emails).
**Sections:** perf cards — sessions this month, QC pass rate (+bar), no-show rate, notes completion (+bar), subjects covered (tags), students coached — each with delta vs last month and red/amber/green banding.
**User can:** read own trends; no peer comparison (that's admin-only).
**Reads:** monthly aggregates of QC results, no-shows, notes completion.

### 1.6 Settings `/coaching/settings`
**Purpose:** Everything the coach self-manages (so they never ask engineering).
**Tabs & functionality:**
- **Profile** — name, email (read-only), campus, role (read-only), photo.
- **Availability** — per-day from/to + active toggle. (Syncs to OpenCal.)
- **Notifications** — per-event channel toggles (booking, 30-min reminder, 5-min reminder, notes overdue, flagged, QC complete, guide notes).
- **Call types** — table of types (Academic, Mastery, SAT, Proctoring, Literally Testing, Post-test) + durations + grades; edit.
- **Integrations** — connect Google Calendar / Microsoft / CalDAV; choose meeting location (Meet / Zoom / phone).
- **Booking page** — min notice period, booking window, allow cancel/reschedule toggles, preview booking page.

### 1.7 Session detail `/coaching/sessions/[id]` — **the pipeline (spine of the product)**
**Purpose:** Single home for everything about one session; moves Before → During → After → QC with a forward CTA at each step.
**Header:** student avatar/name/meta · call-history timeline squares (color = result, stripe = subject, hover = date) · prev/next session nav · Join call.
**Info bar:** title, type+status tags, meeting time, subject/app/level.
**Pipeline stepper:** Before · During · After · QC (QC hidden until recording analysed). Forward CTA bar advances: *Join call & start →* / *End call & document →* / *Mark complete →* / *Back to schedule →*.

**Before tab (3-column):**
- Col 1: doom-loop alert (3+ same blocker) · RIT/test-score alert (<90% flag) · AI pre-call summary · student snapshot + age-grade vs working-grade gap indicator · session focus + private prep notes.
- Col 2: campus DR note · guide context (this session + recent student notes) · delegation note ("why was this student assigned to me").
- Col 3: coaching history (month-grouped timeline + recent results list).
**User needs:** absorb full context fast, write prep notes, optionally message guide.

**During tab (2-column):**
- Col 1: live notes (autosaved, private) · quick actions (mark no-show / reschedule / cancel / message guide / flag) · message thread with guide.
- Col 2: session context (blocker, prep notes).
**User needs:** take notes, handle no-show/reschedule, talk to guide mid-call.

**After tab (2-column):**
- Col 1: **session objective** (prominent) · "Document this session" (recording URL, skill/lesson) · session notes (what happened) + recommendations · doc-status badge · **Save draft** vs **Mark complete & send**.
- Col 2: send-to checkboxes (Guide / Subject DRI / Campus DRI) · QC status.
- On **Mark complete** → flips to a **read-only completed state**: green confirmation, sent-to checklist, locked notes, QC-pending card.
**User needs:** document, choose recipients, complete → dispatches notes + fires QC.

**QC tab (2-column):**
- Col 1: PASS/FAIL verdict · 3 mandatory criteria (Student Progress, Coach Effectiveness, Interaction Dynamics) with rationale · dimension score bars (Progress, Effectiveness, Interaction, Engagement, Charisma, Age-appropriate language).
- Col 2: clip suggestions (thumbnails + timestamps) · AI analysis notes · **Dispute result** / **Share with guide**.
**User needs:** review the grade, dispute if wrong, pull teaching clips.

### 1.8 Session — Messages `/coaching/sessions/[id]/messages`
**Purpose:** Full coach↔guide thread for a session.
**User can:** read history, reply.

### 1.9 Session — No-show state
**Purpose:** Post no-show view.
**Sections:** no-show banner (buffer removed, coach freed early) · session details · optional note · actions (rebook, notify guide, undo no-show) · buffer-impact explainer.

### 1.10 Session — Cancelled state
**Purpose:** Post-cancellation view.
**Sections:** cancelled banner (who/when) · details + reason · actions (rebook, notify guide).

---

## 2. GUIDE

The guide owns the **student relationship**: feed coaches good context, track outcomes, escalate.

### 2.1 Guide dashboard `/coaching/guide`
**Purpose:** Which of my students are coached today + my roster.
**Sections:** welcome · students-with-sessions-today list (student + coach + time) · your-students table.
**User can:** open a student's session prep · add a note · open student profile.

### 2.2 Student profile (deep-dive) `/coaching/guide/students/[id]`
**Purpose:** The guide's working surface for one student.
**Sections:** header + timeline · student snapshot (interests, learning style, app, working grade) · **add guide context** (shared to next coach) · coach feedback received · **flag this student** (urgent / recurring blocker / request subject DRI review + context) · **book a session**.
**User needs:** drop prep context that lands in the coach's Before tab; flag escalations; book on behalf of student.

### 2.3 Book for student `/coaching/guide/book`
**Purpose:** Guide-initiated booking.
**Sections:** session form (call type, subject, date, preferred time, note for coach) · available coaches (with slots left) · recent sessions.
**Backend:** routes through OpenCal (same pipe as student booking).

### 2.4 Booking confirmed
Confirmation banner + return links.

---

## 3. ADMIN / DRI

Coaching DR (Himanshi) = full access + own coaching load. Campus DR = campus-scoped. Subject DR = mostly email/Slack-driven + notes report.

### 3.1 Overview `/coaching/admin`
**Purpose:** System health + triage queue.
**Sections:** stat cards (sessions today, active now, flags, notes overdue) · today's-coaches table (load, no-show, notes pending → View → coach detail) · active flags preview.
**User can:** scan health · drill into a coach · jump to flags.

### 3.2 Students `/coaching/admin/students`
**Purpose:** Search/browse all students.
**Sections:** campus filter + search · students table (campus, grade, subject, total calls, last session, flags → View → profile).

### 3.3 Student profile (admin) `/coaching/admin/students/[id]`
**Purpose:** Full student record for admins.
**Sections:** header (flag badge + timeline) · stat cards (calls this week, QC pass rate, no-show) · session history · guide notes + add note · book for student.

### 3.4 Coaches `/coaching/admin/coaches`
**Purpose:** Roster overview.
**Sections:** campus filter + add coach · coaches table (campus, role, subjects, today, this week, QC pass, notes overdue → View → coach detail).

### 3.5 Coach detail `/coaching/admin/coaches/[id]`
**Purpose:** One coach's full picture.
**Sections:** header (attributes chips) · stats (today, week, QC pass, no-show, notes overdue) · today's sessions · overdue notes · availability summary · week load chart · QC summary (+ view QC recordings) · edit attributes / remove access.

### 3.6 Student flags `/coaching/admin/flags`
**Purpose:** Triage auto-detected booking-pattern flags.
**Sections:** filter tabs (same-subject / high-frequency / all) · flag cards (double-flag callout, meta, actions: view history, notify DRI(s), dismiss).
**Logic:** 5 same-week → coaching+campus DR; 5 same-subject → double flag (coaching+campus+subject DR). Slack ping per campus channel.

### 3.7 Session notes report `/coaching/admin/notes` (Subject DRI view)
**Purpose:** All session notes for a subject, hunting systemic blockers.
**Sections:** filters (subject, coach, date range) · note cards (notes + skill covered) · awaiting-notes state.

### 3.8 Weekly digest `/coaching/admin/digest`
**Purpose:** Configure + preview the Monday digest.
**Sections:** week stats · top subjects bars · recipients checklist · preview digest.

### 3.9 Coach attributes `/coaching/admin/attributes`
**Purpose:** The matrix that *drives routing* (single source of truth).
**Sections:** table (coach × grade bands × subjects × call types × daily cap × in-window cap) · "routing auto-syncs" note · edit.
**Note:** changing the matrix is how routing changes — no separate routing-form rules.

### 3.10 Routing config `/coaching/admin/routing` *(mock-approval gated)*
**Purpose:** Buffer/cap/priority rules.
**Sections:** how-routing-works · buffer rules (5-min between, 30-min from prev end, remove-buffer-on-no-show toggle) · priority rotation explainer.

### 3.11 Routing simulator `/coaching/admin/routing/simulator` *(mock-approval gated)*
**Purpose:** Test routing before go-live; backtest capacity.
**Sections:** simulate-a-booking form (grade/subject/call-type/time) → result · today's live-load table · priority order. **Planned tab:** "Backtest against history" (the capacity simulator — Calendly-served vs our-routing vs ceiling).

### 3.12 Workflows `/coaching/admin/workflows`
**Purpose:** Configurable automated emails/actions.
**Sections:** active workflows list (booking confirmation, 30-min reminder, post-session feedback, cancellation) · new workflow.

### 3.13 Workflow edit `/coaching/admin/workflows/[id]`
**Purpose:** Edit one workflow.
**Sections:** trigger (when / how-far-before) · send-to · email template (subject + body + variables) · send test · save.

### 3.14 Blocked dates `/coaching/admin/blocked-dates`
**Purpose:** Holidays + custom blocked days across coaches.
**Sections:** import public holidays (country/year) · add custom blocked date (scope: all/campus) · upcoming blocked-dates table.

### 3.15 Slack alerts `/coaching/admin/slack`
**Purpose:** Per-campus webhook config.
**Sections:** setup instructions · per-campus webhook table (channel, status, alerts, configure). Campus DRs set their own.

### 3.16 Add coach `/coaching/admin/coaches/new`
**Purpose:** Onboard a coach in one flow.
**Sections:** basic info (name, email, campus, role) · attributes (grade bands, subjects, call types) · caps (daily + in-window) · send Google invite. Role defaults caps (HOA 1/day, Campus DR 7/day).

---

## 4. STUDENT (minimal — students live in Timeback)

> Students never use the main app for real; this surface exists for completeness/edge cases. Their real touchpoints are Timeback (booking auto-fill, feedback) + the AI Coach Chrome extension.

### 4.1 My sessions `/coaching/student`
Hi banner · next session card (join) · book-a-session card · my sessions list with QC results.

### 4.2 Book a call `/coaching/student/book`
3-step stepper: call type + subject → slot picker (matched coaches) → confirm summary.

### 4.3 Booking confirmed
Confirmation · what-happens-next · add-to-calendar · reschedule/cancel.

---

## 5. Data entities referenced (storage-level)

| Entity | Key fields | Notes |
|---|---|---|
| **CoachSession** | bookingId, organizerId, studentEmail, subject, callType, status, prepNotes, sessionNotes, recordingUrl, skill, sessionObjective, completedAt | Created from OpenCal `booking.created` webhook. |
| **StudentFlag** | flagId, studentEmail, campusId, flagType (same-subject/high-frequency), triggerCount, subject, weekOf, status, notifiedRoles | Auto-detected; drives Flags page + Slack. |
| **Timeback Feedback** | id, appointmentId, studentEmail, recordingId, rating (👍/👎), comment, submittedAt, source (timeback/direct/sheet_bridge), tenantId | Per PRD 08. Surfaced as the "Timeback Feedback" view (source=timeback). Powers a feedback column in history + a guide/admin distribution. |
| **CoachAttributes** | coachId, gradeBands[], subjects[], callTypes[], dailyCap, inWindowCap, role | The routing matrix. |
| **QC result** | per recording: PASS/FAIL, 3 criteria, dimension scores, clips | From Social Toolkit. |

> **Timeback Feedback** is a column/view that should appear in: coach Calls history, coach Performance, guide Student profile, and admin distribution — once the entity lands. Not yet built.

---

## 6. Role → page access matrix

| Page group | Coach | Guide | Coaching DR | Campus DR | Subject DR | Student |
|---|---|---|---|---|---|---|
| Own dashboard | ✅ | ✅ | ✅ | ✅ | ✅ | ✅(min) |
| Session pipeline | ✅ | view prep | ✅ | ✅(campus) | view | — |
| Schedule/history | ✅ own | — | ✅ all | ✅ campus | — | own |
| Student profiles | — | ✅ assigned | ✅ all | ✅ campus | ✅ subject | — |
| Flags | — | raise | ✅ | ✅ campus | ✅ subject | — |
| Coaches/attributes | — | — | ✅ | view | — | — |
| Routing/workflows/blocked dates/slack | — | — | ✅ | partial | — | — |
| Notes report | — | — | ✅ | ✅ | ✅ | — |
| Settings | ✅ own | ✅ own | ✅ own | ✅ own | ✅ own | — |

---

## 7. Deferred / pending (don't build into the real app yet)

- **Routing config + simulator** — mock-approval first, then integrate.
- **Capacity simulator (backtest tab)** — designed (`CAPACITY-SIMULATOR.md`), build after spec sign-off.
- **Timeback Feedback ingestion** — entity + `POST /api/feedback` + sheet bridge per PRD 08; show row structure first, then build.
- **DNS** — `coaching.alpha.school` via Amplify + Add-New-DNS-Record (additive CNAME into existing zone).
- **Live OpenCal integration** — booking webhook → CoachSession; attribute matrix → routing.

---

## 8. Open questions for review

1. Confirm route prefix (`/coaching/*`) and that QC lives **inside** session detail (tab) rather than a separate nav item.
2. Confirm "Timeback Feedback" = a filtered view of one `Feedback` entity (source=timeback), surfaced as a column in history + a distribution panel.
3. Which pages are **v1 must-haves** vs later (e.g. routing simulator, notes report, workflows)?
4. Campus DR scoping rules — exactly which admin pages they see, campus-filtered.
5. Should the coach **Performance** page and the admin **Coach detail** share one component?
