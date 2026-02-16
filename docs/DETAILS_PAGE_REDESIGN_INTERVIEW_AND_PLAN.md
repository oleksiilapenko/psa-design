# Details Page Redesign — Interview & Implementation Plan

**Status:** Draft for discussion.  
**Reference:** New layout/design (image) and your written summary. This doc uses the design as reference for data, placement, and layout only — we aim for the gist, not final pixel-perfect UI.

---

## 1. Current state (summary)

| Area | Current behaviour |
|------|-------------------|
| **Detail view** | Patient → Pathway (Diagnosis, Protocol, Phase, anchor, Next due, **always-visible** protocol table with Phase/Threshold/Schedule/MOT/LFT, guideline text, threshold edit, 5-ARI) → PSA (Trend sparkline, Latest result, Add result form when awaiting/action) → Required action → Actions / Discharge. |
| **Cards (board/list)** | Primary tag = **Protocol** (e.g. RP). Secondary tag = **Phase** (e.g. "Year 1"). No ownership. |
| **Pathway** | Protocol table is the main content; "current year" row is highlighted. Year is derived from anchor date and shown as Phase. |
| **Monitoring** | Derived from protocol + phase (and optional per-phase overrides in `scheduleRows`). No single "patient monitoring setting" section; effective schedule is shown in the protocol table. MOT/LFT are per-phase in the table. |
| **PSA results** | `psaHistory` = array of **values only** (numbers). `psaDate` = date of the **latest** result only. No table of dated results; sparkline + "Latest result" + add-result form. |
| **Data model** | `phase`, `frequency`, `scheduleRows`, `scheduleByYear`, `protocol`, `threshold`, `psaHistory[]`, `psaDate` (single). No `ownership`. |

---

## 2. Interview — Clarifications & decisions

### 2.1 Ownership (GP vs Urology)

- **Intent:** "Who has care of the patient at the moment" (GP or Urology).
- **Questions:**
  1. Is ownership **mandatory** (always set) or can it be "unknown" / not set for legacy or partial records?
  2. Should we support **filtering** the board/list by ownership (e.g. "Show only GP-held" / "Urology-held") in this iteration, or is ownership for display and detail-only for now?
  3. When a patient is moved to **Recall**, does ownership typically switch to Urology, or is it edited manually and can stay GP?
  4. Any need to **audit** ownership changes (who changed it, when), or is a simple current value enough for the prototype?

**Proposed data:** `ownership: 'GP' | 'Urology'` (and optionally `''` or `'Unknown'` if you allow unset). Default for new patients: confirm (e.g. GP).

---

### 2.2 Pathway: protocol summary and year only for guideline highlight

- **Intent:** Pathway section = selected protocol (pathway) + ownership + diagnosis + date of surgery/treatment. The **year** is still calculated from the anchor date but **only** used to highlight the current row in the **Guidelines** table. Guidelines live in a **collapsible** section.
- **Questions:**
  1. Should we keep showing "Year N" **somewhere** in the Pathway block (e.g. next to the treatment date) for quick reference, or is it only inside the collapsible Guidelines table?
  2. "Pathway" in the design is a **dropdown** (e.g. Radical Prostatectomy). Today we have Protocol as read-only text on detail. Confirm: Pathway = Protocol selection, and on the detail page it can stay read-only for the prototype (or do you want it editable on detail too)?
  3. Guidelines table columns: you mentioned "Phase, Monitoring schedule, MOT, LFT". Our current table has Phase, **Threshold**, Schedule, MOT, LFT. Should the Guidelines table in the collapsible **omit** Threshold (and keep threshold in the separate Threshold section), or should we show a simplified "Phase | Schedule | MOT | LFT" only?

**Proposed:** Pathway section = Pathway (protocol name, read-only for now) + Ownership (dropdown) + Diagnosis (existing) + Date of surgery/treatment (existing anchor) + optional "Year N" badge. New **collapsible "Guidelines"** containing the phase/schedule/MOT/LFT table with current-year highlight. Threshold stays in its own section.

---

### 2.3 Monitoring: single effective setting + MOT/LFT + Schedule as card tag

- **Intent:** One **effective** monitoring setting per patient (e.g. "Every 6 months"). MOT and LFT as requirements with tooltips. This **schedule value** becomes the **secondary tag on patient cards** (replacing "Year N").
- **Questions:**
  1. **Source of truth:** Today effective schedule is **derived** from protocol + phase (and optional `scheduleRows` overrides). Do you want the detail page to show a **single dropdown** "PSA test schedule" that **overrides** that derivation when set (i.e. "what this patient is actually on"), and when we change it we update the effective schedule (and possibly persist a patient-level override)? Or should it remain derived from protocol/phase/scheduleRows and only **display** the effective value (no separate "monitoring" edit)?
  2. If it’s **editable:** Changing "PSA test schedule" on detail — does that update only the **current** phase in the background (e.g. one row in `scheduleRows`) or a new top-level field like `effectiveSchedule` that overrides derivation everywhere (cards, next-due, etc.)?
  3. **Review for discharge in X years:** New field in the design. Is this (a) derived from protocol (e.g. "discharge after 10 years") or (b) a **patient-level** setting (e.g. "review for discharge in 10 years")? If (b), do we need to store it (e.g. `reviewForDischargeYears`) and use it for reminders/display?
  4. **MOT/LFT at monitoring level:** Design shows "And require: MOT, LFT" checkboxes. Currently MOT/LFT are **per-phase** in `scheduleRows`. Should we (a) keep per-phase MOT/LFT in the Guidelines table only and add **patient-level** "Require MOT/LFT" checkboxes in Monitoring that represent "current requirement" (derived from current phase or overridable), or (b) move to **only** patient-level MOT/LFT and drop per-phase?
  5. **Tooltips for MOT/LFT:** You asked for tooltips with "requirements on hover". Do you have the exact wording (e.g. "Monitoring Other Test: [clinical definition]" and "Liver Function Test: [when required]") or should we use placeholder text and mark for clinical copy later?

**Proposed (minimal):**  
- Add a **Monitoring** section: "PSA test schedule" (dropdown: 3‑monthly, 4‑monthly, 6‑monthly, Annual) showing/editing the **effective** schedule; optional "Review for discharge in" (number + years) if you want it stored.  
- "And require: MOT / LFT" checkboxes with tooltips (we can use existing titles + short requirement text).  
- Cards: secondary chip = **Schedule** (e.g. "6‑monthly") instead of Phase ("Year 1"). List view: same — show schedule instead of year in the relevant column.  
- Data: keep `frequency` as the effective schedule; optionally add `reviewForDischargeYears`. MOT/LFT: either derive from current phase from `scheduleRows` for display and add optional patient-level overrides, or keep per-phase only and show "current" in Monitoring (to be confirmed by you).

---

### 2.4 Threshold

- **Intent:** Same concept, slightly restructured layout.
- **Questions:**
  1. Any change to **when** threshold can be edited (e.g. locked when status = Recall)?
  2. Should the Threshold section stay **below** Monitoring (as in the design) or remain where it is (below Pathway)?

**Proposed:** Keep current threshold logic; only adjust layout (e.g. threshold value + unit + Save, 5-ARI below with hint). No locking for this phase unless you specify.

---

### 2.5 PSA Results: table, add-result form, edit last result

- **Intent:** New **table** of PSA results (date + value). "Add new result" form that **collapses to a button** when closed. **Edit last result** (conditional, e.g. locked for Recall).
- **Questions:**
  1. **Data model:** We currently have `psaHistory: number[]` and `psaDate` (latest only). To support a table we need **per-result dates**. Confirm structure: `psaResults: { date: string, value: number }[]` (or keep `psaHistory` and add `psaDates: string[]` parallel array)? Prefer one canonical list of `{ date, value }` for clarity.
  2. **Edit last result:** Which states **allow** editing the last result (e.g. Upcoming, Awaiting, Review Required, Completed?) and which **lock** it (e.g. Recall, Discharged, Follow-up Required)? Or is it "always allow edit" for prototype?
  3. **Add new result — collapse to button:** When collapsed, should the button say "Add new result" and expand to the full form on click, with a "Close" to collapse again (as in the design)?
  4. **Chart:** You said "simplifying the chart is hard to implement". For this iteration, do we (a) leave the current sparkline as-is, (b) remove it and rely on table only, or (c) keep a minimal chart (e.g. same sparkline, no extra features)?

**Proposed:**  
- Add `psaResults: Array<{ date: string, value: number }>` (ISO date string). Migrate existing `psaHistory` + `psaDate` into `psaResults` on load where possible; keep `psa` and `psaDate` as derived from last entry for backward compatibility.  
- PSA Results section: **Table** (columns: Date of result, relative time, PSA ng/mL; optional status icon). **Add new result**: form that can be collapsed to a single "Add new result" button; when expanded, show Date + PSA + Save and "Close". **Edit last result**: pencil on the last row; show inline edit or small modal; **locked** when e.g. `status === 'recall'` or `status === 'discharged'` (you confirm rules).  
- Chart: leave current sparkline unchanged for this phase unless you choose (b) or (c).

---

### 2.6 Wizard (Add Patient) impact

- **Questions:**
  1. Should **Ownership** be captured in the wizard (e.g. Step 1 or 2), and default to GP?
  2. Should the wizard still show **Phase/Year** in Step 3 (schedule table) for context, and we just don’t show Year on the card after add?
  3. **Review for discharge in:** If we add it, should it be in the wizard (e.g. Step 3) or only on the detail page?

**Proposed:** Add Ownership dropdown in wizard (e.g. Step 2 with protocol); default GP. Keep phase/year in wizard for schedule/guideline context. "Review for discharge" only on detail for now unless you want it in wizard.

---

## 3. Implementation plan (phased)

### Phase 1 — Data model & ownership

- Add `ownership: 'GP' | 'Urology'` to patient model; default `'GP'` for new patients.
- Optionally add `reviewForDischargeYears` (number | undefined) and `psaResults: { date, value }[]` with migration from `psaHistory` + `psaDate`.
- Ensure a single **effective schedule** is available everywhere (keep `frequency`; ensure it’s set from protocol/phase or override).

**Risks:** List/board and detail currently assume no ownership; all places that create or display patients need the new field.

---

### Phase 2 — Detail: Pathway + Guidelines collapsible

- Restructure **Pathway** block: Pathway (protocol name), **Ownership** (dropdown), Diagnosis, Date of surgery/treatment, optional "Year N" badge.
- Move the current protocol table (Phase, Schedule, MOT, LFT; optionally drop Threshold from this table if threshold lives in its own section) into a **collapsible "Guidelines"** section; keep current-year row highlight.
- Keep guideline text inside the collapsible or directly under it.

**Risks:** Detail HTML and `openDetail()` are large; collapsible must be accessible (e.g. `<details>`).

---

### Phase 3 — Detail: Monitoring section

- Add **Monitoring** section: "PSA test schedule" (dropdown), optional "Review for discharge in X years", and "Require MOT / LFT" checkboxes with tooltips.
- Wire "PSA test schedule" to effective schedule (read from `frequency`; on save, update `frequency` and optionally one row in `scheduleRows` or a dedicated override).
- Add tooltips for MOT/LFT (e.g. title + short requirement text).

**Risks:** Clarification needed on whether schedule is editable and how it interacts with `scheduleRows` and derivation.

---

### Phase 4 — Cards & list: Schedule as secondary tag

- Replace **Phase** ("Year N") with **Schedule** (e.g. "6‑monthly") on board cards and in list table.
- Ensure `frequency` is populated for all patients (from protocol/phase or overrides); use it for the new chip/column.
- Keep Phase available in the detail view (Pathway / Guidelines).

**Risks:** Filter/sort by "phase" may need to be renamed or duplicated (e.g. "Schedule" filter); existing copy that says "Year" on cards must be updated.

---

### Phase 5 — Detail: Threshold layout

- Reflow Threshold section to match design (threshold input + unit + Save; 5-ARI + hint below). No logic change unless you ask for locking by status.

---

### Phase 6 — PSA Results: table + add form + edit last

- **Table:** New table built from `psaResults` (or migrated `psaHistory`/`psaDate`). Columns: Date of result, relative time, PSA ng/mL; optional icon for auto-completed/reviewed.
- **Add new result:** Existing form; add collapse behaviour: when collapsed show "Add new result" button; when expanded show form + "Close" to collapse.
- **Edit last result:** Pencil on last row; conditionally disabled (e.g. when status is Recall or Discharged). On save, update last `psaResults` entry and derived `psa` / `psaDate` / `psaHistory` for sparkline.
- **Data:** Persist `psaResults`; derive `psa`, `psaDate`, `psaHistory` from it for backward compatibility and chart.

**Risks:** Edit-last and add-new must keep `psaHistory` and sparkline in sync; migration of existing data to `psaResults` (dates for older results may be approximate or single "latest" date).

---

### Phase 7 — Wizard

- Add Ownership to wizard; default GP.
- Optionally add "Review for discharge in" if we store it.
- No change to card tag at add-time (card will show Schedule once Phase 4 is in).

---

## 4. Caveats & open points

- **Backward compatibility:** Existing `detailData` has no `ownership` and no `psaResults`. We need defaults (e.g. ownership `'GP'`) and migration or derivation for `psaResults` from `psaHistory` (dates for non-latest results may be missing or estimated).
- **List view column:** The list currently has a "Phase" column (Year N). Replacing with "Schedule" may require a header rename and ensuring sort/filter still make sense.
- **MOT/LFT tooltip copy:** Placeholder text until clinical copy is provided.
- **Chart:** Left as-is unless you choose to simplify or remove.
- **Edit last result rules:** Locking by status (Recall, Discharged, etc.) to be confirmed.

---

## 5. Next steps

1. **You:** Answer or confirm the questions in §2 (and any subset you care about first).
2. **Implementation:** Proceed phase by phase; Phase 1 (data + ownership) and Phase 4 (card tag) are high impact, so clarity on Schedule vs Year and ownership behaviour is needed early.
3. **Design:** Treat the reference design as layout/data guide; finalise copy and exact locking rules with clinical input.

If you tell me your decisions on the interview questions (or "go with the proposed" where marked), I can turn this into a concrete task list and start with Phase 1 + 2 in code.
