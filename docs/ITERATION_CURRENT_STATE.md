# Iteration view & current state

**Date:** February 2025  
**Purpose:** Snapshot of this design iteration: what’s implemented, what’s decided, and the main open questions from a clinical and product perspective.

---

## 1. Current state of the prototype

### 1.1 Board (Kanban)

- **Columns:** Upcoming → Awaiting Results → Review Required → Completed → Recall.
- **Cards:** NHS No, Name • Age, protocol + phase chips, **PSA line**, sparkline, due/context line, last updated. Review Required cards show Δ vs threshold and optional inline “Move to Recall”. Recall cards have no suggested action on the card (Discharge only in 3-dot menu and detail).
- **PSA label rule (board):**
  - **Upcoming, Awaiting Results:** “Last PSA” &lt;value&gt; / threshold (or —). No new result in the current cycle.
  - **Review Required, Completed, Recall:** “PSA” &lt;value&gt; / threshold (the result that triggered the column move). Avoids “Last PSA” where it would imply no new result (e.g. “PSA 0.0” would be misleading; use “PSA 0.02” etc.).
- **Actions:** 3-dot menu per card. Single “View / edit patient” (no separate “Edit details”). Status-specific actions (blood form, remind, mark completed, move to Recall, print summary, Discharge). Discharge available in menu and in detail Actions, not as a suggested button on Recall cards.

### 1.2 List view

- **Tabs:** Upcoming | Awaiting | Action Required | Completed | Recall | All.
- **Columns (order):** Patient, NHS No, Status, Protocol, Phase, Trend (sparkline), Last PSA (ng/mL), Threshold, Δ, Next Due, Last updated.
- **Status column** always visible. Numeric columns (Last PSA, Threshold, Δ) right-aligned.
- **Table** borders 1px, soft grey; selected tab matches table header colour (#e0e0e0). Tab labels bold.

### 1.3 Add Patient wizard

- Step 1: NHS No → mock fetch → Name, DOB.
- Step 2: Diagnoses (optional), Protocol (radios + help), anchor date → phase/frequency read-only.
- Step 3: 5‑ARI, protocol default threshold, personal threshold (override → *), last known PSA + date → Add patient (lands in Upcoming).

### 1.4 Patient detail (card / list click)

- Single-column layout (no sidebar). Sections: Patient, Clinical context (Diagnoses, 5‑ARI), Pathway, PSA & threshold (sparkline, latest PSA, Δ, threshold edit), Key dates, Actions (status-driven: e.g. Save result, Mark completed, Move to Recall, Discharge).
- Actions and 3-dot menu share the same conceptual actions; Discharge remains available in both.

### 1.5 Protocols in scope

- **Implemented:** Radical Prostatectomy (0.03), EBRT (2.0), Brachytherapy (0.5), ADT (4.0), Intermittent Hormones (10.0), Watchful Waiting (30.0). Short labels: Radical prostatectomy, EBRT, Brachy, ADT, Intermittent hormones, Watchful waiting.
- **Threshold override:** Personal threshold editable in wizard and detail; * in UI; legend “* Override of protocol default”.

### 1.6 Data and behaviour

- All data mocked in `index.html` (detailData, protocolConfig). No backend.
- Delta (Δ) = PSA − threshold everywhere. Shown in list, detail, and on Review Required cards.
- 5‑ARI note in detail when set. Sparkline thresholds injected from detailData.

---

## 2. Clinical review: main unanswered questions

These are the main open questions that affect safety, workflow, or scope. They are not implementation bugs but product/clinical decisions the prototype does not yet resolve.

### 2.1 Result and timing

1. **Definition of “result in cycle”**  
   When exactly does a “Last PSA” become the “PSA” that drove the move (e.g. to Review Required or Completed)? Is it date-of-result vs date-of-bleed, or “first result after moving to Awaiting”? Need a clear rule for when to show “Last PSA” vs “PSA” in list/detail and for automation (e.g. 2 consecutive rises).

2. **Late results and overdue tests**  
   What should the system show when a test is very late (e.g. bleed 3+ weeks ago, no result)? Should the card stay in Awaiting, move to a separate “Overdue” state, or trigger a different action (e.g. chase)? No overdue/ageing logic in the POC.

3. **Correction and undo**  
   If a user enters the wrong PSA and has already moved the card (e.g. to Completed or Recall), how do they correct it? “Edit result” in detail exists, but there is no defined behaviour for “re-open” or “undo move” and whether that is allowed after 48h or after discharge.

### 2.2 Thresholds and protocols

4. **2 consecutive rises and “bounce”**  
   Protocol text (e.g. RP: “recall if &gt; 0.1 or 2 consecutive rises”; EBRT: “consider bounce”) is hint-only. The system does not auto-detect 2 consecutive rises or suggest “possible bounce” vs “possible recurrence”. Need a decision: support in this product or leave to clinician judgment with trend visible.

5. **5‑ARI and threshold interpretation**  
   The UI states that “threshold interpretation accounts for 5‑ARI” but does not change the numeric threshold or suggest a different action. Unclear whether 5‑ARI should (a) only be informational, (b) adjust displayed threshold, or (c) drive different recall rules. Same for other modifiers (e.g. nadir + 2 in EBRT).

6. **Protocol rules as system behaviour**  
   Normal/Review/Recall bands are in copy and config; column moves are manual (e.g. “Mark completed”, “Move to Recall”). Open question: should the system ever auto-suggest or auto-move (e.g. “Suggest move to Recall” when PSA &gt; 0.1 post-RP), and who owns the final decision?

### 2.3 Workflow and scope

7. **Completed and 48h rule**  
   Spec says “card leaves board after 48h; patient reappears in Upcoming when next due”. No 48h timer or “next due” calculation in the POC. Need rules: how “next due” is computed (from protocol + last test date), and whether “leaves board” means hide from Kanban only or also a status change.

8. **Discharge and archive**  
   Discharge is an action only (no archive list, no audit trail in the UI). Open: what “Discharge” means (e.g. off this list but still in PAS), what is stored, and whether a separate “Discharged” or “Archive” view is required for audit or re-activation.

9. **Blood forms and reminders**  
   Out of scope for this POC. Unclear whether they will be separate integrations, part of this product, or manual; and how “Generate blood form” / “Send reminder” would target the correct episode and patient.

### 2.4 Safety and safety nets

10. **Missing results and no-PSA patients**  
    One mock case (Wilson) has no PSA on record and is in Awaiting. The UI supports “—” and “No PSA results on record”. Need a policy: when to chase, when to allow moving to Review/Completed without a numeric result, and how to prevent accidental “Mark completed” with no result.

11. **Overrides and reason codes**  
    Personal threshold override is stored and shown (*). There is no structured “reason for override” or “reason for Recall/Discharge” in the prototype. For audit and MDT, it may be necessary to capture and display reason codes.

12. **Alerts and escalation**  
    No alerting or escalation (e.g. “stuck in Awaiting &gt; 14 days”, “PSA above threshold not reviewed”). Decision needed: is this a tracking board only, or should it support configurable safety nets?

---

## 3. Summary table

| Area              | Decided / in POC                                      | Open / unanswered                                                                 |
|-------------------|--------------------------------------------------------|-----------------------------------------------------------------------------------|
| PSA label         | “Last PSA” in Upcoming/Awaiting; “PSA” in Review/Completed/Recall | N/A                                                                              |
| Column moves      | Manual only (buttons / 3-dot)                         | Auto-suggest or auto-move; 48h and “next due” rules                              |
| Threshold         | Personal override; * and legend                       | 5‑ARI effect; reason for override; 2 consecutive rises / bounce logic           |
| Discharge         | Action in menu and detail; no archive UI              | Meaning, storage, audit, re-activation                                           |
| Result correction | Edit result in detail                                 | Undo move; behaviour after Completed/Recall                                       |
| Late/overdue      | Not implemented                                      | Definition and UX for late results and overdue tests                             |
| Safety nets       | None                                                  | Alerts, escalation, “no result” handling, reason codes                           |

---

## 4. View of this iteration

This iteration establishes:

- **Board and list** with consistent PSA labelling (Last PSA vs PSA by column).
- **Single “View / edit patient”** and Discharge in menu and detail only on Recall (no suggested Discharge on card).
- **List column order** (Patient, NHS No, Status, …) and visible Status.
- **Completed cards** aligned to same structure and “PSA” label as other result columns.

The prototype is suitable for **walkthroughs and flow validation** with GPs/CNS/Admin. The main gaps are **clinical and product policy** (timing, thresholds, discharge, safety nets), not only UI layout. Recommended next step: validate flows with users (see NEXT_STEPS.md) and then lock content and protocol rules (including the PSA label rule) in the spec before adding further protocols or integration.
