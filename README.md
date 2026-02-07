# NHS Prostate Cancer Follow-Up Dashboard — Prototype

HTML prototype for the Pan Dorset follow-up pathways. **Single-file front-end** with mock data and client-side logic; no backend.

## Run

Open in a browser:

```bash
open index.html
```

Or double-click `index.html`. No server required.

---

## What’s in the prototype

### Dashboard

- **Kanban:** 5 columns — Upcoming, Awaiting Results, Review Required, Completed, Recall. Cards show name (Lastname, Firstname), NHS No, protocol + phase, sparkline, latest PSA vs threshold, next due. Review Required cards show Δ vs threshold; Close button aligned with “Add New Patient” in header.
- **List:** Tabs — Upcoming | Awaiting | Action Required | Completed | Recall | All. Table columns: Patient, Status, NHS No, Protocol, Phase, Last PSA (ng/mL), Threshold, **Δ vs threshold** (signed difference: ↑ above, ↓ below), Trend (sparkline), Next Due. Legend: Δ = PSA − threshold; * = override of protocol default.
- **View toggle:** Kanban ⇄ List. Same data in both views.

### Add Patient wizard (3 steps)

- **Step 1 — Patient identification:** NHS Number → Search → mock PAS result (name, DOB, NHS No) → Continue.
- **Step 2 — Protocol anchoring:**
  - **Diagnoses (optional):** Free text; used for protocol and triage; can be linked from active investigations later.
  - **Protocol:** Radio buttons — Radical Prostatectomy, EBRT (external beam radiotherapy), Brachytherapy. Selecting one shows phase (e.g. Year 2 — 6‑monthly) and **protocol-specific help text** for triage (PSA vs threshold, Phoenix, etc.).
  - Anchor date (surgery / treatment start) → Continue.
- **Step 3 — Threshold & baseline:**
  - **5‑alpha reductase inhibitor:** None | Finasteride | Dutasteride. Hint: PSA is often lower on 5‑ARI; threshold interpretation will account for this.
  - Protocol default threshold (read-only), personal threshold (editable, override if needed).
  - Last known PSA + date (for trend).
  - Back (to step 2) and **Add patient** (toast; no backend).

### Patient detail (opened card)

Opened by clicking a card or list row. Sections:

- **Patient:** Name, NHS No, DOB, protocol + phase chips, status badge.
- **Clinical context:** **Diagnoses** and **5‑alpha reductase inhibitor** (None / Finasteride / Dutasteride). Both editable (Edit → change → Save). When on 5‑ARI, PSA section shows: “Threshold interpretation accounts for 5‑ARI.”
- **Pathway:** Protocol, phase, monitoring frequency, treatment start.
- **PSA & threshold:** Chart with **historical PSA values** (labels under bars), Latest PSA (with ng/mL and **Δ vs threshold**), protocol threshold, personal threshold (editable; * if overridden). For Awaiting/Review Required, inline form to enter or edit PSA result and date.
- **Key dates:** Next test due, last bleed (if relevant).
- **Actions:** Status-driven (e.g. Mark as completed, Move to Recall, Discharge). Toasts only; no archive.

### Conventions in the UI

- **Names:** Shown as *Lastname, Firstname (age)* everywhere (cards, list, detail, wizard).
- **Δ (delta):** PSA − threshold. Positive = above threshold (action/recall); negative = below (normal). Shown in list “Δ vs threshold” column and next to Latest PSA in detail. Not shown on Kanban cards except Review Required (Δ on card for that column only).
- **Threshold override:** Personal threshold different from protocol is indicated by * and legend “* Override of protocol default” (list and detail).
- **Units:** ng/mL used where there is space (detail, list header); omitted on Kanban cards.

---

## Protocols (current)

| Protocol                 | Default threshold | Short label   | Help text in wizard (triage) |
|--------------------------|-------------------|---------------|------------------------------|
| Radical Prostatectomy    | 0.03 ng/mL        | Radical Prost.| Biochemical remission; above → MDT/recall |
| EBRT                     | 2.0 ng/mL         | EBRT          | Phoenix; nadir + 2           |
| Brachytherapy            | 0.5 ng/mL         | Brachy.       | Confirm local policy; rising → imaging/referral |

Adding a new protocol is a matter of extending the `protocolConfig` object and the wizard radio options (see docs below).

---

## Spec and next steps

- **Scope and screens:** [docs/WIREFRAME_POC_SPEC.md](docs/WIREFRAME_POC_SPEC.md)
- **Refinement and protocol extension:** [docs/NEXT_STEPS.md](docs/NEXT_STEPS.md)

---

## Tech

Single-file HTML with embedded CSS and JavaScript. No build step or dependencies.
