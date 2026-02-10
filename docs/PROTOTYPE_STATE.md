# Current prototype state

**Purpose:** Snapshot of what is implemented in the live prototype (`index.html`) as of the latest iteration. Use for walkthroughs and to align docs.

---

## 1. Board (Kanban)

- **Columns (5):** Upcoming → Awaiting Results → Completed → Review Required → Action Required. Recall and Discharged are **not** on the board.
- **Cards:** NHS No, name • age, protocol + phase, sparkline, latest PSA vs threshold, next due / context. Review Required and Action Required show Δ vs threshold. Each card has 3-dot menu; click (outside menu) opens detail.
- **Column copy:** Upcoming — “PSA due next 4 weeks… Move to Awaiting Results when test done.” Completed — “All completed cards appear here. They automatically move to Recall after 7 days.” (No 7-day timer in code.)

---

## 2. List view

- **Tabs:** **All** (separate group) | **Pathway:** Recall, Upcoming, Awaiting, Completed | **GP focus:** Review Required, Action Required | **Exit:** Discharged. Default tab: Recall.
- **Table:** Patient, NHS No, Status, Protocol, Phase, Diagnoses, Trend (sparkline), Last PSA, Threshold, Δ, Next Due, Last updated. Row click opens detail. Status badges and counts per tab.

---

## 3. Patient detail

- **Sections (numbered, larger headings):** **1. Diagnosis** (editable). **2. Pathway** — protocol, phase, treatment start; **protocol table** (Year 1–10, schedule per year, current year highlighted); **PSA guidelines** (e.g. PSA &lt; 0.03 normal; review if &gt; 0.03; recall if &gt; 0.1 or 2 consecutive rises); Next test due and Last bleed. **3. PSA & threshold** — chart (100% width), latest PSA, threshold (editable), result form when applicable; **On 5-ARI?** (Yes/No + which drug, editable). **4. Actions** — status-driven action cards.
- **Context tags:** Protocol, Phase, and **5-ARI** (shown when patient is on a 5-ARI) as chips in the Patient block.
- **Actions:** Section “4. Actions” with **action cards** (left-accent border). Cards vary by status. **Discharge** in a separate subsection below (one card, red left accent; all non-discharged).

**Add Patient wizard:** Step 1 — Find patient (NHS number, mock search). Step 2 — Pathway: diagnoses (optional), protocol radios, treatment start; **protocol table** (Year 1–10: protocol default + override dropdown per year; current year bold). Step 3 — **On 5-ARI?** Yes/No (checkbox); if Yes, “Which one?” (Finasteride/Dutasteride); protocol threshold and personal threshold; last known PSA and date. See [WIZARD_AND_DETAIL_REDESIGN.md](WIZARD_AND_DETAIL_REDESIGN.md).

---

## 4. 3-dot menu (board cards)

- **View details** on every card (opens detail).
- **Upcoming:** Move to Awaiting Results, Send reminder (toast).
- **Awaiting Results:** Move to Completed, Move to Review Required.
- **Review Required:** Move to Completed, Move to Action Required, Move to Recall, Discharge.
- **Action Required:** Move to Recall, Discharge.
- **Completed:** Move to Recall.
- Recall/Discharged: not on board; from list, detail shows Discharge only (or no workflow actions for Recall).

All “Move to…” actions update `detailData`, list row (data-tab, status badge), and board card position. Move to Recall or Discharge removes the card from the board.

---

## 5. Data and sync

- **Source of truth:** `detailData[id]` plus each card’s `data-status` and each list row’s `data-tab` / `data-status`. Moves from 3-dot menu or detail action cards update all three so board, list, and detail stay in sync.
- **Status values:** `upcoming`, `awaiting`, `action` (Review Required), `completed`, `action_required`, `recall`, `discharged`. Display labels use title case (e.g. Action Required, Review Required).
- **Mock only:** No backend; no 7-day auto-move; no next-due calculation. Send reminder and any future “blood form” type actions are toast only.

---

## 6. Open questions (unchanged)

- Definition of “result in cycle”; late/overdue results; correction and undo.
- 5‑ARI effect on threshold; 2 consecutive rises / bounce; protocol rules as system behaviour.
- Discharge meaning and archive/audit; blood forms and reminders integration.
- Safety nets (e.g. stuck in Awaiting, no result when marking completed); reason codes for override/recall/discharge.

---

## 7. Actions reference (summary)

**3-dot menu:** View details on every card. Upcoming: Move to Awaiting Results, Send reminder (toast). Awaiting Results: Move to Completed, Move to Review Required. Review Required: Move to Completed, Move to Action Required, Move to Recall, Discharge. Action Required: Move to Recall, Discharge. Completed: Move to Recall.

**Detail — Actions section:** Status-driven cards (e.g. Upcoming: Move to Awaiting Results; Awaiting: Move to Completed + Move to Review Required with short descriptions; Review Required: Move to Completed, Move to Action Required, Move to Recall; Completed: Move to Recall; Action Required: Move to Recall; Recall: none).

**Detail — Discharge subsection:** Below Actions, no title. One card: “Discharge”, description “Remove from the active monitoring list. Record kept for audit.” Red left accent. Shown for all non-discharged statuses. Behaviour: set status to discharged, update list, remove card from board if present, close modal.

**Naming:** “View details” (not “View / edit patient”). All moves: “Move to &lt;Column&gt;” with title case (Awaiting Results, Completed, Review Required, Action Required, Recall). **Removed:** Generate blood form, Chase result, “Discharge from list” button in Key dates. **Mock:** Send reminder (toast).

---

See [CLINICAL_WORKFLOW_BASE_PLAN.md](CLINICAL_WORKFLOW_BASE_PLAN.md) for the full workflow spec and decisions.

**Doc alignment:** List table (including Diagnoses column) and Δ on Review Required + Action Required match `index.html`. [WIREFRAME_POC_SPEC.md](WIREFRAME_POC_SPEC.md) and [PATIENT_CARD_STATE_MACHINE.md](PATIENT_CARD_STATE_MACHINE.md) were updated to match the prototype.
