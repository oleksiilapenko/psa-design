# Copy and UI copy suggestions

Review dated from UI standardisation pass (patient card, wizard, detail). Apply as needed for consistency and clarity.

---

## Wizard

- **Step 1**
  - ✅ Done: "Enter NHS number and search" (was "Search by NHS Number").
  - ✅ Done: "Confirm patient" (was "Is this the correct patient?").
  - Consider: Add a short line under the Search button: "Demographics will appear here after search."

- **Step 2**
  - ✅ Done: Step label shortened to "Select pathway and treatment start date. Optionally add diagnoses."
  - ✅ Done: Diagnoses hint now states that suggestions "come from the same list as in patient detail" (aligns shared datalist).
  - ✅ Done: Anchor label "Date of surgery or treatment start" (consistent with label in copy).
  - Consider: Protocol help could start with "Guideline: " when showing threshold text for screen readers.

- **Step 3**
  - ✅ Done: Subheadings "5‑alpha reductase inhibitor", "Threshold", "Last known PSA" for form blocks.
  - ✅ Done: "Protocol default" for readonly threshold; "Threshold (ng/mL)" and "PSA value (ng/mL)" / "Date of result" to mirror detail view labels.
  - ✅ Done: 5‑ARI hint mentions "Same options as in patient detail."

---

## Patient card (Board)

- Column notes are already clear. Optional tweaks:
  - **Upcoming**: "Due dates come from treatment start and protocol" could be "Due dates are calculated from treatment start and protocol."
  - **Awaiting**: "When the result arrives, the system moves the card" could be "When the result is entered, the card moves" (avoids implying full automation if not true).
  - **Completed**: Consider shortening to "PSA at or below threshold. Patient reappears in Upcoming at next due date. See List → Completed for full list."

---

## Detail view

- **Section titles**
  - ✅ Done: "Patient", "Clinical context", "Pathway", "PSA & threshold", "Key dates", "Actions" kept; typography bumped (weight).
- **Protocol rules**
  - Summary "Protocol rules (schedule & PSA guideline)" is clear. Alternative: "Schedule and PSA guideline (by protocol)."
- **Threshold legend**
  - Current: "Δ = PSA − threshold (↑ above, ↓ below). * Override of protocol default". Consider splitting: one line for Δ, one for *.

---

## List view

- **Tab**
  - ✅ Done: List tab renamed to "Review required" to match the Board column "Review Required" and status badge.
- **Table header**
  - "Last PSA (ng/mL)" is good. "Δ vs threshold" with legend is clear.
- **Legend**
  - "* Override of protocol default" could sit under the table with the Δ legend in one block.

---

## Shared data (dropdowns / autocomplete)

- **Diagnoses**
  - Single source: `<datalist id="diagnosis-list">` in the wizard; both wizard (wizard-diagnoses) and detail (detail-diagnoses-input) use `list="diagnosis-list"`. Keep one datalist in the DOM (e.g. in wizard markup or moved to a shared fragment) so options stay in sync.
- **5‑ARI**
  - Wizard: radios (None, Finasteride, Dutasteride). Detail: `<select>` with same three options. Values match (none, finasteride, dutasteride). No change needed; hints now mention that options are the same.

---

## Buttons and actions

- **Wizard**
  - "Continue" / "Back" / "Add patient" are clear.
  - Discard modal: "Discard" and "Cancel" are standard.
- **Detail**
  - "Edit" / "Save" per field is clear. "Save result" (PSA) and "Save threshold" are clear.
- **Card**
  - Dropdown title could be "Actions" (if not already) for consistency with detail "Actions" section.

---

## Accessibility and labels

- Ensure every form control has a visible or sr-only label (step 3 "Protocol default" now has aria-label on the readonly input).
- Phase output "Current phase: Year X — frequency" is good; keep as live region if step 2 is dynamic.
