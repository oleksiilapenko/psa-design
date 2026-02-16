# Cursor skill draft: Build a UX prototype (vanilla-first, demo-ready)

**Status:** Draft. Promote to a real Cursor skill (e.g. `.cursor/skills/ux-prototype/SKILL.md`) once the v2 prototype is approved and this approach is validated.

---

Use this skill when the user asks to **build or refactor a UX/UI prototype** for **stakeholder demos** (click-through, no backend), or when they want to **standardise** how prototypes are built (vanilla-first, simple, no framework bloat).

---

## When to use this skill

- User says they want to build a **UX prototype**, **click-through**, **stakeholder demo**, or **HTML prototype**.
- User wants to keep things **vanilla**, **simple**, or **avoid dependencies**.
- User mentions a **lightweight** library (e.g. Oat) and wants it only if integration is **simple**.
- User wants to **document** or **repeat** a successful prototype-building approach.

---

## Principles (how to build)

1. **Vanilla-first**
   - No framework (no React/Vue/Angular) unless the user explicitly asks.
   - Prefer a **single HTML file** (or very few: one HTML + one JS + one CSS, or inline). No build step (no npm, webpack, Vite) unless the user agrees and it stays minimal.

2. **Optional lightweight library**
   - If the user wants a small UI library (e.g. Oat / oat.ink): use it **only if** integration is trivial (e.g. one `<link>` + one `<script>`, or copying 1–2 files). No complex setup.
   - If integration is complex or blocking: **drop the library** and keep the **same approach**: semantic HTML, minimal CSS, clear structure, good defaults, safe interaction patterns (focus, buttons, forms). Tell the user: “We’re keeping the same philosophy without the dependency.”

3. **Wireframe and data-structure first**
   - Treat designs as **wireframe and data model** input, not only visual spec.
   - Before building views, **define the data model**: entities, fields, and how they relate (e.g. patients, schedule rows, PSA results, status).
   - Map each screen to: (a) which data it reads/writes, (b) which blocks it has (header, form, table, actions). Then implement.

4. **Phased and runnable**
   - Build in **small phases**. Each phase should produce a **runnable** artifact (open the file and use the new part).
   - Do **not** do a big-bang refactor. Prefer: new file or new section → wire data → verify → next phase.
   - If the user has an existing “v1” prototype, **do not modify it** when building v2; keep v2 in a separate file so they can fall back.

5. **Views read from one source of truth**
   - One **in-memory** (or script-loaded) dataset that represents the app state (e.g. list of patients, each with pathway, schedule, status, results).
   - **Views** (board, list, detail, add form) **read** from this dataset and **render**. User actions **update** the dataset, then views re-render. Views do not own business logic; they bind to data.

6. **Purpose: demo and handoff**
   - Goal is **stakeholder demo**: click-through, realistic flow, no backend required. “Spec in HTML” (clear structure, semantic markup) is valuable for handoff to developers later.

---

## Process (step-by-step)

1. **Clarify goal**  
   Demo only? User testing? Handoff? Prefer “demo” unless they say otherwise.

2. **Capture data model**  
   From wireframes or requirements: list entities (e.g. Patient, ScheduleRow, PSAResult), fields, and status/workflow. Write this down (e.g. in a plan or doc).

3. **Choose stack**  
   Default: single HTML file, inline or linked CSS/JS. If they want a lightweight lib (e.g. Oat), try CDN or 1–2 file copy; if it’s not simple, skip and use vanilla with the same principles.

4. **Build in phases**
   - Phase 0: Shell (e.g. header, nav, placeholder content) and a minimal data structure in JS. Runnable.
   - Phase 1: One main flow (e.g. “Add item” form) that writes to the data and maybe shows a simple list or message. Runnable.
   - Phase 2: Detail/view screen that reads from the data and allows edits. Runnable.
   - Phase 3+: Board, list, or other views that read from the same data. Each step runnable.

5. **Keep reference intact**  
   If evolving an existing prototype (e.g. v2), keep the previous version (e.g. `index.html`) untouched; create `index-v2.html` (or similar) so the user can always revert or compare.

---

## Good defaults (interaction and style)

- **Semantic HTML:** Use `<button>`, `<form>`, `<table>`, `<section>`, `<label>`, `<input>` with correct types. Use ARIA where it helps (e.g. `aria-label`, `role` for custom components).
- **Safe interactions:** Buttons and links do one clear thing. Forms validate minimally (e.g. required fields) and give simple feedback. Avoid breaking the back button or leaving the user in a dead state.
- **Styling:** If no library: a small set of CSS classes or element rules (e.g. `.card`, `.btn`, `.table`). Consistent spacing, readable font, clear focus states. If Oat (or similar): use its components as intended.
- **No fake loading:** For a demo, avoid spinners or fake delays unless the user asks. Prefer instant feedback.

---

## What to avoid

- **No framework or build** unless the user explicitly wants it.
- **No refactoring the “v1” file in place** when building a new version; use a new file.
- **No big-bang** “rewrite everything in one go”; use phased, runnable steps.
- **No complex library setup**; if it’s heavy, drop the library and keep the approach vanilla.

---

## Example prompt (for the user)

When you want the agent to use this approach, you can say:

- “Build a UX prototype for [X]. Use the vanilla-first approach: single HTML, no framework, wireframe and data model first, phased build. Keep the existing [v1] file untouched.”
- “We’re adding a new flow to our prototype. Follow the same approach: data model first, then views that read from it, in small runnable steps.”
- “Document how we build prototypes here: vanilla-first, optional Oat if simple, phased and runnable, and create a Cursor skill so we can reuse this.”

---

## Promoting this to a real skill

When v2 is approved and this approach is validated:

1. Create a directory: `.cursor/skills/ux-prototype/` (project) or `~/.cursor/skills/ux-prototype/` (personal).
2. Add `SKILL.md` with the YAML frontmatter and the body above (or a shortened version), for example:

```yaml
---
name: ux-prototype
description: Build or refactor UX prototypes the vanilla-first way — single HTML, wireframe/data first, phased and runnable, optional lightweight lib (e.g. Oat) only if simple. For stakeholder demos.
---
```

3. Optionally add `reference.md` with the full text of this draft and a link to `REBUILD_CLARIFICATION.md` for the PSA v2 context.
