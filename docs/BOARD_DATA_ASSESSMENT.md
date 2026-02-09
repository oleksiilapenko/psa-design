# Board mock data assessment & redistribution plan

## 0. Protocol spec vs implementation (double-check)

**Wireframe POC spec (WIREFRAME_POC_SPEC.md)**  
- **In scope:** 3 protocols only — **Radical Prostatectomy, EBRT, Brachytherapy**.  
- **Out of scope (for now):** “ADT, Watchful Waiting / Active Surveillance”.

**NEXT_STEPS.md (extension plan)**  
- “Next: ADT, then Watchful Waiting / **Active Surveillance** (different rules: doubling time, rise > 0.75).”  
- So the intended extension set is: ADT, Watchful Waiting, and **Active Surveillance** (possibly separate from Watchful Waiting).

**What the prototype actually has**  
- **6 protocols** in wizard, `protocolConfig`, filter, and mock data:  
  Radical Prostatectomy, EBRT, Brachytherapy, **Primary ADT**, **Intermittent hormones**, **Watchful waiting**.  
- **Active Surveillance** is **not** implemented as a separate protocol.  
- **Intermittent hormones** is in the build but is **not** listed in the spec’s in/out-of-scope table or in NEXT_STEPS as a “next” protocol; it was added as an extra.

**Summary**  
- **Spec’s original 3:** All supported (RP, EBRT, Brachy).  
- **Spec’s “next” list (ADT, Watchful Waiting, Active Surveillance):** ADT ✅, Watchful waiting ✅, **Active Surveillance ❌** (no separate option; only “Watchful waiting” exists).  
- **Extra in build:** Intermittent hormones (not in spec).  

So we are **not** fully aligned with the spec: we have one extra (Intermittent hormones) and are missing **Active Surveillance** as a distinct protocol if the spec intends it to be separate. For the current mock and filter we are “covering all protocols **that exist in the codebase**” (6); the spec would call for adding Active Surveillance (and optionally documenting Intermittent hormones) to be fully supported.

---

## 1. Current state

### Protocols (filter options = 6) — as implemented today
All six protocols currently in the app are present in the mock and in the filter dropdown:
- **Radical prostatectomy** (RP) — threshold 0.03
- **EBRT** — 2.0 (Phoenix)
- **Brachy** (Brachytherapy) — 0.5
- **ADT** (Primary ADT) — 4.0
- **Intermittent hormones** — 10.0
- **Watchful waiting** — 30.0

Display mapping: `detailData.protocol` uses internal names (e.g. `Radical Prostatectomy`, `Primary ADT`); list/cards use short labels via `protocolShortLabel` or hardcoded text (Radical prostatectomy, EBRT, Brachy, ADT, Intermittent hormones, Watchful waiting). Filter compares to these display strings.

### Board columns (temporal flow)
1. **Upcoming** — PSA due in next 4 weeks; generate forms / reminders  
2. **Awaiting Results** — Blood taken or sample received; card moves to Completed or Review when result arrives  
3. **Review Required** — PSA above threshold; clinician decides Completed vs Recall  
4. **Completed** — PSA ≤ threshold; card leaves board after 48h; patient reappears in Upcoming when next due  
5. **Recall** — No longer on self‑management; back to clinic  

### Current distribution (16 patients)

| Column      | Count | Patients | Protocols in column |
|------------|-------|----------|----------------------|
| Upcoming   | 5     | John Smith (RP Y1), Alan Jones (EBRT Y1), Nicholas Bell (Brachy Y1), David Patel (Intermittent Y1), Frank Wright (Watchful Y2) | RP, EBRT, Brachy, Intermittent, Watchful |
| Awaiting   | 2     | David Brown (RP Y3), Robert Wilson (EBRT Y2) | RP, EBRT |
| Review     | 1     | Michael Clark (RP Y2) | RP |
| Completed  | 2 on board | Peter Green (RP Y4), Ian Collins (ADT Y2) | RP, ADT |
| Recall     | 3     | James Taylor (RP), Philip Hayes (EBRT), Stephen Webb (Brachy) | RP, EBRT, Brachy |

**List vs board:** The list has 16 rows (all patients). The board shows 13 cards; **George Walsh, Terence Bennett, Kevin Shaw** (all Completed) exist in the list but have no card on the board. The Completed column note says “not all shown on board”, so this is intentional.

### Protocol coverage by column
- **Radical prostatectomy:** Upcoming, Awaiting, Review, Completed, Recall ✓  
- **EBRT:** Upcoming, Awaiting, Recall ✓ (Completed only in list: Terence Bennett)  
- **Brachy:** Upcoming, Recall ✓  
- **ADT:** Completed only (Ian Collins) ✓  
- **Intermittent hormones:** Upcoming only  
- **Watchful waiting:** Upcoming only  

So **all protocols are covered** in the dataset. Intermittent hormones and Watchful waiting appear only in Upcoming; moving one of each to another column would better illustrate the pipeline and protocol variety.

---

## 2. What we’re showcasing

- **Protocol variety:** Different thresholds and schedules (4‑monthly RP Y1, 6‑monthly EBRT/Brachy/ADT, etc.).  
- **Temporal nature:** Cards move Upcoming → Awaiting → (Completed or Review) → optionally Recall. The board should look like a snapshot of “work in progress”, not a static list.  
- **Natural look:** In practice, not everyone is “upcoming” at once; some have just had blood drawn (awaiting), some have just been signed off (completed). A balanced spread (e.g. 2–3 Upcoming, 2–3 Awaiting, 2–3 Completed on board) reads more realistically.

---

## 3. Recommended redistribution (no code yet)

**Goal:** Reduce Upcoming from 5 to 3; add one Awaiting and one Completed so protocols and temporal flow are clearer. No new patients; only status/lane moves with consistent story.

### Move 1: David Patel (Intermittent hormones) → **Awaiting**
- **Story:** Blood was due 1 Mar 2025; sample just received, result pending.  
- **detailData:** `status: 'awaiting'`, `nextDue: 'Awaiting result'`, `daysAwaiting: 3`, `lastBleed: '3 days ago'`, `lastUpdated: '4 Feb 2025'`. Keep PSA 6.0, threshold 10.0, spark, etc.  
- **Board:** Move Patel’s card from Upcoming to Awaiting column; `data-status="awaiting"`. `injectAwaitingDueLines()` will set due line to “Waiting for 3 days”.  
- **List:** Row `data-tab="awaiting"` `data-status="awaiting"`, status badge “Awaiting”; Next Due cell is set by `populateListFromDetailData()` from `daysAwaiting`.  
- **Result:** Intermittent hormones appears in Awaiting as well as Upcoming.

### Move 2: Frank Wright (Watchful waiting) → **Completed**
- **Story:** Last PSA 18 (threshold 30); result just reviewed and signed off; next due Jun 2025.  
- **detailData:** `status: 'completed'`, keep `nextDue: 'Next due Jun 2025'`, `lastUpdated: '5 Feb 2025'` (optional).  
- **Board:** Move Frank’s card from Upcoming to Completed column; `data-status="completed"`. Card text already has “Last PSA 18 / 30.0 ✓”; ensure due line “Next due Jun 2025”.  
- **List:** Row `data-tab="completed"` `data-status="completed"`, status badge “Completed”; Next Due stays “Jun 2025”.  
- **Result:** Watchful waiting appears in Completed, so that protocol isn’t only in Upcoming.

### After redistribution
- **Upcoming:** 3 — John Smith (RP), Alan Jones (EBRT), Nicholas Bell (Brachy).  
- **Awaiting:** 3 — David Brown (RP), Robert Wilson (EBRT), David Patel (Intermittent).  
- **Review:** 1 — Michael Clark (RP).  
- **Completed (on board):** 3 — Peter Green (RP), Ian Collins (ADT), Frank Wright (Watchful).  
- **Recall:** 3 — James Taylor (RP), Philip Hayes (EBRT), Stephen Webb (Brachy).  

Protocols in multiple columns: RP (all five), EBRT (4), Brachy (2), ADT (1), Intermittent (2), Watchful (2). Temporal flow is clearer; board looks less “stuck” in Upcoming.

---

## 4. Code touchpoints (when you ask to execute)

- **detailData:** Update `david-patel` (awaiting, daysAwaiting, nextDue, lastBleed, lastUpdated) and `frank-wright` (status completed, nextDue/lastUpdated as above).  
- **Board HTML:** Move Patel card into Awaiting `.column-cards`, set `data-status="awaiting"`. Move Frank card into Completed `.column-cards`, set `data-status="completed"`.  
- **List HTML:** Patel row → `data-tab="awaiting"` `data-status="awaiting"`, badge “Awaiting” / `status-awaiting`, Next Due cell can stay as-is (script overwrites for awaiting). Frank row → `data-tab="completed"` `data-status="completed"`, badge “Completed” / `status-completed`.  
- **No changes:** Filter/sort logic, protocolShortLabel, injectAwaitingDueLines/populateListFromDetailData (they key off `detailData` and `data-status`/`data-tab`).  

Ensure list row order remains consistent with tab order (e.g. upcoming rows, then awaiting, then action, completed, recall) if the DOM order is used for anything; reordering rows for the two moved patients may be needed so they appear under the right list tab.

---

## 5. TLDR

- **Protocols:** All six are already in the mock; filter and display are aligned.  
- **Board:** Currently 5 Upcoming, 2 Awaiting, 1 Review, 2 Completed (on board), 3 Recall.  
- **Issue:** Upcoming is overloaded; Intermittent hormones and Watchful waiting only appear there.  
- **Plan:** Move **David Patel (Intermittent hormones)** to **Awaiting** (blood just taken, waiting 3 days) and **Frank Wright (Watchful waiting)** to **Completed** (result just signed off).  
- **Result:** 3 Upcoming, 3 Awaiting, 1 Review, 3 Completed on board, 3 Recall; protocols better spread across columns; board reads as a realistic, temporal workflow.  
- **Execute:** When you say so, apply the `detailData` updates and the board/list HTML moves above; no other code changes required.
