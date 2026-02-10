# Patient card state machine

**Purpose:** Define the patient card state machine for the PSA follow-up dashboard, distinguish implemented vs mock behaviour, and clarify manual vs automated transitions. Aligned with [WIREFRAME_POC_SPEC.md](WIREFRAME_POC_SPEC.md) and [CLINICAL_WORKFLOW_BASE_PLAN.md](CLINICAL_WORKFLOW_BASE_PLAN.md).

**Current implementation:** See [PROTOTYPE_STATE.md](PROTOTYPE_STATE.md) for what is implemented in the live prototype (board, list, detail, actions, sync).

---

## 1. States (current)

The patient card has **seven states**. The **board** shows five columns (Recall and Discharged are off the board). The **list** has tabs for all seven.

| State (internal)   | Column / tab label   | Meaning (from spec & copy) |
|--------------------|----------------------|----------------------------|
| `upcoming`         | Upcoming             | PSA due in next 4 weeks; move to Awaiting Results when test done. |
| `awaiting`         | Awaiting Results     | Blood taken; waiting for result. |
| `action`           | Review Required      | PSA above threshold; GP decides Completed, Action Required, or Recall. |
| `action_required` | Action Required      | Follow-up action chosen (e.g. USC, Advice & guidance); move to Recall when done. |
| `completed`        | Completed            | Result normal; card stays 7 days then auto to Recall (copy; no timer in POC). |
| `recall`           | Recall               | Back to clinic; not on board. |
| `discharged`       | Discharged           | Off list; not on board. |

**Data and sync:** State is stored in `detailData[id].status`, each card’s `data-status`, and each list row’s `data-tab` / `data-status`. Moves from the 3-dot menu or detail action cards **update all three** so board, list, and detail stay in sync.

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
     │  │ Upcoming │ ──(0)──► Move to Awaiting Results (implemented)             │
     │  └──────────┘                                                                   │ Discharge (implemented) │
     │       ▲                                                                         ▼                    │
     │       │  (5) “Card stays 7 days then auto to Recall” — not implemented (no timer)  [off board / archive] │
     │       │      — no next-due calculation in POC                                       │
     └───────┴──────────────────────────────────────────────────────────────────────────────────────────────┘
```

**Diagram note:** The ASCII diagram above is simplified. The full transition set and implementation status are in §2.2 and in [patient-card-state-machine.mmd](patient-card-state-machine.mmd).

### 2.2 Transition table: basic (state-changing) transitions

| From → To        | Trigger / action              | Implemented? | Notes |
|------------------|------------------------------|-------------|-------|
| — → Upcoming     | Add Patient                  | ✅ Yes      | New patient lands in Upcoming. |
| Upcoming → Awaiting | “Move to Awaiting Results”  | ✅ Yes      | 3-dot menu and detail; syncs board, list, detailData. |
| Awaiting → Review Required | “Move to Review Required” | ✅ Yes      | Same; full sync. |
| Awaiting → Completed | “Move to Completed”        | ✅ Yes      | Same. |
| Review Required → Completed | “Move to Completed”     | ✅ Yes      | Same. |
| Review Required → Action Required | “Move to Action Required” | ✅ Yes  | Same; required_action set separately. |
| Review Required → Recall | “Move to Recall”          | ✅ Yes      | Card removed from board; list/detailData updated. |
| Action Required → Recall | “Move to Recall”         | ✅ Yes      | Same. |
| Completed → Recall | “Move to Recall” (optional) | ✅ Yes     | Manual shortcut; copy says 7-day auto (not implemented). |
| Completed → (leave board) | 7-day rule               | ❌ No       | Copy only; no timer in POC. |
| Completed → Upcoming | Next due                    | ❌ No       | No next-due calculation. |
| Recall → Discharged | “Discharge”                | ✅ Yes      | From detail Discharge subsection or 3-dot (list); updates status, list, removes card. No archive UI. |
| Any → Discharged    | “Discharge”                | ✅ Yes      | Discharge subsection in detail for all non-discharged statuses. |

### 2.3 Actions that do **not** change state (mock or side-effect only)

| Action                  | State(s)   | Implemented effect        | Mark as mock? |
|-------------------------|------------|---------------------------|---------------|
| Send reminder           | Upcoming   | Toast only               | **Mock**      |
| Save result (PSA+date)  | Awaiting / Review | Updates `detailData` PSA/date only; no state change | No (data only) |
| Discharge               | All (non-discharged) | Updates status to discharged; removes card; no archive UI | Implemented   |
| View details            | All        | Opens detail modal       | No            |

---

## 3. Protocol / guideline alignment

- **WIREFRAME_POC_SPEC.md** and **CLINICAL_WORKFLOW_BASE_PLAN.md** define the flow. Board shows 5 columns; Recall and Discharged off board. Completed copy: 7 days then auto to Recall (no timer). Discharge from detail subsection and 3-dot menu; no archive UI.
- **PROTOTYPE_STATE.md** describes current implementation: sync on all moves, Action Required column, Discharge subsection, “Move to Awaiting Results” etc.

**Conclusion:** All manual transitions are implemented with full sync. Gaps: no 7-day timer, no next-due calculation, no archive list. Send reminder is mock (toast).


## 4. Open questions and future work

- **Definition of "result in cycle";** late/overdue results; correction and undo.
- **5‑ARI effect on threshold;** 2 consecutive rises / bounce; protocol rules as system behaviour.
- **Discharge meaning and archive/audit;** blood forms and reminders integration.
- **Safety nets** (e.g. stuck in Awaiting, no result when marking completed); reason codes for override/recall/discharge.
- **Automation:** triage suggestion or auto-move on Save result; 7-day auto to Recall; next-due calculation; lab feed. See [CLINICAL_WORKFLOW_BASE_PLAN.md](CLINICAL_WORKFLOW_BASE_PLAN.md) and [PROTOTYPE_STATE.md](PROTOTYPE_STATE.md) §6 for open questions.
