# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A11ySidekick is a **single static HTML file** (`a11y-sidekick.html`) with no build step, no dependencies beyond Google Fonts, and all CSS/JS inline. To preview, open the file directly in a browser or serve with any static file server:

```bash
npx serve .        # then open http://localhost:3000/a11y-sidekick.html
python -m http.server 5500
```

## Architecture

Everything lives in one file in this order:

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
  itemStates:        new Map(),        // criterionId → 'checked'|'na' (open = absent)
  activeRole:       'design',          // active role tab in checklist
  activeTrigger:     null,             // active trigger filter chip value, or null
  generatedIds:      [],               // ids in current checklist, in order
  lastScreenName:    '',
};
```

`itemStates` persists across regenerations — criteria removed from the new list simply fall out of the render; their states are kept in the Map if they return.

## Key functions

| Function | What it does |
|---|---|
| `getApplicableCriteria(struct, elem)` | Filters CRITERIA by layer/triggers, deduplicates by id |
| `renderChecklist(criteria)` | Rebuilds `#checklist-output` HTML — progress card, role tabs, trigger filter chips, criteria cards |
| `renderChecklistCard(criterion)` | Renders one card with meta pills, title, ✓/NA buttons, do/dont, extra block |
| `generateChecklist()` | Reads form, updates state, calls renderChecklist, hides `#filter-card` |
| `openEditModal()` | Pre-populates `#edit-modal` from current state and calls `.showModal()` |
| `toggleItemState(id, nextState)` | Toggles done/na/open, re-renders checklist |
| `setMode(mode)` | Switches between Screen Checklist and WCAG by Role panels |
| `switchTab(name)` | Switches tabs inside the WCAG by Role reference panel |

All click handling for `#checklist-output` (state buttons, role filter, trigger filter, edit icon) is delegated to a single listener using `.closest()`.

## Design system (Kami tokens)

Primary tokens used throughout — do not hardcode colours:

```
--bg / --surface / --surface-alt    page + card backgrounds
--accent (#1B365D) / --accent-mid / --accent-soft    primary blue scale
--ink / --ink-mid / --ink-mute      text hierarchy
--border / --border-soft            dividers
--do-bg/border/ink   green   (✓ Do blocks)
--dont-bg/border/ink red     (✗ Don't blocks)
--doc-bg/border/ink  amber   (Document for dev blocks)
--impl-bg/border/ink green-grey (Implement & test blocks)
--new-bg/border/ink  blue    (WCAG 2.2 New badges)
--split-bg/border/ink purple (Content "Your part" blocks)
```

Fonts: **Newsreader** serif (headings, titles, serif UI) + **Inter** sans (body, labels, chips).

## Intentional deviations from spec

The original `a11y-sidekick-spec.md` describes some behaviours that have since been deliberately changed:

- **Guidance always visible** — the spec described a "Show guidance" expand/collapse toggle; cards now always show Do/Don't inline.
- **Progress bar removed** — the spec described a progress bar + done/NA/open stat pills; these were removed to keep the progress card compact.
- **Edit modal instead of collapsed summary** — the spec described a collapsed "Screen: X" card with an Edit button; this was replaced with a `<dialog>` edit modal triggered by a pencil icon in the progress card.
- **Role tabs replace "All" view** — the checklist defaults to the first role tab (Design), not an "All" view.
- **Trigger filter chips in progress card** — the selection chips (Has overlay, Button, etc.) live inside the progress card and double as trigger filters, replacing the separate trigger filter bar that was below the role tabs.
