# Schedule: Phase, Threshold, MOT, LFT (custom per-patient)

**Status:** Implemented in prototype (wizard Step 3 and detail view).

## Goal

The protocol defines a **default schedule** (phases with year ranges, schedule, threshold). During **Add Patient (wizard)** the user can build a **patient-specific schedule**: override any phase (edit year range, schedule, threshold), add phases at the end, and delete the last phase. The result is stored per patient and shown on the detail view.

## Column structure

| Column   | Description |
|----------|-------------|
| **Phase** | Year range (e.g. 2–5). Editable: users can change ranges, split or merge. |
| **Threshold** | PSA threshold (ng/mL) for that phase. Default from protocol; editable per row. |
| **Schedule** | PSA monitoring interval: 3‑monthly, 4‑monthly, 6‑monthly, Annual. |
| **MOT** | Checkbox. Full name in tooltip on hover (e.g. Monitoring Other Test). |
| **LFT** | Checkbox. Full name in tooltip on hover (e.g. Liver Function Test). |

- UI: labels are short (**MOT**, **LFT**); full names are in a `title` (tooltip) on hover.
- Clinical meaning of MOT/LFT to be confirmed with clinical lead; prototype uses standard abbreviations.

## Add phase

- Button **Add phase** appends one row at the end.
- New phase is **one year only**: `yearFrom = yearTo = lastYear + 1` (e.g. if last phase is Year 6–10, new row is Year 11–11).
- Defaults: schedule 6‑monthly, threshold = protocol default, MOT/LFT unchecked, `isProtocol: false`.

## Delete phase

- **Rule:** Only the **last** phase in the list can be deleted (whether it was from the protocol or added by the user). This avoids gaps in the list.
- Delete button is shown only on the last row, and only when there is more than one row (at least one phase must remain).

## Phase editing

- Users can edit any phase: change year ranges (e.g. Year 2–5 → Year 2–4), so protocol rows are not read-only.
- Overlapping or non-contiguous year ranges are allowed in the prototype; validation can be added later.

## Data model (per patient)

- **scheduleRows:** `Array<{ yearFrom, yearTo, schedule, threshold, mot, lft, isProtocol }>`
  - `yearFrom`, `yearTo`: numbers (e.g. 1–1, 2–5, 6–10).
  - `schedule`: e.g. `'3-monthly'`, `'4-monthly'`, `'6-monthly'`, `'annual'`.
  - `threshold`: string (ng/mL).
  - `mot`, `lft`: boolean.
  - `isProtocol`: true for rows initialised from protocol, false for user-added rows (used only for display/audit; delete rule is “last row only”).
- Backward compatibility: patients without `scheduleRows` get them **derived** from `protocolPhases[protocol]` + `scheduleByYear` + `threshold` when loading (wizard or detail view).
- For existing logic (card phase, next due): `scheduleByYear` and `threshold` are still derived from `scheduleRows` (e.g. from the row that matches the current phase year).

## Caveats (not enforced in v1)

- Year-range overlap or gaps: no automatic validation.
- “Current phase” for the patient is derived from anchor date; the first matching row (by phase year) is used for threshold and schedule when multiple rows could cover the same year.
- MOT/LFT: checkboxes only; no backend or clinical workflow in prototype.

## References

- Wizard: [index.html](index.html) Step 3, `#wizard-protocol-table`, `buildWizardProtocolTable`, `getWizardScheduleRows`.
- Detail: `#detail-protocol-tbody`, populated from `scheduleRows` when present.
- Protocol presets: `protocolPhases`, `protocolConfig` in index.html.
