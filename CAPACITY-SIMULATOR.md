# Coaching Capacity Simulator — Design Doc

**Status:** Design / set-aside (not yet built)
**Owner:** Goutham
**Date:** 2026-06-10
**Depends on:** Himanshi's written routing spec (for the *live* policy only — the simulator below can be built without it)

---

## 1. The problem

Demand concentrates in the **8:00–12:00 CT** window (4 hours). We want to **maximize the number of coaching calls served in that window** — equivalently, maximize coach utilization without fragmenting calendars.

This is not abstract: on **1 May 2026** the campus booked out (coaches on leave), and the week of **27 Apr 2026** was heavy. Himanshi wants a concrete comparison: *what did Calendly actually serve vs. what could we have served.*

## 2. The math (constraint geometry)

- Window = 240 min. Call = 30 min. Buffer = 5 min between consecutive calls.
- `n` calls occupy `35n − 5` minutes → **7 calls fit exactly** (35×7 − 5 = 240).
- 10-min buffer → 6 calls (40×6 − 10 = 230 ≤ 240).
- Per-coach geometric ceiling in window = **7**; current policy target = **5** (the other 2 of 7/day fall outside the window).
- With `N` coaches: in-window ceiling `7N`, policy target `5N`.

> **OPEN — pending Himanshi's spec:** transcript says *7 in window*; handoff says *5 in window / 7 day*. Also the "30-min from previous call end" line is ambiguous (cannot be a per-call gap or the math collapses to 4 calls/window). The simulator parameterizes buffer + in-window cap so either resolution drops in.

## 3. Problem classification

**Multi-machine interval scheduling with eligibility + setup times:**
- machines = coaches, jobs = calls,
- eligibility = call runs only on a coach whose attributes match (subject × grade × call-type),
- setup time = buffer between consecutive calls on the same coach.

At this scale (≈3–10 coaches × ≤7 slots) instances are tiny → exact optimization is cheap; no heavy OR machinery required.

## 4. Two modes

### Mode A — Live routing (online, production, explainable)
Bookings arrive one at a time; future is unknown → optimal packing impossible. Use a greedy heuristic (near-optimal at this scale, and explainable to ops):

1. Compute **eligible coaches** for the request (attribute match + availability).
2. Filter to coaches where the requested/offered slot is **buffer-feasible** (≥5 min from every existing call, within caps).
3. Rank:
   1. **Packing fit** — prefer a coach where the slot is adjacent to an existing call (fills a gap) over opening an idle coach. *(Fights fragmentation — the core lever.)*
   2. **Priority rotation** — among similar fits, the coach with fewest calls yesterday goes first (weekly load equalization).
4. Assign; on **no-show**, drop the buffer and free the coach early (raises effective capacity).

Because students pick from **offered** slots, the real control point is *which slots we surface*: bias toward contiguous slots on the priority coach; open a fresh coach lazily.

### Mode B — Capacity simulation (offline, analysis/demo)
Demand for a day is fully known → solve for the true maximum.

1. Ingest the day's bookings as fixed `(subject, grade, call-type, duration, requested-time?)` items.
2. Run an **exact packer** (tiny ILP or brute-force; instance is small) to compute the max calls placeable under the constraint model.
3. Also run **Mode A's greedy** on the same demand to show what our live routing would have achieved.
4. Report the three numbers.

## 5. Data inputs

From the Calendly scheduled-calls sheet / export, per booking:
| Field | Use |
|---|---|
| start time | place interval |
| duration (or derive from call-type) | interval length |
| coach (organizer) | machine identity |
| subject / grade / call-type | eligibility |
| status (completed / no-show / cancelled) | model no-show buffer-removal |

Reverse-analysis needed: meet recordings only carry a meet ID, so subject/grade mapping must come from the tracker sheet join (already a known gap from the call).

## 6. Output (the demo artifact)

For 1 May + week of 27 Apr:

```
Date: 2026-05-01  ·  Window 08:00–12:00 CT  ·  Coaches available: 4
  Calendly actually served:      14 calls
  Our routing (greedy) would do: 18 calls   (+29%)
  Theoretical ceiling:           20 calls   (5/coach policy)  /  28 (7/coach geometric)
  Idle-gap time reclaimed:       2h 10m
```

Plus a per-coach utilization bar and a fragmentation breakdown (minutes lost to unfillable gaps under Calendly vs. our routing).

## 7. Implementation notes

- Pure function, no infra: input = booking list + coach roster + config (buffer, caps, window); output = schedule + metrics. Trivially unit-testable.
- Config object holds the contested params (buffer 5/10, in-window cap 5/7, the 30-min rule on/off) so Himanshi's final spec is a one-line change.
- Exact solver: at ≤10 coaches × ≤7 slots, a constraint-respecting DFS with pruning finds the optimum in milliseconds; ILP (OR-tools) is optional overkill kept behind a flag for larger what-ifs (more coaches, wider window).
- Surface it in the existing **Routing Simulator** admin page as a "Backtest against history" tab.

## 8. Open questions (carry into Himanshi's spec review)

1. In-window cap: 5 or 7? Buffer: 5 or 10 min?
2. Is the "30-min from previous call end" a real constraint or a transcription artifact?
3. Variable durations (Literally Testing 10–15 min; Proctoring 15–30) — fixed per call-type, or learned from data patterns?
4. Priority-rotation tiebreak vs. packing-fit: which wins when they conflict? (Proposed: packing fit first, priority as tiebreak.)
