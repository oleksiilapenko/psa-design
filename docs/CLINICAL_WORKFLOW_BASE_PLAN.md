# Clinical workflow: base plan (product & business alignment)

**Purpose:** Structure clinical/business feedback into a single base plan for modifying and extending the PSA monitoring workflow. This document defines target states, transitions, card actions, and data; resolves ambiguities where possible; and lists clarifying questions for stakeholders. It also checks alignment with the attached state machine draft.

**Role:** Senior BA / clinical expert — analyse, structure, ask questions. Implementation follows when the plan is agreed.

---

## 1. Summary of feedback (structured)

### 1.1 New column: Action required

- **Requirement:** A new column **Action required** holds patients who need **additional action** after their result is identified as **above the limit** (abnormal).
- **Key data:** The main information on the card and patient page is the **required action** field.
- **Required action options (current list):**
  - USC (Urology)
  - Advice & guidance
  - Pending other results
  - Other
- **When it’s set:** When the patient is in **Review Required**, the user must **select the reason** (explanation of what action is pending) before or when moving them to Action required. So: Review Required is “GP evaluates”; Action required is “GP has decided an action type; that action is recorded and visible.”

**Implication:** We need to distinguish two columns:
- **Review Required** — PSA above threshold; GP reviews trend and result; **no** required-action type yet.
- **Action required** — A specific follow-up action has been chosen (USC, Advice & guidance, etc.); this is the “queue” for that action.

Flow for abnormal results: **Awaiting result** → (result above threshold) → **Review Required** → (user selects reason) → **Action required** **or** **Recall** (see §3).

---

### 1.2 Normal result flow (updated logic)

| Step | Column / state | Description |
|------|----------------|-------------|
| 1 | **Recall** | Waiting for due date (full pool of patients on monitoring). |
| 2 | **Upcoming** | Shortened list: subset of Recall who are due soon — prepared for test upfront. |
| 3 | **Awaiting result** | Move here **manually for now** when test has been done; following up the result. |
| 4 | **Completed** | Results resolved as **normal** (manually or auto); card **remains visible for one week** for visibility. |
| 5 | **Recall** | After that week, patient **moves back to the start of the flow automatically** (i.e. back into Recall, then eligible for Upcoming when due). |

**Clarifications captured:**
- **Recall** is the “waiting for due date” pool; **Upcoming** is the time-bound shortlist for imminent tests (e.g. next 4 weeks).
- **Upcoming → Awaiting:** manual “test done” (for now).
- **Completed:** “hangs” for **one week** (not 48h as in current POC copy) for visibility.
- **Completed → Recall:** automatic after that period; then normal flow continues (Recall → Upcoming when due, etc.).

**Confirmed:** Recall → Upcoming uses a due-date rule (e.g. “due in next 4 weeks”); see §6.

---

### 1.3 Abnormal result flow and the Review vs Completed question

**Requirement:** For **bad/abnormal** results, the user can either:
- **Move to Action required** (temp name) and **specify the details** (required action type), or  
- **Move to Recall** (optionally with some context of the action saved).

**Uncertainty you raised:** Should the patient:
- **A)** Go **via Completed (e.g. “Review completed?”)** first, to align with the “good result” flow, or  
- **B)** Go **straight to Recall** (perhaps with action context saved)?

**Assumption from your team:**  
- **Review Required** = focus for **GPs** to evaluate (above-threshold result; decide what happens next).  
- **Completed** = focus for **Admin** to double-check automation results (normal results; quick check).

That implies:
- **Completed** = “result normal; no clinical action needed; admin visibility for a week then back to Recall.”
- **Review Required** = “result abnormal; GP decides: Action required (with type) or Recall (back to clinic).”

So for **abnormal** results we do **not** need a “Review completed” state that mirrors Completed. The GP’s decision is binary from Review Required: **Action required** (with reason) or **Recall**. Going via a “Completed (Review completed?)” would mix two different meanings of “completed” (normal result vs “review done”) and blur who owns the column. Recommendation below in §2.

---

## 2. Resolved logic: Review vs Completed and abnormal path

### 2.1 Recommendation: do **not** route abnormal via Completed

**Proposed:**

- **Completed** = only for **normal** results (PSA ≤ threshold). Meaning: “No action needed; result acceptable; card stays 1 week then auto → Recall.” Primary use: Admin double-check.
- **Review Required** = only for **abnormal** results (PSA &gt; threshold). Meaning: “GP evaluates.” From here the only transitions are:
  - **→ Action required** (user **must** select required action: USC, Advice & guidance, Pending other results, Other), or  
  - **→ Recall** (back to clinic; optionally store a short reason/context for audit).

**Why not “Review completed” via Completed?**

| Approach | Pros | Cons |
|----------|------|------|
| **Abnormal → Completed (“Review completed”) first** | Symmetric with normal flow (everyone passes through Completed). | **Semantic clash:** Completed today means “result normal.” “Review completed” would mean “GP finished review” but result is still abnormal. Admin would see a mix of “normal” and “abnormal” in the same column. Reporting and filters (e.g. “all normal results”) become unclear. |
| **Abnormal → Action required or Recall directly** | Clear split: Completed = normal only; Action required = specified action; Recall = back to clinic. GP and Admin roles stay clear. | None for workflow clarity. We only need to ensure “required action” and optional “reason for Recall” are stored for audit. |

**Conclusion:** Keep **Completed** for normal results only. For abnormal results, go **directly** from Review Required to **Action required** (with required action selected) or **Recall** (with optional context). Do not introduce a “Review completed” state that reuses Completed.

### 2.2 Potential conflict and how to avoid it

- **Conflict:** If we ever wanted “all reviewed cases” in one place, mixing “Review completed” into Completed would make that one place ambiguous (normal vs abnormal).
- **Avoidance:** “All reviewed cases” can be derived from: (a) Completed (all normal), (b) Action required (all abnormal with action type), (c) Recall (includes abnormal sent back to clinic). So we keep one meaning per column and derive aggregates in reporting.

---

## 3. Target state machine (aligned with feedback)

### 3.1 States (columns)

| State (internal) | Column label | Owner / meaning |
|------------------|--------------|------------------|
| `recall` | **Recall** (see §8 for naming options) | Pool waiting for due date; auto-fed from Completed after 1 week; also where abnormal patients go when sent “back to clinic.” |
| `upcoming` | **Upcoming** | Shortlist of Recall due soon (due-date rule, e.g. next 4 weeks); prepare for test. |
| `awaiting` | **Awaiting result** | Test done (manual move); waiting for result. Result entry can be manual or auto; move to Completed/Review Required is **auto** based on PSA vs threshold. |
| `review` (or keep `action`) | **Review Required** | Result above threshold; **GP evaluates**; no required-action type yet. |
| `action_required` (new) | **Action required** (see §8 for naming options) | User has chosen a **required action** (USC, Advice & guidance, Pending other results, Other + free text); key data on card/page = this field. **Required action is editable** after move. |
| `completed` | **Completed** | Result **normal** (≤ threshold); visible **1 week** for Admin; then **auto → Recall**. |
| — | **Discharged** | Off board; **available from all states** (mock until product defines). |

### 3.2 Transitions (target)

| From | To | Trigger | Notes |
|------|-----|---------|------|
| — | Recall or Upcoming | Add new patient | **Recall** if due date not soon; **Upcoming** if due soon (based on condition). |
| Recall | Upcoming | Auto (due date) | When “due in window” (e.g. next 4 weeks); move based on due date. **Confirmed.** |
| Upcoming | Awaiting result | Manual “Test done” / “Blood taken” | Manual for now. |
| Awaiting result | Completed | **Auto** (default) or manual | When result entered: if PSA ≤ threshold, **auto** move to Completed. Manual result entry also triggers auto-move to correct column. |
| Awaiting result | Review Required | **Auto** (default) or manual | When result entered: if PSA &gt; threshold, **auto** move to Review Required. Manual result entry also triggers auto-move. |
| Review Required | Action required | Manual | **User must select required action** (USC, Advice & guidance, Pending other results, **Other + free text**). |
| Review Required | Recall | Manual | Back to clinic; **no additional info required**. |
| Action required | Recall | Manual | User moves to Recall when appropriate (e.g. action done or no longer needed). |
| Any state | Discharged | Manual | **Discharge** available from **all states**; mock until product defines. |
| Completed | Recall | **Auto after 1 week** | Card visible 1 week then auto-move. **Confirmed.** |

**New data:** **required_action** (USC (Urology), Advice & guidance, Pending other results, **Other** with **free-text field**). Required action is **editable** when patient is in Action required. No mandatory recall reason when moving to Recall.

---

## 4. Alignment with the attached state machine diagram

### 4.1 What matches

- **Recall** as the “waiting for due date” pool and **auto-move from Completed** (dashed “Move to recall after 7 days (auto)”) — aligns with “hangs for a week” then auto → Recall.
- **Upcoming** between Recall and Awaiting.
- **Awaiting Results** → **Completed** (normal) and → **Review Required** (abnormal).
- **Review Required** → **Action required** and → **Recall**.
- **Action required** → **Recall** (dashed line in diagram).
- **Discharge** from Recall (grey; mock/out of scope for now).

### 4.2 Diagram condition (fixed)

- The arrow to **Review Required** should say result **higher** than threshold (abnormal). **Fixed on stakeholder’s side.**
- For completeness:
  - **Awaiting Results → Completed:** result **lower than or equal to** threshold (normal).
  - **Awaiting Results → Review Required:** result **higher** than threshold (abnormal).

### 4.3 Gaps vs this base plan

- Diagram does not show **required action** (USC, Advice & guidance, etc.) as a **mandatory choice** when moving from Review Required to Action required; the plan makes that explicit.
- “Schedule this phase” / “Monitoring” and protocol are out of scope of the diagram; no change needed.
- **Recall → Upcoming:** diagram shows “Move based on the due date (auto)”; plan confirms due-date rule (e.g. next 4 weeks).
- **Discharge:** plan states Discharge is available from **all states**; diagram may show it only from Recall — align if needed.

---

## 5. Card and patient page: key data

### 5.1 Action required column

- **Card:** Show **required action** prominently (e.g. “USC (Urology)”, “Advice & guidance”, “Pending other results”, “Other”). If “Other”, consider short free text on card or only on detail.
- **Patient (detail) page:** **Required action** is a key field: select before moving from Review Required; **editable** when in Action required. Options: USC (Urology), Advice & guidance, Pending other results, **Other** (with **free-text field**).

### 5.2 Review Required column

- **Card:** Continue to show PSA vs threshold and trend; actions = “Select action and move to Action required” (opens choice of required action) and “Move to Recall” (optional reason).
- **Patient page:** When in Review Required, show clear actions: (1) **Move to Action required** — **required action** dropdown (USC, Advice & guidance, Pending other results, Other + free text) mandatory; (2) **Move to Recall** — no additional info required.

### 5.3 Last PSA outcome signifier on cards (Upcoming column)

- **Purpose:** Surface the **outcome of the last PSA** (normal vs above threshold) next to the value so users can scan quickly. Uses existing data (last PSA + threshold); no new field required.
- **Where:** **Upcoming column** cards only. Show the signifier only where **last PSA is already present** (i.e. we have a numeric last result to compare to threshold).
- **Signifier (small, next to the value):**
  - **Last PSA normal (≤ threshold):** grey **checkbox** (✓) next to the PSA value — “fine.”
  - **Last PSA above threshold:** **arrow + delta** (e.g. ↑ +0.03) next to the value — same convention as elsewhere (above threshold).
- **Placement:** Immediately next to the last PSA value on the card.
- **Note:** The same pattern can be used on **Recall** column cards to distinguish “on pathway, waiting for due” (last normal) vs “recalled to clinic” (last above); see §5.4 if that is in scope.

### 5.4 Recall column: context from last PSA result (optional)

- On the **Recall** column, the same signifier (checkbox if last PSA ≤ threshold, arrow + delta if above) gives context: last result normal → “on pathway, waiting for due date”; last result above → “recalled to clinic.” Optional extension of §5.3 to Recall cards (and List when viewing Recall).

---

## 6. Stakeholder decisions (confirmed)

| # | Question | Decision |
|---|----------|----------|
| 1 | New patient: land in Upcoming or Recall? | **Recall**, or **Upcoming** if due soon — **based on condition** (due date). |
| 2 | Recall → Upcoming: exact rule? | **Yes** — due-date rule (e.g. due in next 4 weeks). |
| 3 | When does patient leave Action required? | **Manual** move to **Recall**; plus **Discharge** (available from **all states**). |
| 4 | “Other” for required action? | **Other + free-text field**. |
| 5 | Required action editable after move? | **Yes**. |
| 6 | Awaiting → Completed / Review: auto or manual? | **Auto as default** when result is entered; **manual result entry** also allowed, and move to the respective column is **auto**. |
| 7 | Completed visibility before auto → Recall? | **Confirmed:** 1 week. |
| 8 | Mandatory recall reason when moving to Recall? | **No** additional info needed. |
| 9 | Discharge: from Recall only? | **Discharge** available from **all states** (mock until product defines). |

---

## 7. Implementation checklist (when plan is agreed)

- [ ] Add new state `action_required` and column **Action required** (board + list).
- [ ] Add **required_action** to patient data model; options: USC (Urology), Advice & guidance, Pending other results, Other (optional free text).
- [ ] **Review Required:** from this column, only allow transition to **Action required** (with required action selected) or **Recall** (optional reason). Enforce selection of required action before move to Action required.
- [ ] **Action required:** show required action on card and patient page; define transition(s) out (e.g. to Recall when action done).
- [ ] Normal flow: **Completed** = normal only; **Completed** visible 1 week then **auto → Recall** (replace 48h copy and logic).
- [ ] **Recall → Upcoming:** implement or document due-date rule (e.g. due in next 4 weeks).
- [ ] **Upcoming → Awaiting result:** ensure manual “Test done” / “Blood taken” is available (already in scope).
- [ ] Sync state across board, list, and detail (single source of truth) for all moves.
- [ ] Update state machine diagram: fix Awaiting → Completed (≤ threshold) and Awaiting → Review Required (&gt; threshold); add required action to Review Required → Action required.
- [ ] Update copy: column notes, list tabs, and any in-app legend to match “1 week”, automation, and chosen state names.
- [ ] **Discharge:** available from **all states** (UI + behaviour; mock until product defines).
- [ ] **Required action:** Other + free-text field; required action editable when in Action required.
- [ ] **Result entry:** auto-move to Completed or Review Required based on PSA vs threshold (manual result entry supported; move is auto).
- [ ] Apply **state naming** choices from §9 (if any renames are adopted).
- [ ] **Upcoming column cards:** Small signifier next to last PSA value when present: **grey checkbox** if last PSA ≤ threshold (normal), **arrow + delta** if above threshold. Upcoming only; see §5.3. Optional: same on Recall cards (§5.4).

---

## 8. State naming: analysis and proposal

This section analyses the current column/state names in this iteration and proposes alternatives that better encapsulate the workflow. Use this to decide final copy for board, list, and detail.

### 8.1 Current names and issues

| Current name | Role in workflow | Issue |
|--------------|------------------|--------|
| **Recall** | (1) Pool waiting for due date; (2) destination when GP sends patient “back to clinic” from Review Required. | “Recall” can suggest only “recall to clinic,” but the column holds both the pathway pool (waiting for next due) and recalled patients. The name is acceptable: “Recall” works as the list you’re *on* (recall list / monitoring recall). “Move to Registry” was considered but rejected: it sounds like filing into a system rather than a workflow column. |
| **Upcoming** | Shortlist of Registry/Recall due soon. | Clear; no strong issue. |
| **Awaiting result** | Test done; waiting for result. | Clear. |
| **Review Required** | GP evaluates (result above threshold). | Clear. “Required” is shared with next column. |
| **Action required** | User has chosen a follow-up action type (USC, Advice & guidance, etc.). | Overlaps with “Review required” (both use “required”). Doesn’t signal that this is the **post-review** step or a **queue** of chosen actions. |
| **Completed** | Result normal; 1 week visibility; then auto → Recall. | Short but doesn’t say “normal.” For reporting and Admin, “Completed” could mean “any completed step,” not specifically “result normal.” |
| **Discharge** | Off board (all states). | “Discharged” (past tense) would align with “patient has been discharged.” |

### 8.2 Proposed names and rationale

| Current | Proposed | Why it’s better |
|---------|----------|------------------|
| **Recall** | **Recall** (keep) | “Move to Registry” sounds odd (Registry = the system/list, not a column you move into). **Recall** is fine as the column name: it reads as the “recall list” or “on monitoring recall.” Use the **column note** to clarify: e.g. “On pathway — waiting for next due date, or recalled to clinic.” |
| **Upcoming** | **Upcoming** (keep) or **Due soon** | “Upcoming” is already clear. “Due soon” is more action-oriented; optional. |
| **Awaiting result** | **Awaiting result** (keep) or **Results pending** | Current name is fine. “Results pending” is a minor alternative. |
| **Review Required** | **Review required** (keep) or **Needs review** | Both work. “Needs review” is slightly softer; “Review required” is explicit. |
| **Action required** | **Follow-up action** or **Action queue** | “Follow-up action” makes clear this is the **next step after review** (chosen action type). “Action queue” signals a **list of patients with a chosen action** (USC, Advice & guidance, etc.). Either reduces overlap with “Review required” and clarifies role. |
| **Completed** | **Completed** (keep) or **Result normal** | “Completed” is brief. “Result normal” is explicit for reporting and Admin (“all normal results”) but longer. Recommend keeping “Completed” and supporting meaning with column note: “Result normal; no action needed.” |
| **Discharge** | **Discharged** | Past tense aligns with “patient has been discharged” and is consistent with other past-tense outcomes. |

### 8.3 Recommendation summary

- **Keep:** **Recall**. “Move to Registry” was considered but sounds like filing into a system; “Recall” works as the column name (recall list / on monitoring recall). Clarify with a **column note**: e.g. “On pathway — waiting for next due date, or recalled to clinic.”
- **Rename:** **Action required** → **Follow-up action** or **Action queue** to distinguish from Review required and to describe the column’s purpose (chosen follow-up type).
- **Consider:** **Discharged** instead of Discharge for the off-board state.
- **Keep:** Upcoming, Awaiting result, Review required, Completed; use column notes where needed (e.g. Completed = “Result normal”).

---

## 9. Document control

- **Source:** Clinical/product feedback, state machine draft (FigJam/diagram), and stakeholder decisions (§6).
- **Current state:** Decisions captured; naming in §8 (title case: Action Required, etc.).
- **Implementation status (prototype):** Single-file prototype in `index.html` (~2,190 lines). Action Required column and required_action field are implemented. Board shows 5 columns (Upcoming, Awaiting Results, Completed, Review Required, Action Required); Recall and Discharged are off the board. Completed copy: 7 days then auto to Recall (no timer in code). Discharge is available from all non-discharged statuses via the patient detail “Discharge” subsection and via the 3-dot menu where applicable. All moves sync board, list, and detail. Add Patient wizard: 3 steps (Find patient → Pathway + protocol table → 5-ARI, threshold, baseline PSA); submitting adds a real patient to `detailData`, board, and list. Patient detail is reorganised into four sections: 1. Diagnosis, 2. Pathway (table + guidelines + Next due), 3. PSA & threshold (chart, 5-ARI), 4. Actions. See [PROTOTYPE_STATE.md](PROTOTYPE_STATE.md) and [WIZARD_AND_DETAIL_REDESIGN.md](WIZARD_AND_DETAIL_REDESIGN.md).
- **Related docs:** [PATIENT_CARD_STATE_MACHINE.md](PATIENT_CARD_STATE_MACHINE.md), [WIREFRAME_POC_SPEC.md](WIREFRAME_POC_SPEC.md), [PROTOTYPE_STATE.md](PROTOTYPE_STATE.md), [WIZARD_AND_DETAIL_REDESIGN.md](WIZARD_AND_DETAIL_REDESIGN.md), [CUSTOM_SCHEDULE_EXPLORATION.md](CUSTOM_SCHEDULE_EXPLORATION.md).

---

## 10. Recent changes and implementation context

This section summarises notable implementation changes so the doc stays aligned with the live prototype.

### 10.1 Add Patient wizard (current flow)

- **Step 1:** NHS number, mock search → patient result (name, DOB, NHS No) → Continue.
- **Step 2:** Diagnoses (optional), protocol radios, treatment start (anchor date). **Protocol table:** Year 1–10 with protocol default frequency per year and an **override** dropdown per row (Same as protocol | 4‑monthly | 6‑monthly | Annual). Current year (from anchor) is highlighted. Continue.
- **Step 3:** **On 5-ARI?** Yes/No (checkbox); if Yes, “Which one?” (Finasteride/Dutasteride). Protocol default threshold and personal threshold; last known PSA and date. **Add patient** creates a real patient: `detailData[id]` is populated, a board card is added to Upcoming, and a list row is added. Toast: “Patient added. They appear in Upcoming.”
- **Data from wizard:** Name, NHS, DOB, protocol, phase, anchor, `scheduleByYear` (from overrides), threshold, last PSA/date, diagnoses, `medication5ARI` ('none' | 'finasteride' | 'dutasteride'). New cards show a 5-ARI chip when applicable.

### 10.2 Patient detail layout

- **1. Diagnosis** — Diagnoses only (editable).
- **2. Pathway** — Protocol, phase, treatment start; **protocol table** (Year 1–10, effective schedule; current year highlighted); PSA guidelines text; Next due and Last bleed.
- **3. PSA & threshold** — Full-width sparkline; latest PSA, threshold (editable); **On 5-ARI?** Yes/No and which drug (editable); result entry when relevant.
- **4. Actions** — Status-driven action cards; **Discharge** in a separate subsection (red accent; all non-discharged).

Removed from detail: collapsible “Schedule by year”; “Key dates” as a separate section (folded into Pathway); frequency edit and schedule asterisk in the old layout.

### 10.3 Defensive JavaScript (robustness)

To avoid a single missing DOM element or runtime error breaking the whole app (e.g. no listeners attached), the script was hardened as follows:

- **Safe listener helper:** `on(el, ev, fn)` — only calls `addEventListener` when `el` exists. All critical UI buttons (view toggle, Add Patient, wizard steps, wizard submit, detail close) use `on(...)` instead of direct `element.addEventListener(...)`.
- **Null guards:** `showView`, `showToast`, `openDetailModal`, `closeDetailModal`, `setWizardStep`, `getWizardStep`, and `updateProtocolOutput` check that the relevant DOM nodes exist before using them. Card actions dropdown and protocol tooltip handlers guard against null.
- **Sparkline init:** `injectSparklineThresholds()` is wrapped in `try { ... } catch (err) { console.warn('injectSparklineThresholds:', err); }` so a throw there does not prevent the rest of the script from running. Accesses to `sparkEl.parentNode` are guarded.
- **Single wizard-step3-back handler:** The two previous handlers for “Back” on step 3 were merged into one (step navigation + hiding 5-ARI “which” wrap).

Effect: If any one element is missing or an early init step throws, other features (wizard, list, board, card menus, detail) can still run. Console warnings help track any remaining init issues.

### 10.4 Data and sync (unchanged)

- **Source of truth:** `detailData[id]`; board cards use `data-detail` and `data-status`; list rows use `data-detail`, `data-tab`, `data-status`. Moves from 3-dot menu or detail action cards update all three. New patients from the wizard are appended to board and list and to `detailData`.
- **Protocol config:** `protocolConfig` (radical, ebrt, brachytherapy, primary-adt, intermittent-hormones, watchful-waiting, active-surveillance) drives threshold, schedule summary, guideline text, and `getFrequency(year)` for the wizard table and detail.
