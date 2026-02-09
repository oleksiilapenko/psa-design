# Custom PSA schedule (exploration)

**Goal:** Allow a patient’s PSA monitoring schedule (frequency / interval) to differ from the protocol default, in the same way we allow a custom threshold. Support it in the **Add Patient wizard** and on the **patient detail** page. No implementation in this doc—design only.

---

## How schedule works today

- **Protocol** + **anchor date** (treatment start) drive the schedule:
  - `protocolConfig[protocol].getFrequency(phaseYear)` returns a label such as `'4‑monthly'`, `'6‑monthly'`, `'annual'`.
  - Phase year is derived from the anchor (e.g. “Year 1”, “Year 2”).
- Each patient in **detailData** has:
  - `anchor` (e.g. `'15 Dec 2024'`)
  - `phase` (e.g. `'Year 1'`)
  - `frequency` (e.g. `'4‑monthly'`, `'6‑monthly'`) — **display only**; next-due is stored as `nextDue` (no calculation in the POC).
- **Next due** is stored per patient (`nextDue`); the app does not currently *compute* next due from frequency + last test date. So “custom schedule” would still be expressed as a **frequency label** (and optionally a **next due date**), with any future calculation logic using “protocol default vs custom” the same way threshold uses `protocolThreshold` vs `threshold`.

---

## Parallel: personal threshold

We already support “override protocol default” for threshold:

| Aspect | Threshold | Schedule (proposed) |
|--------|-----------|----------------------|
| **Protocol default** | `protocolThreshold` from protocol config | `frequency` from `getFrequency(phaseYear)` (and `scheduleSummary`) |
| **Effective value** | `threshold` (used in UI and logic) | `frequency` (or a new `effectiveFrequency`) |
| **Override flag** | `personalThresholdSet` (true if user set a different value) | `personalScheduleSet` (or similar) |
| **Wizard** | Step 3: “Protocol default threshold” (read-only) + “Threshold” (editable); hint about override | Step 3: “Protocol schedule” (read-only, e.g. “Year 1 — 4‑monthly”) + “PSA schedule” (editable: same as protocol or override) |
| **Detail page** | Pathway: “Protocol threshold” (read-only) + “Threshold” with edit + * when overridden | Pathway: “Schedule (protocol)” (read-only) + “Monitoring” with edit + * when overridden |

So the **custom schedule** would mirror the **custom threshold** pattern: show protocol default, allow an override, and mark when the patient is on a custom schedule.

---

## Data model (per patient)

Add (or reuse) fields so we can store “protocol says X, but for this patient we use Y”:

- **`protocolFrequency`** (or keep deriving from protocol + phase): the default from the protocol matrix for this phase, e.g. `'4‑monthly'`, `'6‑monthly'`, `'annual'`. Could be set when the patient is created and when protocol/anchor change (or computed when needed).
- **`frequency`**: the **effective** monitoring interval (what we show as “Monitoring” and use for any next-due logic). Default = protocol default; if the user overrides, store the chosen value here.
- **`personalScheduleSet`** (boolean): `true` if the user has explicitly set a schedule that differs from the protocol default (analogous to `personalThresholdSet`).

Optional for a richer model:

- **`customScheduleReason`** (string): free text or code for why the schedule was overridden (e.g. “Patient preference”, “Comorbidity”), similar to a possible “reason for threshold override”.

So in **detailData** we’d have, for each patient, something like:

- `protocolFrequency` — from protocol + phase (or from protocol config at add/edit time).
- `frequency` — what we display and use (protocol default or custom).
- `personalScheduleSet` — true if `frequency !== protocolFrequency` (or explicitly “use custom” in UI).

`nextDue` can stay as today (stored value); when you add **next-due calculation** later, the formula would use `frequency` (and last test date) instead of the protocol default when `personalScheduleSet` is true.

---

## Wizard (Add Patient) — Step 3

Mirror the threshold block:

1. **Protocol schedule (read-only)**  
   - Label: e.g. “Protocol schedule” or “Schedule (protocol)”.  
   - Value: same as today’s “Year X — Y” from Step 2, e.g. “Year 1 — 4‑monthly” (from `getFrequency(phaseYear)` and phase text).  
   - So the user sees what the protocol says for the chosen protocol + anchor.

2. **PSA schedule (editable)**  
   - Label: e.g. “PSA monitoring schedule” or “Monitoring”.  
   - Control: dropdown or radio with a small set of options, e.g.:
     - “Same as protocol” (default)
     - “4‑monthly”
     - “6‑monthly”
     - “Annual”
     - (Optional: “3‑monthly”, “Other” with free text or interval in months.)  
   - Default: “Same as protocol” (so `frequency` = protocol default, `personalScheduleSet` = false).  
   - If the user picks a different option, set `frequency` to that value and `personalScheduleSet` = true when creating the patient.

3. **Hint**  
   - Short line: e.g. “You can override the protocol schedule if this patient needs a different monitoring interval.”

On “Continue” from Step 2, Step 3 would prefill the protocol schedule from the selected protocol and anchor (as we do for threshold), and set the default “PSA schedule” to “Same as protocol”.

---

## Patient detail page (pathway section)

Mirror the threshold pattern in the **Pathway** block:

1. **Schedule (protocol)**  
   - New row or reuse “Schedule (protocol)” in the existing `<details>` “Protocol rules (schedule & PSA guideline)”.  
   - Value: e.g. “Year 1: 4‑monthly; Year 2–5: 6‑monthly; …” from `scheduleSummary`, or a single line “Year X — Y” from `protocolFrequency` / phase.  
   - Read-only; shows what the protocol says.

2. **Monitoring** (effective schedule)  
   - Today we have “Monitoring” showing `d.frequency`.  
   - Make it **editable** when the user clicks “Edit” (same pattern as Threshold):
     - Display: current `frequency` (e.g. “6‑monthly”).
     - When overridden: show an asterisk * and (optional) legend “* Override of protocol schedule”, same idea as threshold.
   - Edit control: dropdown (or radio) with the same options as the wizard: “Same as protocol”, “4‑monthly”, “6‑monthly”, “Annual”, etc.  
   - “Same as protocol” would reset `frequency` to the current protocol default for this phase and set `personalScheduleSet` = false.  
   - Saving updates `detailData[id].frequency` and `detailData[id].personalScheduleSet` (and optionally `protocolFrequency` if we store it and it can change when phase changes).

3. **Where it sits in the UI**  
   - Either keep “Monitoring” in the main Pathway grid and add “Schedule (protocol)” in the collapsible “Protocol rules” section, or add “Schedule (protocol)” next to “Monitoring” (protocol read-only, monitoring editable with *).  
   - Consistency with threshold suggests: one read-only “Protocol schedule” and one editable “Monitoring” with * when overridden.

---

## Cards and list

- **Board cards:** Today they show e.g. “Due 12 Apr 2025 · 4‑monthly”. The “4‑monthly” (or “6‑monthly”) comes from `d.frequency`. So once we store the effective frequency in `frequency`, no change needed except that it may now be a custom value.
- **List:** No schedule column today; if we add one later, it would show `frequency` (protocol or custom). No change required for the exploration.

So: **cards and list keep using `frequency`**; they don’t need to know whether it’s protocol or custom. Only the detail (and wizard) need to show “protocol default” vs “override” and set the flag.

---

## Edge cases and choices

1. **Phase changes (e.g. Year 1 → Year 2)**  
   - Protocol default can change (e.g. 4‑monthly → 6‑monthly).  
   - If `personalScheduleSet` is false, we’d update `frequency` to the new protocol default when phase is recalculated.  
   - If `personalScheduleSet` is true, we keep the custom `frequency` unless the user clears the override (e.g. “Same as protocol”).

2. **Protocol change**  
   - If the user could change protocol on the detail page (not in the POC today), we’d recompute protocol default and either keep custom schedule or reset to protocol when user chooses “Same as protocol”.

3. **Options for “custom” value**  
   - **Fixed list** (4‑monthly, 6‑monthly, annual) is simplest and matches most pathways.  
   - **Extended list** (e.g. 3‑monthly, 12‑monthly) or “Other” with months could be added later if needed.

4. **Next due calculation**  
   - Today, next due is stored (`nextDue`). When you add calculation, it would use:
     - Last test date (or anchor if no test yet).
     - Interval from **effective** `frequency` (e.g. 4 months, 6 months, 12 months).
   - So custom schedule feeds into “when is the next test due?” the same way as protocol schedule, once that logic exists.

---

## Summary

| Where | What to add |
|-------|-------------|
| **Data** | `protocolFrequency` (or derive), keep `frequency` as effective value; add `personalScheduleSet`. |
| **Wizard Step 3** | “Protocol schedule” (read-only); “PSA schedule” (editable: same as protocol / 4‑monthly / 6‑monthly / annual); hint. |
| **Detail – Pathway** | “Schedule (protocol)” (read-only); “Monitoring” editable with * when overridden; save updates `frequency` and `personalScheduleSet`. |
| **Cards / list** | Keep using `frequency` (no change). |

This mirrors the custom-threshold pattern and supports a custom PSA schedule in both the wizard and on the patient page, with workflow order and protocol default clearly visible and overrides explicit and auditable.
