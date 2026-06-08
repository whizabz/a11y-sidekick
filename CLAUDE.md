# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A11ySidekick is a **single static HTML file** ([`index.html`](index.html)) with no build step, no dependencies beyond Google Fonts and Phosphor icons, and all CSS/JS inline. To preview, open the file directly in a browser or serve with any static file server:

```bash
npx serve .        # then open http://localhost:3000/index.html
python -m http.server 5500
```

## Repository layout

| File | Role |
|---|---|
| `index.html` | Main app (~3,300 lines) — the complete product |
| `wcag-guidelines.html` | Earlier reference-only version; superseded by the WCAG by Role panel in `index.html` |
| `a11y-sidekick-spec.md` | Original build spec |
| `how-i-built-this.html` | Companion page — build story / write-up |

Deployed on GitHub Pages at `https://whizabz.github.io/a11y-sidekick/` (serves `index.html` at the site root).

## Architecture

Everything lives in `index.html` in this order:

```
<style>    Kami design tokens + all component CSS
<body>
  .page-header
  .mode-bar          Screen Checklist | WCAG by Role  (role="tablist")
  #panel-filter      Screen Checklist mode
    #filter-card     Initial form — hidden after first Generate
    #checklist-output  Dynamically rendered by renderChecklist()
  #panel-reference   WCAG by Role mode (static HTML, 4 tabs)
  #edit-modal        <dialog> for editing selections post-generation
<script>   All logic — no modules, plain ES2020
```

## Data model

The `CRITERIA` array (defined in `<script>`) holds every WCAG criterion as a plain object:

```js
{
  id:       '1.4.3',          // WCAG number — also the dedup key
  title:    'Contrast (Minimum)',
  layer:    'element',        // 'always' | 'structure' | 'element'
  role:     'design',         // 'design' | 'ddev' | 'dev' | 'content'
  triggers: ['text','button'],// which chip values surface this criterion
  isNew:    true,             // optional — WCAG 2.2 additions only
  rule:     '...',
  do:       '...',
  dont:     '...',
  note:     '...',            // optional
  // exactly one of these, or none:
  doc:      '...',            // design→dev handoff note
  impl:     '...',            // dev implementation/test note
  owns:     '...',            // content team responsibility
}
```

## Layer model

- **Always (7 criteria)** — `triggers: []`, included in every checklist regardless of selections.
- **Structure (12 criteria)** — triggered by layout chips: `navigation`, `sticky`, `nonlinear`, `overlay`, `timeout`, `orientation`.
- **Element (37 criteria)** — triggered by element chips: `text`, `button`, `link`, `formfield`, `selection`, `image`, `icon`, `media`, `notification`, `datadisplay`, `gesture`.

A criterion can appear in multiple trigger lists; `getApplicableCriteria()` deduplicates by `id`.

## Runtime state

```js
const state = {
  mode:             'checklist',       // 'checklist' | 'reference'
  selectedStructure: new Set(),        // active structure chip values
  selectedElements:  new Set(),        // active element chip values
  triggerStates:     new Map(),        // "criterionId:trigger" → true when reviewed
  activeRole:       'design',          // active role tab in checklist
  activeTrigger:     null,             // active trigger filter chip value, or null
  generatedIds:      [],               // ids in current checklist, in order
  lastScreenName:    '',
};
```

`triggerStates` persists across regenerations — criteria removed from the new list simply fall out of the render; their states are kept in the Map if they return.

## Key functions

| Function | What it does |
|---|---|
| `getApplicableCriteria(struct, elem)` | Filters CRITERIA by layer/triggers, deduplicates by id |
| `renderChecklist(criteria)` | Rebuilds `#checklist-output` HTML — progress card, role tabs, trigger filter chips, criteria cards |
| `renderChecklistCard(criterion)` | Renders one card with title, do/dont, per-chip review checkboxes |
| `generateChecklist()` | Reads form, updates state, calls renderChecklist, hides `#filter-card` |
| `openEditModal()` | Pre-populates `#edit-modal` from current state and calls `.showModal()` |
| `toggleTriggerState(id, trigger, checked)` | Sets reviewed state for a criterion chip, re-renders checklist |
| `buildExportSvg()` | Builds SVG string — all roles, criterion title + review chips per card |
| `downloadChecklistSvg()` | Downloads `.svg` file for drag-and-drop into Figma |
| `setMode(mode)` | Switches between Screen Checklist and WCAG by Role panels |
| `switchTab(name)` | Switches tabs inside the WCAG by Role reference panel |

All click handling for `#checklist-output` (state buttons, role filter, trigger filter, edit icon) is delegated to a single listener using `.closest()`.

## Design system (Kami)

Follow `KAMI-DESIGN.md`. Primary tokens in `index.html`:

```
--brand / --brand-light             ink-blue accent (#1B365D) — only chromatic color
--parchment / --ivory / --warm-sand surface hierarchy
--near-black / --dark-warm / --olive / --stone   text (four levels)
--border / --border-soft / --ring-warm / --ring-deep
--tag-light / --tag-standard / --tag-mid          solid tag tints (no rgba backgrounds)
--focus (#3898ec)                   focus rings only
```

Typography: **Newsreader** serif 400/500 for English body + headings. **Inter** sans for UI only (buttons, chips, labels). No italic. Line-height ≤ 1.55 for body. Serif headings weight 500 only.

Semantic blocks (Do/Don't/doc/impl) use warm solid hex — not a second accent color.

## Intentional deviations from spec

The original `a11y-sidekick-spec.md` describes some behaviours that have since been deliberately changed:

- **Guidance always visible** — the spec described a "Show guidance" expand/collapse toggle; cards now always show Do/Don't inline.
- **Progress bar removed** — the spec described a progress bar + done/NA/open stat pills; these were removed to keep the progress card compact.
- **Edit modal instead of collapsed summary** — the spec described a collapsed "Screen: X" card with an Edit button; this was replaced with a `<dialog>` edit modal triggered by a pencil icon in the progress card.
- **Role tabs replace "All" view** — the checklist defaults to the first role tab (Design), not an "All" view.
- **Trigger filter chips in progress card** — the selection chips (Has overlay, Button, etc.) live inside the progress card and double as trigger filters, replacing the separate trigger filter bar that was below the role tabs.
- **SVG export** — Download menu → Download SVG generates a checklist summary file for drag-and-drop into Figma.
