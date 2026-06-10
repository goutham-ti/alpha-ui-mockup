# Alpha Coaching Platform — User Flows

One platform, four roles. OpenCal is the scheduling backend (invisible to users); everything below happens inside the coaching dashboard. Students never log in — they book from Timeback and meet coaches in Google Meet.

---

## 1. Information Architecture (sitemap)

What each role sees in the sidebar. The sidebar is **labeled and always visible** — no hidden icons.

```mermaid
flowchart TD
    Login[Google Login] --> Role{Role?}

    Role -->|Coach| CH[Coach Home]
    Role -->|Guide| GH[Guide Home]
    Role -->|Coaching/Campus/Subject DR| AH[Admin Home]
    Role -->|Student| TB[Timeback — not our app]

    CH --- C1[Today's worklist]
    CH --- C2[Schedule]
    CH --- C3[Calls history]
    CH --- C4[Awaiting notes]
    CH --- C5[Performance]
    CH --- C6[Settings]

    GH --- G1[My students today]
    GH --- G2[Student profiles]
    GH --- G3[Session history]

    AH --- A1[Overview]
    AH --- AP[People: coaches · students · flags]
    AH --- AA[Analytics: notes report · digest · performance]
    AH --- AC[Config: attributes · routing · workflows · blocked dates · Slack]
```

---

## 2. The Coach Session Loop (the core — 90% of a coach's time)

This is a **pipeline**, not a set of tabs. Each session moves Before → During → After → QC with a clear forward action at every step.

```mermaid
flowchart LR
    Dash[Dashboard worklist] -->|next-call banner T-8min| Before
    Before[Before call\nprep] -->|Join call & start| During
    During[During call\nlive notes] -->|End & document| After
    After[After call\ndocument] -->|Mark complete| Done{Complete}
    Done -->|QC runs ~2min| QC[QC review\nPASS/FAIL]
    Done -->|5-min buffer| Dash

    During -.student no-show.-> NoShow[Mark no-show\nbuffer removed]
    NoShow --> Dash
    QC -.disagree.-> Dispute[Dispute → Coaching DR]
```

**Step detail:**

| Phase | Coach does | System does | Forward action |
|---|---|---|---|
| **Before** | Reads AI summary, alerts (doom-loop, RIT, grade gap), guide + campus-DR notes, history. Writes prep notes. | Pulls student snapshot from Timeback, assembles context. | **Join call & start →** |
| **During** | Takes live notes. Can message guide. | Holds quick actions (no-show / reschedule / cancel / flag). | **End call & document →** |
| **After** | Session objective, recording URL, skill, notes, recommendations. Picks recipients. | — | **Mark complete →** |
| **QC** | Reviews PASS/FAIL + criteria + clips. | Social Toolkit analyses recording. | Share clip / Dispute |

---

## 3. Booking → Session (cross-system ripple)

How a session appears on a coach's worklist in the first place.

```mermaid
sequenceDiagram
    participant S as Student (Timeback)
    participant OC as OpenCal (backend)
    participant AVC as Coaching Platform
    participant C as Coach
    participant G as Guide

    S->>OC: Opens booking link (subject/grade pre-filled)
    OC->>OC: Match attribute matrix → rank by buffer/cap/priority
    OC-->>S: Show matched coach's open slots
    S->>OC: Picks slot
    OC->>AVC: webhook booking.created
    AVC->>AVC: Create CoachSession
    AVC-->>C: Appears on today's worklist
    AVC-->>G: Appears on guide dashboard
    OC-->>S: Confirmation email + Meet link + reminder
```

Guide-initiated booking (Flow G4) enters the same pipe one step later — guide picks slot inside AVC, which calls OpenCal.

---

## 4. Notes dispatch + QC (what happens on "Mark complete")

```mermaid
flowchart TD
    MC[Coach marks complete] --> Notes{Recipients ticked?}
    Notes -->|Guide| G[Guide inbox + profile updated]
    Notes -->|Subject DRI| SD[Subject DR notes report]
    Notes -->|Campus DRI| CD[Campus DR inbox]
    MC --> QCrun[QC fires via Social Toolkit]
    QCrun -->|~2 min| QCdone[QC tab unlocks + coach notified]
    QCdone -->|PASS or FAIL| Coach[Coach reviews]
    Coach -.disagree.-> Disp[Dispute → Coaching DR manual review]
```

---

## 5. Flag fan-out (auto-detection → escalation)

```mermaid
flowchart TD
    Sys[System watches booking patterns] --> Check{Pattern?}
    Check -->|5 calls same week| HF[High-frequency flag]
    Check -->|5 calls same subject| SS[Same-subject double flag]

    HF --> CDR[Coaching DR + Campus DR]
    SS --> Triple[Coaching DR + Campus DR + Subject DR]

    CDR --> Slack1[Slack ping to campus channel]
    Triple --> Slack2[Slack ping to campus channel]
    CDR --> App[In-app flag in Flags page]
    Triple --> App

    App --> Triage{Coaching DR triages}
    Triage -->|escalate| Notify[Notify DRIs]
    Triage -->|benign| Dismiss[Dismiss]
```

---

## 6. Guide flow

```mermaid
flowchart LR
    GD[Guide dashboard] --> Today[Students coached today]
    Today --> Prep[Open student profile]
    Prep --> Note[Write prep context]
    Note -->|surfaces in| CoachBefore[Coach's Before tab]
    Prep --> Book[Book session for student]
    Prep --> Flag[Flag student → DRIs]
    GD --> Review[Read coach notes after session]
    Review --> Update[Update student profile]
```

---

## 7. Admin / Coaching DR daily flow

```mermaid
flowchart TD
    AO[Admin overview] --> Scan[Scan: sessions today · flags · notes overdue]
    Scan --> FlagQ{Flags waiting?}
    FlagQ -->|yes| Triage[Triage flags → notify/dismiss]
    Scan --> CoachQ{Coach underwater?}
    CoachQ -->|yes| CD[Open coach detail → view-as-coach]
    AO --> Config[Config when needed: attributes · routing · workflows]
    AO --> OwnCoaching[Her own coaching = full coach loop]
```

---

## 8. Navigation principles (why it feels fluid)

1. **Orient** — labeled sidebar always visible; breadcrumb in topbar shows `Section / Page / Item`.
2. **Momentum** — every screen has one primary next action; dashboards are worklists that pull you through the day.
3. **Continuity** — student/coach names are clickable everywhere; "back" returns to where you came from; no dead-ends — empty states point to the next thing.
