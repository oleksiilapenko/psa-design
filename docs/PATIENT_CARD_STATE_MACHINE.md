# Patient card state machine

**Purpose:** Define the patient card state machine for the PSA follow-up dashboard, distinguish implemented vs mock behaviour, and clarify manual vs automated transitions. Aligned with [WIREFRAME_POC_SPEC.md](WIREFRAME_POC_SPEC.md), [NEXT_STEPS.md](NEXT_STEPS.md), and [ITERATION_CURRENT_STATE.md](ITERATION_CURRENT_STATE.md).

**No code changes in this doc — definition and questions only.**

---

## 1. States (current)

The patient card has **five states**, mapped 1:1 to Kanban columns and List tabs:

| State (internal) | Column / tab label   | Meaning (from spec & copy) |
|------------------|----------------------|----------------------------|
| `upcoming`       | Upcoming             | PSA test due in the next 4 weeks; generate forms/reminders. |
| `awaiting`       | Awaiting Results     | Blood taken or sample received; waiting for result. |
| `action`         | Review Required      | Latest PSA above threshold; clinician decides Completed vs Recall. |
| `completed`      | Completed            | PSA at or below threshold; card leaves board after 48h; reappears in Upcoming when next due. |
| `recall`         | Recall               | No longer on self-management; back to clinic. |

**Data:** In the prototype, state is stored in (a) each card’s `data-status`, (b) each list row’s `data-tab` / `data-status`, and (c) each patient’s `detailData[id].status`. Currently, **only (a) is updated when moving a card via the 3-dot menu**; (b) and (c) are static from initial HTML/mock data, so List and detail modal can show stale status after a move.

---

## 2. Current state machine (as implemented)

### 2.1 Diagram (states and transitions)

```
                    ┌─────────────────────────────────────────────────────────┐
                    │  [Add Patient] → new patient lands in Upcoming          │
                    └─────────────────────────────────────────────────────────┘
                                                     │
     ┌───────────────────────────────────────────────┼───────────────────────────────────────────────┐
     │                                               ▼                                                       │
     │  ┌──────────┐     (1)                         ┌──────────┐     (2a)        ┌─────────────┐          │
     │  │ Completed│◄────────────────────────────────│ Awaiting │────────────────►│Review Req’d │          │
     │  └──────────┘     Mark as completed            └──────────┘  Move to Rev.  └──────┬──────┘          │
     │       ▲                    (manual)                 ▲                (manual)     │                  │
     │       │                                              │                            │ (2b)              │
     │       │              (3a)                           │                            │ Move to Recall   │
     │       │         Mark as completed                    │                            ▼ (manual)         │
     │       │              (manual)                        │                     ┌──────────┐             │
     │       └──────────────────────────────────────────────┴─────────────────────│  Recall  │             │
     │                            from Review Required                            └──────────┘             │
     │                                                                                  │                    │
     │  ┌──────────┐                                                                   │ (4)                │
     │  │ Upcoming │ ──(0)──► ??? (Upcoming → Awaiting: not in UI; “Test done” implied in spec)             │
     │  └──────────┘                                                                   │ Discharge (mock)   │
     │       ▲                                                                         ▼                    │
     │       │  (5) “Card leaves after 48h; reappears in Upcoming when next due”       [off board / archive] │
     │       │      — not implemented (no 48h timer, no next-due calc)                                       │
     └───────┴──────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Transition table: basic (state-changing) transitions

| From → To        | Trigger / action              | Implemented? | Notes |
|------------------|------------------------------|-------------|-------|
| — → Upcoming     | Add Patient                  | ✅ Yes      | New patient lands in Upcoming. |
| Upcoming → Awaiting | “Test done” / blood taken   | ❌ No       | Spec flow says “Upcoming → Awaiting (e.g. ‘Test done’)”; no button or menu action in prototype. |
| Awaiting → Review Required | “Move to Review Required” | ✅ Yes (Kanban only) | 3-dot menu: moves card DOM + `data-status`; **detailData and List row not updated**. |
| Awaiting → Completed | “Mark as completed”        | ✅ Yes (Kanban only) | Same as above. |
| Review Required → Completed | “Mark as completed”     | ✅ Yes (Kanban only) | Same. |
| Review Required → Recall | “Move to Recall”          | ✅ Yes (Kanban only) | Same. |
| Completed → (leave board) | 48h rule                 | ❌ No       | Spec: card leaves after 48h; no timer in POC. |
| Completed → Upcoming | Next due                    | ❌ No       | Spec: “patient reappears in Upcoming when next due”; no calculation or auto-move. |
| Recall → (Discharge) | “Discharge”                | Mock only  | Toast only; no status change, no archive. |

### 2.3 Actions that do **not** change state (mock or side-effect only)

| Action                  | State(s)   | Implemented effect        | Mark as mock? |
|-------------------------|------------|---------------------------|---------------|
| Generate blood form     | Upcoming   | Toast only               | **Mock**      |
| Send reminder           | Upcoming   | Toast only               | **Mock**      |
| Chase result            | Awaiting   | Toast only               | **Mock**      |
| Save result (PSA+date)  | Awaiting / Review | Updates `detailData` PSA/date only; no state change | No (data only) |
| Discharge               | Recall     | Toast only               | **Mock**      |
| Print summary           | Completed  | Toast only               | **Mock**      |
| View / edit patient     | All        | Opens detail modal       | No            |

---

## 3. Protocol / guideline alignment

- **WIREFRAME_POC_SPEC.md**  
  - Flow: Upcoming → Awaiting Results (e.g. “Test done”) → Completed (normal) or Review Required (abnormal) → Completed or Recall; Discharge from Recall (illustration only).  
  - Out of scope for POC: “Blood forms / reminder generation”, “Real PAS/Lab integration”.  
  - So: **Upcoming → Awaiting** is in spec but not implemented; **blood form / reminder** are explicitly out of scope (mock only).

- **NEXT_STEPS.md “Capability questions”**  
  - Confirms open questions: Will “Generate blood form” produce PDF or trigger another system? Will “Send reminder” exist separately? Will result entry be manual only or partly from lab feed? Will system **automatically** move card to Completed/Review when result is entered, or user always chooses?  
  - So: **Automation of “result in → triage” is undecided**; current behaviour is manual (user chooses Mark completed / Move to Review).

- **ITERATION_CURRENT_STATE.md**  
  - “Column moves: Manual only (buttons / 3-dot)”.  
  - “Completed and 48h rule: No 48h timer or next due calculation in the POC.”  
  - “Discharge: Action only; no archive UI.”

**Conclusion:** The state machine in code matches the spec’s **states** and **manual** transitions for Awaiting ↔ Review Required ↔ Completed ↔ Recall. Gaps: (1) **Upcoming → Awaiting** not in UI, (2) **Completed → leave board / reappear in Upcoming** not implemented, (3) all “external” actions (blood form, reminder, chase, discharge, print) are **mock**.

---

## 4. Focus: basic transitions first

For “most basic transitions first”, the minimal set is:

1. **Upcoming → Awaiting**  
   - **Current:** Missing.  
   - **Proposal:** One explicit manual action, e.g. “Blood taken” / “Test done”, that sets status to `awaiting` and (when we have persistence) updates `detailData` and list row.

2. **Awaiting → Completed or Review Required**  
   - **Current:** Implemented via 3-dot menu only; card and `data-status` move; `detailData` and list do not.  
   - **Proposal:** Keep manual choice; ensure one source of truth (e.g. `detailData.status`) and sync board + list + detail when user chooses “Mark as completed” or “Move to Review Required”. Optionally: after “Save result”, suggest or auto-fill triage based on PSA vs threshold (see automation below).

3. **Review Required → Completed or Recall**  
   - **Current:** Same as (2)—manual via 3-dot; DOM/`data-status` only.  
   - **Proposal:** Same: single source of truth + sync on “Mark as completed” / “Move to Recall”.

4. **Recall → Discharge**  
   - **Current:** Mock (toast).  
   - **Proposal:** Leave as mock until product defines meaning (archive, PAS, audit). For basic state machine, can treat Discharge as “leave active list” (new state or flag) when product is ready.

5. **Completed → (leave board) and → Upcoming**  
   - **Current:** Not implemented.  
   - **Proposal:** Defer until “next due” and 48h rules are defined; then either time-based (auto) or manual “Remove from board” / “Reappear when due”.

So “basic” = implement or clarify **Upcoming → Awaiting**, and **sync state** across board, list, and `detailData` for existing manual moves; then consider automation.

---

## 5. Automation (later)

Possible automation (to be decided with product/clinical):

- **On “Save result” (Awaiting):**  
  - Option A: System suggests “Mark as completed” vs “Move to Review Required” from PSA vs threshold (user still confirms).  
  - Option B: System **auto-moves** to Completed if PSA ≤ threshold, to Review Required if above (user can override).  
  - Spec copy says “the system will triage” — so some automation is in scope later; POC is currently manual.

- **Lab feed:**  
  - If results arrive via integration, same triage rule could run on “result received” and suggest or auto-move.

- **48h and next due:**  
  - When implemented: auto-hide card from board after 48h in Completed; auto-show in Upcoming when next due (from protocol + last test date).

- **Safety nets (ITERATION_CURRENT_STATE):**  
  - e.g. “Stuck in Awaiting > 14 days”, “PSA above threshold not reviewed” — could drive alerts or suggested actions, not necessarily auto-move.

---

## 6. Questions to align scope and next steps

### 6.1 What the system can already provide (confirm or correct)

- **Manual moves:** User can move a card from Awaiting to Completed or Review Required, and from Review Required to Completed or Recall, via 3-dot menu. The move is only reflected on the **board** (card position and `data-status`); **list** and **detail** still read from initial `detailData`/HTML.  
  **Q:** Is “single source of truth + sync board/list/detail on every move” the intended next step for “basic” transitions?

- **Save result:** User can enter or edit PSA and date in the detail modal; this updates `detailData` only (no state change).  
  **Q:** Should “Save result” ever directly trigger a state change (e.g. auto-move to Completed if PSA ≤ threshold), or should we only add a **suggestion** (“Result below threshold — consider Mark as completed”) and keep the actual move manual?

- **Add Patient:** New patient lands in Upcoming.  
  **Q:** Any other “already provided” behaviour we should treat as fixed for the state machine (e.g. threshold override, 5‑ARI, diagnoses)?

### 6.2 Nice-to-have / put on hold

- **Generate blood form, Send reminder, Chase result:** Currently mock (toast). Spec and NEXT_STEPS say blood forms/reminders are out of scope for POC.  
  **Q:** Confirm these stay mock/on-hold until product defines integration (PDF, task, letter, etc.)?

- **Discharge and archive:** Mock; no archive list or audit trail.  
  **Q:** Keep as mock until “Discharge” meaning and storage are defined?

- **Print summary:** Mock.  
  **Q:** On hold until required for a specific workflow?

- **48h rule and “next due” / reappear in Upcoming:** Not implemented.  
  **Q:** Defer until protocol rules (next due calculation, 48h) are locked?

- **Auto-suggest / auto-move** based on PSA vs threshold or “2 consecutive rises” / bounce: Not implemented.  
  **Q:** Treat as “later / automation” phase after basic manual transitions and sync are solid?

### 6.3 Upcoming → Awaiting

- Spec describes “Test done” (or equivalent) to move from Upcoming to Awaiting.  
  **Q:** Do we want a single explicit action in the UI (e.g. “Blood taken” / “Test done”) for this transition in the next iteration? If yes, should it live in 3-dot menu, detail Actions, or both?

### 6.4 Protocol and safety

- **Reason codes:** No structured “reason for override” or “reason for Recall/Discharge” in prototype.  
  **Q:** Are reason codes in scope for the first “basic” state machine release, or on hold for audit/MDT later?

- **No-PSA / missing result:** One mock patient (Wilson) is in Awaiting with no PSA.  
  **Q:** Should “Mark as completed” be blocked or warned when there is no numeric result, or is that a later safety-net?

---

## 7. Summary

| Item | Current | Proposed focus |
|------|--------|-----------------|
| **States** | 5: Upcoming, Awaiting, Review Required, Completed, Recall | Keep; document as above. |
| **Basic transitions** | Upcoming→Awaiting missing; Awaiting↔Review↔Completed↔Recall implemented on board only; detailData/list not updated | Add Upcoming→Awaiting; sync state to detailData + list on every move. |
| **Mock actions** | Generate blood form, Send reminder, Chase result, Discharge, Print summary | Keep mock; no “(mock)” in production label (NEXT_STEPS); hold until product defines. |
| **Automation** | None; “system will triage” is copy only | After basic sync: consider suggest or auto-move on Save result / lab feed; 48h and next-due when rules defined. |

Once the questions in §6 are answered, the state machine can be tightened (e.g. formal transition table and guards) and implementation can proceed for basic transitions and sync, then automation.
