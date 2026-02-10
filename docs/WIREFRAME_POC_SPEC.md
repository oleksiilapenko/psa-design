# Wireframe / POC Spec: NHS Prostate Cancer Follow-Up Dashboard

**Version:** 1.1  
**Date:** February 2025  
**Purpose:** Define screens and flows for the UX prototype. All data is mocked; logic is client-side only.

---

## Scope summary

| In scope | Out of scope (for now) |
|----------|------------------------|
| Add Patient wizard (3 steps, mock PAS) | Archive / Discharged patients list (tab only) |
| Kanban board (5 columns: Upcoming, Awaiting Results, Completed, Review Required, Action Required) | Blood forms / reminder generation (Send reminder = toast) |
| List view (All + Pathway + GP focus + Exit tabs; Recall → … → Discharged) | Role-specific views (single XP) |
| Card front (sparkline, PSA vs threshold, Δ on Review Required / Action Required) | Real PAS/Lab integration |
| Protocols: RP, EBRT, Brachy, ADT, Intermittent hormones, Watchful waiting, Active Surveillance | Consecutive rises / bounce automation |
| Clinical context: **Diagnoses**, **5‑ARI** (wizard + detail) | Tablet/mobile (desktop first) |
| Δ vs threshold (list + detail); threshold override (*) | |
| Actions: 3-dot menu + detail Actions section + Discharge subsection (no archive UI) | |
| New patient lands in **Upcoming** | |
| Completed: copy says 7 days then auto to Recall (no timer in POC) | |

---

## User flows to illustrate

1. **Add new patient**  
   Add Patient → Step 1 (NHS No, mock fetch) → Step 2 (**Diagnoses** optional, **Protocol** as radios with help text, anchor date) → Step 3 (**5‑ARI**, protocol default + personal threshold, last PSA + date) → **Add patient** → Patient appears in Upcoming.

2. **Move through board**  
   Upcoming → Awaiting Results (Move to Awaiting Results) → Completed (normal) or Review Required (abnormal); Completed cards copy: 7 days then auto to Recall (no timer). Review Required → Move to Action Required (with required action) or Move to Recall. Discharge available in menu (Review Required, Action Required) and in detail Discharge subsection (all non-discharged). Recall and Discharged not on board.

3. **Switch views**  
   Toggle between **Kanban** and **List**; same data. List: All (separate) | Pathway: Recall, Upcoming, Awaiting, Completed | GP focus: Review Required, Action Required | Exit: Discharged.

4. **Card detail**  
   Click card or list row → full context: **Clinical context** (Diagnoses, 5‑ARI), pathway, **historical PSA** chart, Latest PSA with **Δ vs threshold**, threshold (with * if override). **Actions** section: status-driven action cards (e.g. Move to Completed, Move to Recall). **Discharge** subsection below (red accent card); syncs board/list/detail.

---

## Screens (as implemented)

### 1. Dashboard – Kanban view (primary)

- **Header:** Title, view toggle (Kanban | List), **Close** (wizard) next to “Add New Patient”.
- **Five columns:** Upcoming, Awaiting Results, Completed, Review Required, Action Required. Recall and Discharged are not on the board.
- **Cards:** Name (Lastname, Firstname, age), NHS No, protocol + phase badge, sparkline, latest PSA vs threshold. **Review Required** and **Action Required** cards show **Δ vs threshold**. 3-dot menu per card (View details, Move to …, Discharge where applicable). * indicates threshold override (see legend in List).
- **Empty states:** Placeholder text per column where needed.

### 2. Dashboard – List view

- **Same header** as Kanban.
- **Tabs:** All (separate) | Pathway: Recall, Upcoming, Awaiting, Completed | GP focus: Review Required, Action Required | Exit: Discharged.
- **Table columns:** Patient (Lastname, Firstname, age), NHS No, Status, Protocol (short label), Phase, Diagnoses, Trend (sparkline), Last PSA (ng/mL), Threshold (* if override), **Δ** (↑/↓ signed or —), Next Due, Last updated.
- **Legend:** “Δ = PSA − threshold. * Override of protocol default.”
- **Row actions:** Click row opens detail modal.

### 3. Add Patient – Step 1: Patient identification

- NHS Number → Search / Fetch → mock: success with Name, DOB, NHS No (pre-filled).
- Continue to Step 2.

### 4. Add Patient – Step 2: Protocol anchoring

- **Diagnoses (optional):** Free text; used for protocol and triage.
- **Protocol:** **Radio buttons** — Radical Prostatectomy, EBRT, Brachytherapy. Each shows **help text** (triage / threshold interpretation).
- **Anchor date:** Date of surgery / treatment start.
- **System output:** Current phase (e.g. “Year 2 — 6‑monthly”) — read-only.
- **Back** (to Step 1), **Continue** (to Step 3).

### 5. Add Patient – Step 3: Risk & safety net

- **5‑alpha reductase inhibitor:** None | Finasteride | Dutasteride. Hint: threshold interpretation accounts for 5‑ARI.
- **Protocol default threshold:** Read-only (0.03 / 2.0 / 0.5 per protocol).
- **Personal threshold:** Editable; defaults to protocol. If changed, * indicates override (no “(personal)” wording).
- **Historical PSA:** Last known PSA + date (for sparkline baseline).
- **Back** (to Step 2), **Add patient** (→ lands in Upcoming; toast). No Cancel on this step; use Close in header to discard.

### 6. Card structure (front)

- **Header:** Name (Lastname, Firstname, age), NHS Number.
- **Badge:** Protocol (short) + Phase (e.g. “Radical Prost. | Year 2”).
- **Sparkline:** Last 3–5 PSA values (or 1 for new patient).
- **Latest result:** PSA vs threshold; **Review Required** and **Action Required** columns: **Δ vs threshold** (PSA − threshold).
- **Override:** * when personal threshold differs from protocol (no “personal” label on card).
- **Context:** In Review Required / Action Required, reason (e.g. “Above threshold”) and Δ.

### 7. Card / patient detail

- **Patient:** Name, NHS No, DOB, protocol + phase chips, status.
- **Clinical context:** **Diagnoses** (editable), **5‑alpha reductase inhibitor** (None / Finasteride / Dutasteride, editable). When on 5‑ARI, note under PSA: “Threshold interpretation accounts for 5‑ARI.”
- **Pathway:** Protocol, phase, frequency, anchor.
- **PSA & threshold:** **Historical PSA** chart (labels under bars), Latest PSA (ng/mL) with **Δ vs threshold**, protocol threshold, personal threshold (editable; * if override). Enter/edit result for Awaiting or Review Required.
- **Key dates:** Next due, last bleed (no Discharge button here).
- **Actions:** Status-driven **action cards** (e.g. Move to Awaiting Results, Move to Completed, Move to Review Required, Move to Action Required, Move to Recall) with short descriptions. **Discharge** in a separate subsection below (one card, red left accent); available for all non-discharged patients. All moves sync board, list, and detail.

### 8. Recall and Discharge

- **Recall** and **Discharged** patients are not on the board; they appear in the List only. From **Review Required** or **Action Required:** Move to Recall (card leaves board) or Discharge (card removed, status → discharged). Discharge subsection in detail for any non-discharged status. No archive list in POC.

---

## Protocol rules (for labels and copy)

- **Radical Prostatectomy:**  
  Year 1: 4‑monthly; Yr 2–5: 6‑monthly; Yr 6–10: annual. Normal &lt; 0.03 ng/mL; Review &gt; 0.03; Recall &gt; 0.1 or 2 consecutive rises (hint only in POC).

- **EBRT:**  
  Yr 1–5: 6‑monthly; Yr 6–10: annual. Normal &lt; 2.0 ng/mL (Phoenix); Review &gt; 2.0; Recall consider bounce (hint only in POC).

- **Brachytherapy:**  
  Yr 1–5: 6‑monthly; Yr 6–10: annual. Normal &lt; 0.5 ng/mL (confirm local policy); Review &gt; 0.5; rising PSA may need imaging/referral.

---

## Implementation notes (prototype evolution)

The live prototype (`index.html`) has evolved from the original wireframe as follows:

- **Naming:** All displays use “Lastname, Firstname (age)”.
- **Wizard:** Close in header; Back on steps 2–3; primary button “Continue” (step 3: “Add patient” only). Step 2 uses **radio** protocol selection with **help text**; Step 2 includes optional **Diagnoses**; Step 3 includes **5‑ARI** and default threshold from selected protocol.
- **Threshold:** No “(personal)” wording; **asterisk** when overridden; legend “* Override of protocol default”.
- **Delta (Δ):** Single meaning **PSA − threshold**. Shown in list column “Δ vs threshold” (signed; — when no result) and in detail next to Latest PSA; on **Review Required** and **Action Required** cards. Units ng/mL in list header and detail; not on Kanban cards.
- **Clinical context:** **Diagnoses** and **5‑ARI** in wizard (Step 2 / Step 3) and in detail under “Clinical context”; both editable in detail. Data: `diagnoses` (string), `medication5ARI` ('none'|'finasteride'|'dutasteride').
- **Protocol display:** Short labels (Radical Prost., EBRT, Brachy.) via `protocolShortLabel`; protocol config holds `threshold`, `phase`, `help` per protocol for wizard and defaults.
- **List tabs:** All (separate group); Pathway: Recall, Upcoming, Awaiting, Completed; GP focus: Review Required, Action Required; Exit: Discharged.
- **PSA label on board cards:** Only the first two columns (Upcoming, Awaiting Results) use **“Last PSA”** (no new result in the cycle yet). Review Required, Completed, and Recall use **“PSA”** plus the value that triggered the move (e.g. “PSA 0.02 / 0.03 ✓”), so the label reflects the new result in the cycle. Avoid “PSA 0.0”; use actual value (e.g. 0.02).

For workflow and protocol decisions, see [CLINICAL_WORKFLOW_BASE_PLAN.md](CLINICAL_WORKFLOW_BASE_PLAN.md).

---

## Design notes

- **Reference:** NHS Digital Design System (e.g. [service-manual.nhs.uk](https://service-manual.nhs.uk/design-system)) for components and accessibility.
- **Desktop first;** no tablet/mobile wireframes in this POC.
- **Mock data:** All PAS and lab data simulated; no real integration.

---

## Deliverables

- Implemented prototype: single-file `index.html` (screens 1–7 and Recall/Discharge as in §8).
- Flows: Add Patient, Kanban movement, View toggle, card detail, Recall/Discharge.
- Documentation: this spec, [CLINICAL_WORKFLOW_BASE_PLAN.md](CLINICAL_WORKFLOW_BASE_PLAN.md), [PROTOTYPE_STATE.md](PROTOTYPE_STATE.md).
