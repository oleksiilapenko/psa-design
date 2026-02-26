# v2 Rebuild — Clarification and translation plan

**Note:** The main prototype is now the single `index.html` (formerly built as index-v2.html and promoted to main).

This doc clarifies how the new requirements translate into the board and list, how we avoid getting stuck, and what “standardise this approach” means. It reflects your interview answers.

---

## 1. Your intentions (goals and constraints)

| Goal | Meaning |
|------|--------|
| **Primary** | **Demonstrate** the new Add patient + details flow to stakeholders. Click-through, no backend. |
| **Risk** | **Phased build** (each step runnable) **and** preserve a fallback during development (v2 was built in a separate file until promoted to main). |
| **Board/list** | **Same structure** (board columns, list table) but **rebuild HTML/CSS using Oat components** so it looks and behaves consistently. Not a data-only change — rebuild the UI with the chosen component/style approach. |
| **Standardise** | **Process doc + template + Cursor skill** so future prototypes follow the same approach. |
| **Dependencies** | **As vanilla as possible.** Oat looked like a good fit (simple, minimal). You’re **not** against Oat if integration is simple; if there are **blockers**, you want the **same approach** without it: simple styling, OK by default, standard and safe for UX prototyping. |

So: **vanilla-first; Oat only if straightforward.** If Oat adds complexity, we use the same philosophy (simple HTML/CSS/JS, semantic, good defaults, safe interactions) and skip the library.

---

## 2. How we translate requirements into board and list

### 2.1 Data flow (no breaking refactors)

- **Single source of truth:** One in-memory (or script-loaded) **patient list** that matches the v2 data model:
  - Identity (name, NHS, DOB, etc.)
  - Pathway, diagnosis, ownership, anchor date
  - **scheduleRows** (one row per year: yearFrom, yearTo, schedule, mot, lft)
  - **threshold**
  - **psaResults** (array of { date, value })
  - **status** (recall, upcoming, awaiting, completed, action, action_required, discharged, etc.)
  - **lastUpdated**
- **Board and list never own business logic.** They **read** from this dataset and **render**:
  - Board: filter by status → one column per status → one card per patient in that status.
  - List: same dataset → table rows; tabs/filters by status.
- **Add patient flow** and **details page** **write** into this dataset (add patient, update pathway/schedule/threshold/PSA results, change status). Board and list just re-render from the updated data.

So: **requirements → data model first; board/list are views over that model.** We don’t “refactor” v1’s logic into v2; we **define the v2 model**, then **build views** that consume it.

### 2.2 Board view — translation

| Requirement | Translation |
|-------------|-------------|
| Same conceptual structure | Same status columns (e.g. Upcoming, Awaiting, Completed, Review Required, Recall, Follow-up Required, Discharged) as now. |
| New data | Each card shows data from v2: protocol/pathway label, **schedule** (e.g. “6‑monthly”) or phase/year from scheduleRows, last PSA, threshold, next due, status. |
| Oat / simple UI | Cards and columns built with **semantic HTML**; if we use Oat, use its card/button styles; if not, use a small set of CSS classes (same approach: simple, consistent). |
| No breakage | Board is a **new** block in the new file; previous version stayed as reference during build. |

**Build order:** After we have the v2 data model and at least one “Add patient” run that appends to the list, we add the board: one column, one card shape, then add the rest of the columns. Each step is runnable.

### 2.3 List view — translation

| Requirement | Translation |
|-------------|-------------|
| Same structure | Same idea: status tabs, table with columns (Patient, NHS, Status, Protocol, Schedule, Diagnoses, Trend, Last PSA, Threshold, Δ, Next Due, Last updated). |
| New data | Table rows from the same v2 dataset; Schedule column from scheduleRows (e.g. effective schedule for current year); Trend from psaResults; Status from `status`. |
| Oat / simple UI | Table, buttons, badges use semantic HTML + Oat (or the same minimal CSS approach). |
| No breakage | List is a **new** block in the new file; previous list untouched during build. |

**Build order:** After board (or in parallel), add list: table shell, then bind columns to v2 data. Each step runnable.

### 2.4 What we do *not* do

- We did **not** refactor the original file in place; the new version was built in a separate file (e.g. `index-v2.html`) and is now the main `index.html`.
- We do **not** change the v2 data model shape for “easier” board/list rendering. Board/list adapt to the model.
- We do **not** add a build step or npm unless we hit a hard blocker and you agree.

---

## 3. Phased rebuild (avoid getting stuck)

| Phase | What we build | Runnable? | Fallback |
|-------|----------------|-----------|----------|
| 0 | New file (e.g. `index-v2.html`), minimal shell: header, one “Add patient” button, one “Board” / “List” placeholder. Load a **v2 data module** (e.g. inline JS object or separate script) with the v2 patient model. | Yes: open file, see header and button. | Previous version unchanged. |
| 1 | **Add patient flow:** Step 1 (Find patient), Step 2 (Specify the pathway: pathway, diagnosis, ownership, date, schedule table, threshold, “+ Add patient”). On submit, **append** to the v2 patient list and redirect to board or list. | Yes: add a patient, see them in data. | Previous version unchanged. |
| 2 | **Details page:** Open a patient from a simple list or URL; show pathway, schedule (view: grouped; edit: full table + Customise), threshold, PSA results (add new, edit latest), footer actions. Updates write back to the v2 list. | Yes: add patient, open detail, edit, save. | Previous version unchanged. |
| 3 | **Board:** Status columns, cards bound to v2 data; click card → details. | Yes: full click-through Add patient → Board → Detail. | Previous version unchanged. |
| 4 | **List:** Tabs and table bound to v2 data; click row → details. | Yes: full click-through including list. | Previous version unchanged. |

If we get stuck in a phase, we **don’t** break the previous phase. We either fix in place (small scope) or revert the new file to the last good state.

---

## 4. Prototype standard (repeatable approach)

So that we can reuse this for future prototypes:

- **Vanilla-first:** No framework. Single HTML file (or very few: e.g. one HTML + one JS + one CSS, or all inline). No npm/build unless necessary.
- **Optional lightweight lib:** If we use something (e.g. Oat), it must be **simple** to integrate (e.g. link tag + script tag, or copy in a few files). If integration is complex or blocking, we **drop the lib** and keep the **same approach**: semantic HTML, minimal CSS, clear component-like patterns, good defaults, safe interactions.
- **Wireframe/data-structure first:** Use designs as **wireframe and data-structure** input, not only visual spec. Define the data model (entities, fields) before building views.
- **Phased and runnable:** Each phase produces a runnable artifact. Don’t do a big-bang refactor.
- **Reference preserved:** When evolving a prototype (e.g. v2), keep the previous version untouched during build so we can fall back or compare until the new version is promoted to main.
- **Purpose:** Stakeholder **demo** (click-through). Not production, not full backend. “Spec in HTML” is a bonus for handoff.

This is what we’ll document as the **process**, bake into a **template** (e.g. v2 file structure + how we load data and views), and encode in a **Cursor skill** so the agent can “build a UX prototype this way” when asked.

---

## 5. Oat specifically

- **Use Oat if:** We can add it with a link/copy and use it for buttons, inputs, tables, cards without custom build or heavy config. Then board/list are “same structure, Oat UI.”
- **Skip Oat if:** Integration is complex, or you prefer to stay with plain HTML/CSS. In that case we keep **the same approach**: simple, semantic, consistent styling and interaction patterns, so the prototype still feels like “one library’s philosophy” without the dependency.

Your call at the start of the build: try Oat first, or go plain from the start. We can document both in the skill.
