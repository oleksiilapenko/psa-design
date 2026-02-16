# v2 Plan — Interview Decisions (resolved)

Decisions from the product interview. Use these when building the v2 prototype.

---

## Add patient / Specify the pathway

| Question | Decision |
|----------|----------|
| **"Is prostate cancer confirmed?"** | **Remove.** Single combined Pathway dropdown (all pathways in one list). |
| **Diagnosis** | **Dropdown with fixed list + "Other"** (optional free text or note). |
| **Guideline link** ("Dorset PSA Follow-up Protocol") | **Real URL** (provide or use standard NHS/guideline URL). |
| **5-ARI checkbox** | **Drop** for v2. |
| **Reset schedule** | **Confirmation dialog** before resetting to protocol default. |
| **Protocol vs freeform** | **Pathway first → table prefills from protocol;** clinician can override every year. |

---

## Scope and build approach

| Question | Decision |
|----------|----------|
| **Scope of v2** | **Full update:** Add patient + board + list + details page all aligned to new designs. |
| **v2 file strategy** | **Rebuild from scratch** (keep fast and reliable). |
| **Board and list views** | Use a **lightweight library (oat.ink)** for components and general style; keep prototype light and generic. Board/list: same conceptual structure (status columns, cards, list table) fed by v2 data; refine once oat.ink baseline is in place. |
| **Details page: Pathway/Threshold in view mode** | **Always editable** (inline dropdowns) so user can change without entering Customise. |

---

## Board/list + oat.ink (guidance)

- **Oat (oat.ink):** Ultra-lightweight, semantic, zero-dependency HTML UI library (~8KB CSS + JS). Good fit for a light, generic prototype.
- **Approach:** Use Oat for components and general style. Board and list keep the same **conceptual** structure as current (status columns, cards, list table) but:
  - Built with Oat where it helps (buttons, forms, inputs, tables, etc.).
  - Fed by v2 data model (scheduleRows, status, psaResults, threshold, etc.).
- **Refinement:** Once the Oat baseline is in place, board/list can be refined (e.g. card content, list columns, filters) without changing the overall approach.

---

## Rebuild and standardise (from second interview)

| Question | Decision |
|----------|----------|
| **Primary goal** | **Demonstrate** the new Add patient + details flow to stakeholders (click-through, no backend). |
| **Risk** | **Both:** Phased build (each step runnable) **and** keep v1 (`index.html`) **untouched**. |
| **Board/list refactor** | **Same structure** (board columns, list table) but **rebuild HTML/CSS with Oat** (or same approach if Oat is complex) for consistent look and behaviour. |
| **Standardise** | **All:** Process doc + template + Cursor skill. |
| **Dependencies** | **As vanilla as possible.** Oat only if integration is simple; if blockers, use the **same approach** without it (simple, OK by default, standard/safe for UX prototyping). |

See **REBUILD_CLARIFICATION.md** for how requirements translate to board/list and the phased plan. See **CURSOR_SKILL_UX_PROTOTYPE_DRAFT.md** for the skill to promote once v2 is approved.

---

## Copy / small fixes (from plan)

- **Discharge banner:** "Patient will be suggested for **Discharge** after the end of year N." (Fix "of the of" → "of" if present.)
- **PSA empty state:** "No results added yet. **Add** new result to start PSA monitoring." (Fix "And" → "Add" if present.)
