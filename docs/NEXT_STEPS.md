# Next steps: Refine and extend protocols

Proposed order of work to harden the prototype and extend to further protocols. **Radical Prostatectomy, EBRT, and Brachytherapy** are already implemented; next candidates: ADT, Watchful Waiting, Active Surveillance.

---

## Unpacked: What each step means in practice

**1. Validate flows**  
- Book 2–3 sessions with GPs/CNS/Admin (or one mixed group).  
- Give them the prototype and a short scenario: “Add a patient, then open someone in Awaiting, enter a result, and choose an action.”  
- Watch where they hesitate, misclick, or ask “what does this do?”  
- Write down: confusing labels, missing actions, and whether the order of information (card vs modal) works.  
- Explicitly ask: “If a result is very late, what would you expect to see?” and “If you marked someone Completed by mistake, how would you fix it?”

**2. Lock content**  
- Export all user-facing strings (column titles, button labels, hints, toasts) into a single list or spreadsheet.  
- One pass: plain English, consistent terms (e.g. “threshold” not “limit”), and NHS tone.  
- Decide for each: keep as is, shorten, or move to “Learn more” / collapsible.  
- Add the missing bits: empty state per column, validation messages for PSA/date, and any new reason codes.

**3. Enrich mock data**  
- Add patients until List tabs have 3–5 rows each; keep Kanban to 2–3 cards per column so the board stays scannable.  
- For each patient, set next-due and last-PSA dates so they’re consistent with the protocol (e.g. 6‑monthly spacing).  
- Include at least one “story” per type: stable, rising, one spike, personal threshold, no result yet.

**4. UX polish**  
- Modal: Escape to close, focus trap, focus on open.  
- Buttons: clear labels, no “(mock)” in production.  
- Optional: sticky first column in List when horizontal scroll appears.

**5. Document for build**  
- One table: protocol name, phase, frequency, normal/alert/recall thresholds, and any special rule (e.g. “2 consecutive rises”).  
- One page: how we store and show personal threshold, reason for override, completed, and discharged.  
- So a developer can add a new protocol without re-reading the whole PRD.

**6. Add more protocols**  
- Brachytherapy is already in the prototype (default threshold 0.5 ng/mL, short label “Brachy.”). Next: ADT, then Watchful Waiting / Active Surveillance (different rules).  
- No new columns or views; only new protocol options and protocol-specific hint text where needed. See **Protocol extension** below for how to add a protocol in code.

---

## 1. Validate flows with users

- **Walkthrough with GPs / CNS / Admin:** Run through Add Patient, enter result (Awaiting → Save result → triage), Review Required (edit result, Mark completed / Move to Recall), and List view tabs. Capture where copy or layout is unclear.
- **Edge cases:** What if result is delayed (e.g. 3 weeks since bleed)? What if the user enters the wrong PSA and needs to correct it after moving the card? Confirm whether “edit result” in Review is sufficient or if we need an “undo” or “re-open” from Completed.
- **Completed column:** Confirm that “cards leave after 48h” and “see List → Completed for all” matches how practices want to work. Decide if a count (“4 completed this week”) would help.

---

## 2. Lock content and copy

- **Column descriptions:** Short descriptions are in place; decide if they stay visible by default or become “What does this column mean?” (collapsible). Align with NHS content style (plain English, no jargon where possible).
- **Actions:** Text and actions are status-driven. Add any missing reason codes (e.g. for threshold override or for Move to Recall).
- **Error and empty states:** Add copy for “No patients in this column” and for validation (e.g. invalid PSA format, missing date).

---

## 3. Enrich mock data (still no backend)

- **Volume:** Add enough patients so each List tab has 3–5 rows and the “All” view feels like a real list (e.g. 15–20 total). Keep Kanban to a handful of cards per column so the board doesn’t feel crowded.
- **Consistency:** Align next-due dates with protocol (e.g. 6‑monthly = 6 months after last test). Add a mix of “last PSA” dates so some cards feel recent and some older.
- **Illustrative cases:** Include at least one of each: stable low PSA, gradual rise, one spike (e.g. EBRT bounce), personal threshold set, and “awaiting result” with no prior PSA. Add NHS-style names and DOBs so demos feel realistic.

---

## 4. Small UX tweaks

- **Keyboard and focus:** Ensure modal can be closed with Escape; focus trap inside modal; focus moves to “Close” or first control when opening.
- **Accessibility:** Check headings order, button labels, and status badges with a screen reader. Ensure “Save result” and “Save threshold” have clear success feedback (toast is in place; consider live region if needed).
- **List view:** If horizontal scroll appears with many columns, consider sticky “Patient” column or a compact layout for small screens (later).

---

## 5. Document protocol rules for build

- **Single source of truth:** Turn the protocol matrix (frequency by year, normal/alert/recall thresholds) into a small spec or table that front-end and future backend can share. This will make adding ADT, Watchful Waiting, etc. a matter of adding rows, not re‑inventing logic.
- **Conventions:** Document how “personal threshold” and “reason for override” are stored and displayed (in the prototype: `threshold`, `protocolThreshold`, `personalThresholdSet`; display uses * and legend “* Override of protocol default”), and how “completed” and “discharged” are represented (so archive behaviour is clear when we add it).
- **Delta (Δ):** In the prototype, Δ = PSA − threshold (one meaning everywhere). Shown in the list column “Δ vs threshold” (signed; “—” when no result), in detail next to Latest PSA, and on **Review Required** cards only. Legend in list/detail: “Δ = PSA − threshold”.
- **Clinical context:** Diagnoses and 5‑ARI are **patient-level** (not protocol-specific). They are stored per patient (`diagnoses`, `medication5ARI`) and used in the UI for context; future logic (e.g. threshold interpretation, eligibility) can use them across protocols.

---

## 6. Then add more protocols

- **Order:** Brachytherapy is done. Next: ADT (e.g. 6‑monthly, 4.0), then Watchful Waiting / Active Surveillance (different rules: doubling time, rise &gt; 0.75).
- **UI:** Reuse the same Kanban and List; protocol and phase already drive labels and thresholds. Add protocol to filters if needed (e.g. “Show only EBRT”).
- **Copy:** Adjust Actions and any protocol-specific hints (e.g. “Consider EBRT bounce”) per protocol where the PRD calls it out.

---

## Protocol extension: How to add a new protocol (developer)

The prototype is built so that adding a protocol is mostly configuration plus one-off UI wiring.

**1. Protocol config (JavaScript)**  
In `index.html`, extend the `protocolConfig` object. Each key is the option id used in the wizard; each value has:
- `threshold` — default PSA threshold (string, e.g. `'0.5'`)
- `phase` — default phase label shown in wizard (e.g. `'Year 1 — 6‑monthly'`)
- `help` — short help text for triage (shown under the protocol radio in Step 2)

Example (Brachytherapy is already present):
```js
brachytherapy: { threshold: '0.5', phase: 'Year 1 — 6‑monthly', help: 'PSA < 0.5 ng/mL often used; confirm local policy. Rising PSA may need imaging or referral.' }
```

**2. Wizard Step 2**  
- Add a new radio input: `name="protocol"`, `id="protocol-<id>"`, value matching the config key (e.g. `brachytherapy`).
- Add the label and help text block (same pattern as existing protocols). The existing `updateProtocolOutput()` reads the selected value and fills phase and Step 3 default threshold from `protocolConfig`.

**3. Short label**  
In the function `protocolShortLabel(p)` (used for chips and list “Protocol” column), add a case for the new protocol’s full name, e.g. return `'Brachy.'` for `'Brachytherapy'`.

**4. Data**  
- Each patient in `detailData` has `protocol` (full name, e.g. `'Brachytherapy'`). Add or reuse patients with the new protocol for demos.
- Optionally add list/card rows in the mock data so the new protocol appears in Kanban and List.

**5. 5‑ARI and diagnoses**  
These are **patient-level** fields (`medication5ARI`, `diagnoses`). They are already in the wizard (Step 2: Diagnoses; Step 3: 5‑ARI) and in the detail “Clinical context” section. No protocol-specific changes needed; future threshold or eligibility logic can use them across all protocols.

---

## Summary

| Step | Goal |
|------|------|
| 1 | Validate flows with real users; confirm edge cases |
| 2 | Lock all copy and error/empty states |
| 3 | More, consistent, illustrative mock data |
| 4 | Keyboard, focus, a11y polish |
| 5 | Document protocol matrix and conventions for build |
| 6 | Add ADT → Watchful Waiting / Active Surveillance (Brachytherapy done) |

After 1–5 the prototype is ready to hand over for build or to extend to further protocols without reworking core flows.

---
## Capability questions: What will the system actually do?

To align the **Actions** section (and the rest of the UI) with the real product, it helps to pin down what will be possible. Below are questions for product / clinical lead. Answers will drive which actions we show, what they’re called, and what happens after (e.g. confirmation, audit log).

**Upcoming**  
- Will “Generate blood form” (or equivalent) produce a printable/PDF form, or trigger a request to another system?  
- Will “Send reminder” exist as a separate action (e.g. letter, SMS, task to admin), or is it the same as the blood form?  
- Can the user change the “next due” date manually (e.g. patient away), or is it always calculated from the protocol?

**Awaiting results**  
- Will PSA results be entered only manually, or will some (or all) come from a lab feed? If both: do we show “Result received” vs “Enter result” depending on source?  
- When the user enters a result, will the system automatically move the card to Completed or Review Required, or will the user always choose the action after entering?  
- Will “Chase result” create a task, send a message, or just record that chase was done?

**Review required**  
- When the user chooses “Mark as completed” (accept result), will they be required to add a short reason or code (e.g. “Benign cause”, “Post-EBRT bounce”)?  
- When they choose “Move to Recall”, will they have to pick a reason or referral type, or is it free text?  
- Can a result be corrected after the card has been moved (e.g. wrong value entered)? If yes, do we allow “Reopen” from Completed?

**Completed**  
- Is the 48h rule fixed, or configurable per practice?  
- Will “completed” patients be visible anywhere (e.g. List → Completed) with a “Reopen” option, or truly gone until the next due date?

**Recall**  
- Will “Discharge” require a date and/or outcome (e.g. back to clinic, deceased)?  
- Where will discharged patients appear (archive list, report only, never again)?

**Thresholds**  
- When a GP changes the personal threshold, must a reason always be selected, or only when the value differs from the protocol?  
- Will reason codes be fixed (e.g. Age-adjusted, Known benign tissue) or configurable?

---

## Analysis: Do more protocols fit in this structure?

**Short answer: yes.** The current design is **status-based**, not protocol-based. The columns (Upcoming, Awaiting, Review Required, Completed, Recall) describe where the patient is in the workflow. Protocol and phase only determine **what** is shown on the card (threshold, frequency, next due) and **which** rules apply (e.g. bounce, consecutive rises). So adding Brachytherapy, ADT, Watchful Waiting, or Active Surveillance does not require new columns or a different board.

**What stays the same**  
- Same five columns and the same List tabs.  
- Same card layout: name, NHS No, protocol+phase badge, sparkline, latest PSA vs threshold, next due or status line.  
- Same Actions pattern: status drives which actions are offered (e.g. Upcoming → blood form; Review → Mark completed / Move to Recall).  
- Same detail modal: identity, pathway, PSA & threshold, key dates, Actions.

**What changes per protocol**  
- **Data:** Protocol dropdown and protocol matrix (frequency by year, normal/alert/recall thresholds, and any special rule).  
- **Labels:** “Protocol” and “Phase” already show the right text from the matrix.  
- **Copy:** In Actions (and optionally on the card), protocol-specific hints where the PRD calls for them (e.g. “Consider EBRT bounce before moving to Recall”).  
- **Optional filter:** A “Protocol” or “Pathway” filter on the board/list so users can restrict to e.g. “Radical Prostatectomy only” if the list is long.

**Edge cases**  
- **Watchful Waiting / Active Surveillance** use different logic (doubling time, rise &gt; 0.75/year). The **card** can still show latest PSA and trend; the **rules** for “Review” vs “Recall” live in the backend or a config table. The UI can stay generic (“Above threshold” / “Rise suggests review”) and only the copy or a small “Protocol note” can explain the rule.  
- **Discharge rules** (e.g. auto-flag at Year 5 for prostatectomy) can be implemented as background logic; the card or List might show a “Eligible for discharge” badge or similar when the rule fires.

**Conclusion**  
The current structure fits all protocols in the PRD. Adding a new protocol is mainly: (1) add a row to the protocol matrix, (2) add the option to the Add Patient wizard and any filters, and (3) add or adjust protocol-specific hint text. No need to redesign the board or the Actions section.
