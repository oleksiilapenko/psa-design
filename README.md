# NHS Prostate Cancer Follow-Up Dashboard — Prototype

Single-file HTML prototype for Pan Dorset PSA follow-up pathways. Mock data and client-side logic only; no backend.

## Run

Open in a browser (no server required):

```bash
open index.html
```

---

## What’s in the prototype

### Dashboard

- **Board (Kanban):** 5 columns — **Upcoming**, **Awaiting Results**, **Completed**, **Review Required**, **Action Required**. Recall and Discharged are not on the board (actionable patients only). Cards show name, NHS No, protocol + phase, sparkline, latest PSA vs threshold, next due. Review Required and Action Required cards show Δ vs threshold. Each card has a 3-dot menu: View details, and status-specific moves (e.g. Move to Awaiting Results, Move to Completed, Move to Recall, Discharge where applicable).
- **List:** Tabs in groups — **All** (separate); **Pathway:** Recall, Upcoming, Awaiting, Completed; **GP focus:** Review Required, Action Required; **Exit:** Discharged. Table: Patient, NHS No, Status, Protocol, Phase, Diagnoses, Trend, Last PSA, Threshold, Δ, Next Due, Last updated. Row click opens patient detail.
- **View toggle:** Kanban ⇄ List. Same data in both views.

### Add Patient wizard (3 steps)

- **Step 1:** NHS Number → Search → mock result (name, DOB, NHS No) → Continue.
- **Step 2:** Diagnoses (optional), Protocol (radios + help), anchor date → phase/frequency read-only → Continue.
- **Step 3:** 5‑ARI, protocol default threshold, personal threshold (override → *), last known PSA + date → Add patient (lands in Upcoming; toast).

### Patient detail (card or list click)

- **Sections:** Patient (name, NHS No, DOB, protocol + phase, status badge); Clinical context (Diagnoses, 5‑ARI, editable); Pathway (protocol, phase, frequency, treatment start, required action when applicable); PSA & threshold (sparkline, latest PSA, Δ, threshold edit; result entry for Awaiting/Review Required); Key dates (next due, last bleed).
- **Actions:** Status-driven **action cards** (e.g. Move to Awaiting Results, Move to Completed, Move to Review Required, Move to Recall) with short descriptions where helpful. **Discharge** is in a separate subsection below, with a red left accent; available for all non-discharged patients. All moves sync board, list, and detail.

### Conventions

- **Names:** Lastname, Firstname (age). **Δ:** PSA − threshold (↑ above, ↓ below). **Completed:** cards stay 7 days then automatically move to Recall (copy; no timer in prototype). **Title case** for column names: Upcoming, Awaiting Results, Completed, Review Required, Action Required, Recall, Discharged.

---

## Protocols (in prototype)

| Protocol              | Default threshold | Short label   |
|-----------------------|-------------------|---------------|
| Radical Prostatectomy | 0.03 ng/mL        | RP            |
| EBRT                  | 2.0 ng/mL         | EBRT          |
| Brachytherapy         | 0.5 ng/mL         | Brachy        |
| Primary ADT           | 4.0 ng/mL         | ADT           |
| Intermittent Hormones | 10.0 ng/mL        | Intermittent  |
| Watchful Waiting      | 30.0 ng/mL        | Watchful      |
| Active Surveillance   | 15.0 ng/mL        | Active surveill. |

---

## Docs

- **Scope and screens:** [docs/WIREFRAME_POC_SPEC.md](docs/WIREFRAME_POC_SPEC.md)
- **Workflow and states:** [docs/CLINICAL_WORKFLOW_BASE_PLAN.md](docs/CLINICAL_WORKFLOW_BASE_PLAN.md)
- **Current implementation:** [docs/PROTOTYPE_STATE.md](docs/PROTOTYPE_STATE.md)
- **State machine:** [docs/PATIENT_CARD_STATE_MACHINE.md](docs/PATIENT_CARD_STATE_MACHINE.md)

---

## Tech

Single-file HTML with embedded CSS and JavaScript. No build step or dependencies.
