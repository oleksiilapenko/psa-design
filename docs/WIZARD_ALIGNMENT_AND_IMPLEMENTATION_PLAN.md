# Wizard alignment with new details page — implementation plan

**Status:** Draft.  
**Reference:** Wizard screens (Step 1–3 from design), details page redesign, and requirement: *On Step 2, if the answer to the Y/N question is Yes, autofill Diagnosis with "Localised prostate cancer" (placeholder).*

---

## 1. Screen-by-screen alignment

### 1.1 Step 1: Find the patient

| Design | Current | Alignment |
|--------|---------|-----------|
| Title: "Find the patient" | Same | OK |
| Subtitle: "Specify patient's NHS number to add them to the monitoring registry" | "We look up the patient in the NHS patient service…" | Use design copy (or keep both for prototype). |
| Label: "Search by NHS number" | "NHS Number" | Use "Search by NHS number"; keep one input. |
| Button: "Find" (with magnifying glass) | "Search" | Rename to "Find" (icon optional). |
| After search: green banner "Patient found. Double-check details before proceeding" | "Is this the correct patient?" in confirm-box | Add success banner; keep confirmation concept. |
| Patient card: NHS, Name, Gender/Age/DOB, Surgery, Address, dismiss (X) | Name, DOB, NHS only | Add optional **practice** (e.g. "Berrylands Surgery") and **address** in mock; X to clear selection. |
| Button: "Confirm and continue" | "Continue" | Rename to "Confirm and continue". |

**Data / flow:** Step 1 only identifies the patient. No pathway data yet. Mock: on "Find", show a fixed or NHS-based mock (e.g. "Williams, Steve" / "Mr. Williams, Steve", DOB, practice, address). On "Confirm and continue" → go to Step 2 and pass patient identity (name, DOB, NHS) into Step 2.

---

### 1.2 Step 2: Select pathway

| Design | Current | Alignment |
|--------|---------|-----------|
| Title: "Select pathway" | "Choose treatment pathway" | Use "Select pathway". |
| Subtitle: protocol rules + next step = monitoring schedule | Same intent | Keep. |
| **"Is prostate cancer confirmed?"** Yes / No (radios) | Not present | **Add** as first question. |
| **If Yes:** Pathway list = Radical Prostatectomy, EBRT, Brachytherapy, ADT, Intermittent hormones, Active surveillance, Watchful waiting | Current protocol list | Keep; show only when Yes. **Autofill Diagnosis** = "Localised prostate cancer" (placeholder). |
| **If No:** Pathway list = "General PSA monitoring", "Raised PSA follow-up" (with short descriptions) | Not present | **Add** two options; show only when No. Diagnosis not autofilled (leave empty or separate rule — see questions). |
| **Ownership:** "Who will manage recalls and act on results" — General Practice / Urology | Not present | **Add** radios; default e.g. General Practice. |
| **Date of surgery or start of treatment** + calendar + **Year N** badge (e.g. "Year 2") | Currently in Step 3 | **Move** to Step 2. Keep year calculation; show badge next to date. |
| Diagnosis field | "Diagnoses (optional)" free text + datalist | When **Yes**: prefill "Localised prostate cancer", optionally read-only or still editable. When **No**: leave empty (or editable); no autofill. |
| Button: "Continue" | "Continue" | Same. |

**Implementation (Y/N → Diagnosis):**

- On "Is prostate cancer confirmed?" **Yes**: set `wizard-diagnoses` value to `"Localised prostate cancer"` and optionally disable or leave editable with hint "Autofilled from pathway; you can change if needed."
- On **No**: clear diagnosis (or set to `""`) and do not autofill.
- When switching from Yes → No: clear autofill. When switching from No → Yes: set again to "Localised prostate cancer".

**Data model impact:**

- New patients get `ownership: 'GP' | 'Urology'` (from Step 2).
- New patients get `diagnoses` from wizard: either "Localised prostate cancer" (when Y) or "" / user input (when N).
- Mock `detailData`: add `ownership` to every new patient; existing mock patients can get a default (e.g. `'GP'`) for consistency with detail page.

---

### 1.3 Step 3: Set up monitoring schedule

| Design | Current | Alignment |
|--------|---------|-----------|
| Title: "Set up monitoring schedule" | "Schedule and threshold" | Use "Set up monitoring schedule". |
| **Guidelines** for [Protocol]: table Phase | Monitoring schedule | MOT | LFT, current row highlighted, guideline text below | Current protocol table has Phase, Threshold, Schedule, MOT, LFT | Show **Guidelines** table without Threshold column (Phase | Monitoring schedule | MOT | LFT); threshold in separate block. Highlight current year row. |
| **Monitoring:** "Set how often… PSA test, and when reviewed for discharge" | Per-phase schedule in table | Add **Monitoring** block: **PSA test schedule** dropdown (e.g. Every 3/4/6 month, Annually), **Review for discharge in** dropdown (e.g. 5, 7, 10 years), **And require: MOT, LFT** checkboxes. Optional "Save" or apply on Add patient. |
| **Threshold:** value + ng/mL, auto-complete vs flag for review copy | Same | Keep; add short description. **On 5-ARI** checkbox + hint below. |
| Footer: "You will be able to edit this information later." | — | Add. |
| Button: **"+ Add patient"** | Step 3: "Continue" → Step 4: "Add patient" | Design shows **Add patient** on Step 3. So either: **(A)** Merge baseline PSA into Step 3 (optional section) and single "+ Add patient" here, or **(B)** Keep Step 4 for baseline PSA and Step 3 button "Continue" (design then has 4 steps). |

**Open point:** Design has only 3 steps ending with "+ Add patient". Current flow has 4 steps (baseline PSA on Step 4). Decision needed: 3-step (baseline PSA optional in Step 3 or collected later on detail) vs 4-step (keep baseline PSA as Step 4).

---

## 2. Flow summary (with Y/N and diagnosis)

1. **Step 1:** User enters NHS number → Find → sees "Patient found" + patient card → Confirm and continue → **output:** name, DOB, NHS (and optionally practice, address for mock).
2. **Step 2:** User sees "Is prostate cancer confirmed?"  
   - **Yes** → Pathway list = cancer pathways (RP, EBRT, …). **Diagnosis autofilled** "Localised prostate cancer". User picks pathway, ownership, date of surgery/treatment; sees Year N.  
   - **No** → Pathway list = General PSA monitoring, Raised PSA follow-up. Diagnosis not autofilled (empty or editable). User picks pathway, ownership; date may still apply for "Raised PSA" or be N/A for "General" (see questions).  
   Continue → **output:** cancerConfirmed (bool), protocol/pathway, ownership, anchor date, phase year, diagnoses (autofill or not).
3. **Step 3:** Guidelines table (from pathway), Monitoring (PSA schedule, review for discharge, MOT/LFT), Threshold (+ 5-ARI). Then either "+ Add patient" (3-step) or "Continue" to Step 4 (4-step).
4. **Step 4 (if kept):** Baseline PSA (value + date) → Add patient.

**When "Add patient" runs:**

- Build patient from Step 1 + 2 + 3 (+ 4 if present).
- Set `ownership` from Step 2.
- Set `diagnoses`: if Step 2 was Yes use "Localised prostate cancer" unless user changed it; if No use wizard diagnosis field or "".
- Set `frequency`, `reviewForDischargeYears`, MOT/LFT, threshold, 5-ARI from Step 3; baseline PSA from Step 4 if present.

---

## 3. Mock data and consistency

- **New patients (from wizard):** Always have `ownership` (from Step 2). `diagnoses` = "Localised prostate cancer" when Y (or user edit), else "" or value from diagnosis field when N.
- **Existing `detailData`:** Add `ownership` to all mock patients (e.g. default `'GP'`) so detail page and any filters work without null checks.
- **List/board:** Diagnosis column already exists; it will show "Localised prostate cancer" for wizard-added cancer patients when Y was selected. No change to column structure.
- **Non-cancer pathways:** If we add "General PSA monitoring" and "Raised PSA follow-up", we need minimal `protocolConfig` (and optionally `protocolPhases`) entries so that Step 3 Guidelines and Monitoring still render (e.g. single phase "Ongoing", schedule 6‑monthly or annual, threshold optional). Define in implementation.

---

## 4. What doesn’t add up / questions

### 4.1 Step 2 — Diagnosis when "No"

- When "Is prostate cancer confirmed?" = **No**, should the Diagnosis field be (a) left empty, (b) still editable with no autofill, or (c) set to a placeholder (e.g. "Raised PSA" for "Raised PSA follow-up")? **Recommendation:** (b) empty by default, editable.

### 4.2 Step 2 — Date of treatment when "No"

- For "General PSA monitoring" or "Raised PSA follow-up", is "Date of surgery or start of treatment" still relevant? If not, we could hide or make optional and show "Year N" only when a date is entered (or N/A). **Recommendation:** Show date field for both; for non-cancer pathways we can treat it as "start of monitoring" or leave year as 1 if empty.

### 4.3 Step 3 vs Step 4 — Baseline PSA

- Design shows Step 3 ending with "+ Add patient". Do we (a) **drop Step 4** and add an optional "Baseline PSA" block in Step 3 (or collect first PSA on detail page), or (b) **keep Step 4** and use "Continue" on Step 3 so the flow stays 4 steps? **Recommendation:** (b) keep 4 steps for prototype so baseline PSA is still captured at add; we can change button label on Step 3 to "Continue" and keep "Add patient" on Step 4.

### 4.4 Non-cancer pathways — protocol rules

- "General PSA monitoring" and "Raised PSA follow-up" need at least: a label, a default schedule (e.g. 6‑monthly), and optionally a threshold and guideline text. Do you have clinical rules (e.g. threshold, discharge) or should we use placeholders (e.g. "Follow local policy")? **Recommendation:** Add two minimal protocol keys with placeholder schedule/threshold/guideline until clinical input.

### 4.5 Step 1 — Practice and address in mock

- Design shows practice (e.g. "Berrylands Surgery") and address on the patient card. For the prototype, should we (a) add these to the mock (hardcoded or derived from NHS), or (b) leave them blank/placeholder? **Recommendation:** (a) one fixed mock with practice + address for the "found" patient.

### 4.6 "Confirm and continue" without selecting a pathway

- Step 2 requires a pathway. If user clicks "Continue" without selecting one, we should block (validation) or default. **Recommendation:** Require pathway selection before Continue; optionally require date when pathway is a cancer pathway.

---

## 5. Implementation plan (ordered)

### Phase A — Step 2: Y/N and diagnosis autofill

1. Add "Is prostate cancer confirmed?" Yes/No radios at the top of Step 2.
2. When **Yes** is selected: set `#wizard-diagnoses` to `"Localised prostate cancer"`; optionally make field read-only or show hint that it’s autofilled.
3. When **No** is selected: clear `#wizard-diagnoses` (and do not autofill).
4. On change of Y/N, update diagnosis field and show/hide the correct pathway list (see Phase B).
5. In `addNewPatientFromWizard()`, read `diagnoses` from `#wizard-diagnoses` (so autofill is persisted when Yes; user can have changed it). Ensure new patient object has `diagnoses: 'Localised prostate cancer'` when Y was chosen (or user’s edit).

### Phase B — Step 2: Conditional pathways + Ownership + Date

1. **Conditional pathways:**  
   - If Yes: show current cancer pathway radios (RP, EBRT, Brachy, ADT, Intermittent, Watchful, Active surveillance).  
   - If No: show two radios: "General PSA monitoring", "Raised PSA follow-up" (with short descriptions).  
   Use two containers and toggle visibility from Y/N.
2. **Protocol keys for non-cancer:** Add e.g. `general-psa-monitoring` and `raised-psa-follow-up` to `protocolConfig` (and `protocolPhases`) with minimal schedule/threshold/guideline so Step 3 doesn’t break.
3. **Ownership:** Add "Ownership" radios (General Practice / Urology) to Step 2; default General Practice. Store in wizard state and pass to `addNewPatientFromWizard()`; set `ownership` on new patient. Backfill existing `detailData` with `ownership: 'GP'` (or from mock).
4. **Move anchor date to Step 2:** Move "Date of surgery or start of treatment" and the Year N badge from Step 3 to Step 2. Keep `#anchor-date` and phase calculation; show "Year N" next to the date. Step 3 still reads anchor for guideline highlight.

### Phase C — Step 1: Copy and patient card

1. Update Step 1 label to "Search by NHS number" and button to "Find" (icon optional).
2. Add success banner: "Patient found. Double-check details before proceeding."
3. Improve patient card: keep NHS, name, DOB; add optional practice and address in mock; add X to clear selection if desired.
4. Rename "Continue" to "Confirm and continue".

### Phase D — Step 3: Guidelines + Monitoring + Threshold layout

1. **Guidelines:** Rename/reflow table to Phase | Monitoring schedule | MOT | LFT (no Threshold column); keep current-year highlight; guideline text below. Title: "Guidelines for [Protocol name]".
2. **Monitoring block:** Add "PSA test schedule" dropdown, "Review for discharge in" dropdown, "And require: MOT, LFT" checkboxes. Wire to `frequency`, `reviewForDischargeYears`, and MOT/LFT (patient-level or from current phase — see details plan). Tooltips on MOT/LFT.
3. **Threshold:** Keep threshold input + 5-ARI; add short description. No logic change.
4. Add footer text: "You will be able to edit this information later."
5. If we keep 4 steps: Step 3 button stays "Continue"; if we merge to 3 steps: Step 3 button becomes "+ Add patient" and baseline PSA becomes optional in Step 3 or on detail.

### Phase E — Data model and mock data

1. Ensure every new patient has `ownership` and `diagnoses` (from Step 2, with autofill when Y).
2. Backfill `detailData` with `ownership: 'GP'` (or as needed) for existing mock patients.
3. Ensure detail page and list/board can read `ownership` and show it (detail page Pathway block; list/board optional filter later).

### Phase F — Validation and edge cases

1. Step 2: Require pathway selection before Continue; optionally require anchor date when pathway is cancer.
2. Step 1: Require successful "Find" (result visible) before "Confirm and continue" is enabled or before proceeding.

---

## 6. Checklist for Y/N and diagnosis

- [ ] Step 2: "Is prostate cancer confirmed?" Yes/No added.
- [ ] When Yes: Diagnosis field = "Localised prostate cancer"; user can edit or leave as is.
- [ ] When No: Diagnosis field cleared / not autofilled.
- [ ] Switching Yes → No clears autofill; No → Yes sets "Localised prostate cancer".
- [ ] New patient from wizard has `diagnoses` set from field (so autofill is saved when Yes).
- [ ] Mock data: existing patients get default `ownership`; new patients get `ownership` from Step 2.
- [ ] Detail page shows Diagnosis and Ownership; no regression.

---

## 7. Dependencies on details page plan

- **Ownership:** Detail page Pathway section should show Ownership (dropdown or read-only); wizard feeds it. Already in details plan.
- **Monitoring (single setting):** Step 3 "PSA test schedule" and "Review for discharge" are the same concepts as on the detail Monitoring section; store as `frequency` and `reviewForDischargeYears`.
- **Guidelines table:** Step 3 Guidelines table matches the collapsible Guidelines on the detail page (Phase | Monitoring schedule | MOT | LFT, highlight current year).
- **Schedule as card tag:** Once detail/card show Schedule instead of Year, wizard-added patients already have `frequency` from Step 3; cards will show it.

Once you confirm the open points (4.1–4.6) and 3 vs 4 steps (4.3), implementation can proceed in the order above.
