# FigJam diagram: Patient card state machine

Use this to build or import the state machine from [PATIENT_CARD_STATE_MACHINE.md](PATIENT_CARD_STATE_MACHINE.md) into FigJam.

---

## Option A: Import as image (fastest)

1. Open **[Mermaid Live Editor](https://mermaid.live)**.
2. Delete the sample diagram and paste in the contents of **`patient-card-state-machine.mmd`** (in this folder).
3. Use **Actions → Export → PNG** or **SVG**.
4. In **FigJam**: drag the exported file onto the board, or use **+ → Upload** and choose the image.
5. Optionally add a title frame: “Patient card state machine” and a legend (see below).

---

## Option B: Build it in FigJam with shapes

If you prefer to build the diagram natively in FigJam (editable labels, consistent styling):

### 1. States (5 + exit)

Create **rectangles** or **sticky notes** for:

| Label | Colour hint | Short description (add as subtitle or sticky) |
|------|-------------|-------------------------------------------------|
| **Upcoming** | Blue | PSA due next 4 weeks. Mock: blood form, reminder |
| **Awaiting Results** | Orange | Blood taken, waiting result. Mock: Chase result |
| **Review Required** | Red | PSA above threshold. Clinician decides |
| **Completed** | Green | PSA ≤ threshold. 48h / next due not impl. |
| **Recall** | Purple | Back to clinic. Mock: Discharge |
| **Discharged** (or “Off board”) | Grey | Archive / off board |

### 2. Arrows and labels

Draw **arrows** between states and add **text labels** on the arrows:

| From | To | Arrow label |
|------|-----|-------------|
| (start) | Upcoming | Add Patient |
| Upcoming | Awaiting Results | Test done / Blood taken *(not in UI)* |
| Awaiting Results | Review Required | Move to Review Required (manual) |
| Awaiting Results | Completed | Mark as completed (manual) |
| Review Required | Completed | Mark as completed (manual) |
| Review Required | Recall | Move to Recall (manual) |
| Recall | Discharged | Discharge (mock) |
| Completed | Upcoming | Next due *(not impl.)* |

### 3. Legend (optional)

Add a sticky or text frame:

- **Implemented (manual):** Awaiting ↔ Review Required ↔ Completed ↔ Recall (3-dot menu; Kanban only; list/detail not synced yet).
- **Not in UI:** Upcoming → Awaiting (“Test done”).
- **Not implemented:** Completed → leave board (48h), Completed → Upcoming (next due).
- **Mock (toast only):** Generate blood form, Send reminder, Chase result, Discharge, Print summary.

### 4. Layout suggestion

- Top: **Upcoming**.
- Middle row: **Awaiting Results** (left), **Review Required** (centre), **Completed** (right).
- Bottom row: **Recall** (left), **Discharged** (right).
- Use connector lines or arrows; FigJam will keep them attached when you move shapes.

---

## File reference

- **Mermaid source:** `docs/patient-card-state-machine.mmd` — use in Mermaid Live to export PNG/SVG for FigJam (Option A).
- **Spec:** `docs/PATIENT_CARD_STATE_MACHINE.md` — full states, transitions, mock vs implemented, and open questions.
