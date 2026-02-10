# Wizard and detail view redesign

**Status:** Design and implementation notes. Aligns with [CUSTOM_SCHEDULE_EXPLORATION.md](CUSTOM_SCHEDULE_EXPLORATION.md) and [WIREFRAME_POC_SPEC.md](WIREFRAME_POC_SPEC.md).

---

## 1. Wizard: full protocol override at add-time

**Goal:** When adding a patient, allow overriding not only the schedule for the current year but the **entire protocol** (all years) based on patient specifics.

**Proposed UX (Step 2):**

- After protocol + anchor date, show a **protocol table**:
  - **Column 1:** Year (1–10), with **protocol default** for each year (from `getFrequency(year)`).
  - **Current year** (derived from anchor) is **bold** / highlighted.
  - **Column 2:** Control to **redefine** that year (e.g. Same as protocol | 4‑monthly | 6‑monthly | Annual).
- User can leave all as “Same” or adjust any year. Effective schedule for the patient = protocol default + overrides.

### Is the “year/schedule summary” from the previous step redundant?

**Challenge: not entirely.**

- **Keep a one-line summary** (e.g. “Current period: **Year 2 — 6‑monthly**”) **above** the table because:
  - **At a glance:** Confirms which row is “now” without scanning 10 rows.
  - **Accessibility:** Screen-reader users get the current period without stepping through the whole grid.
  - **Consistency:** Matches the “current phase” concept used on cards and in the detail view.
- **Redundant only if** the table is the sole focus and we always bold “current year” and add an explicit “You are here” label in the table. Even then, the one-line summary is low cost and reduces cognitive load.

**Conclusion:** Keep the short “Current period: Year X — Y” line; treat the table as the place to **edit** the full pathway; the summary stays as the quick answer to “what applies right now?”

---

## 2. 5-ARI: checkbox + tag

**Change:** Replace the 5-ARI **radio group** (None / Finasteride / Dutasteride) with:

- **Primary control:** “On 5-ARI?” **Yes / No** (checkbox or two radio buttons).
- **Description:** “Finasteride or Dutasteride. We interpret PSA in light of 5-ARI (e.g. lower values expected).”
- **If Yes:** Capture **which drug** (Finasteride / Dutasteride) for interpretation and audit (e.g. dropdown or second line).

**Use 5-ARI as a tag:** Show “5-ARI” alongside **Protocol** and **Phase** as context (e.g. on cards and in detail header/chips).

### Questions and drawbacks

- **Why Y/N first?** Clarifies the clinical question (“Is the patient on a 5-ARI?”) before drug choice. Avoids “None” as the default option in a long list.
- **Drawback:** If we only store “on 5-ARI: yes/no”, we lose which drug. For threshold interpretation and audit, **which drug** matters → keep storing `medication5ARI`: `'none' | 'finasteride' | 'dutasteride'`; UI = checkbox + conditional “Which one?”.
- **Tag on cards:** “5-ARI” on the card could mean “on a 5-ARI” (binary). Showing “5-ARI” only when true keeps the tag set small (Protocol, Phase, 5-ARI). Optional: show “Finasteride”/“Dutasteride” on hover or in detail only.
- **Where in the wizard?** Step 3 (with threshold and baseline PSA) keeps “clinical context that affects interpretation” together. Alternatively Step 2 with pathway; we placed it in Step 3 so Pathway = protocol + schedule, Step 3 = threshold + PSA + 5-ARI.

---

## 3. Detail view: section reorganisation

| Section | Content |
|--------|---------|
| **1. Diagnosis** | Diagnoses only. Larger subheading. |
| **2. Pathway** | Same protocol table as wizard; **current period** highlighted. Guidelines for PSA (e.g. PSA &lt; 0.03 normal; review if &gt; 0.03; recall if &gt; 0.1 or 2 consecutive rises). **Next due** (and last bleed if relevant) integrated here. No collapsible “Schedule by year”; table + guidelines are always visible. |
| **3. PSA &amp; threshold** | Chart **100% width**. Latest PSA, threshold, result form, Δ legend. **5-ARI** checkbox/setting moved here (On 5-ARI? Y/N and which drug). |
| **4. Actions** | Unchanged (status-driven action cards + Discharge). |

**Removed / moved:**

- “Clinical context” as a section: Diagnoses → Section 1; 5-ARI → Section 3.
- Collapsible “Schedule by year &amp; when to review or recall”: replaced by always-visible pathway table + guidelines text.
- “Key dates” as its own section: Next due and last bleed live in Section 2 (Pathway).

**Subheadings:** Section titles use a larger font size and weight (e.g. 1.1–1.25rem, bold) so the four blocks are clearly separated.

---

## 4. Data model (per patient)

- **Schedule overrides:** Support **per-year** overrides. e.g. `scheduleByYear`: `{ 1: '4-monthly', 2: '6-monthly', ... }`. If a year is missing, use protocol default. `personalScheduleSet` = true if any year differs from protocol (or we derive it from `scheduleByYear`).
- **5-ARI:** Keep `medication5ARI`: `'none' | 'finasteride' | 'dutasteride'`. For display, “On 5-ARI?” = (medication5ARI !== 'none').
- **Protocol guideline:** Keep `protocolGuideline` (or take from `protocolConfig[].guidelineSummary` by protocol). Shown in Section 2 as plain text.

---

## 5. Protocol table (shared wizard + detail)

- **Wizard (Step 2):** Table with Year | Protocol default | Override (dropdown). Current year row bold. On submit, build `scheduleByYear` from overrides (only years that differ from protocol).
- **Detail (Section 2):** Same table; Protocol default column; second column = effective value (protocol or override). Current year row highlighted. Read-only unless we add “Edit schedule” (future); for POC, table is read-only in detail.
