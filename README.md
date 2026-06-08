# A11ySidekick

A focused WCAG 2.2 AA checklist for designers, developers, and content teams — delivered as a single, zero-dependency HTML file ([`index.html`](index.html)).

**[→ Open A11ySidekick](https://whizabz.github.io/a11y-sidekick/)**

---

## What it does

Most accessibility checklists hand you all 55 WCAG 2.2 AA criteria at once. A11ySidekick only shows you the ones that apply to the screen you're reviewing.

Tell it what's on your screen — its structure and UI elements — and it filters the full criteria set down to exactly what's relevant. Every criterion is assigned to the team that owns it: **Design**, **Design → Dev**, **Dev Only**, or **Content**.

---

## How to use it

1. **Name your screen** — e.g. "Checkout – Payment step"
2. **Select screen structure** — does it have a sticky header? An overlay? A timeout?
3. **Select elements** — buttons, form fields, images, media, etc.
4. **Generate** — your focused checklist appears, grouped by role
5. **Filter by selection** — click any chip (e.g. "Form field") to see only criteria relevant to that element
6. **Switch role tabs** — focus on Design, Design → Dev, Dev Only, or Content criteria independently
7. **Mark items** — check off each selection chip as you review a criterion
8. **Edit anytime** — the pencil icon reopens your selections in a modal
9. **Export** — download menu: paste editable frames into Figma, copy a text fallback, or download CSV

The **WCAG by Role** tab gives you the complete reference for all 55 criteria, with Do/Don't guidance, handoff notes, and implementation details — organised by POUR principle within each role.

---

## Key features

- **Filtered checklists** — 55 criteria narrowed to only what applies to your screen
- **Role-based grouping** — Design / Design→Dev / Dev Only / Content
- **Trigger chips** — see which of your selections triggered each criterion, and filter by them
- **X/Y Done counter** — NA items reduce the denominator so your progress reflects reviewable work
- **Edit modal** — change screen name or selections at any time without losing your checked states
- **Full WCAG reference** — all criteria with Do/Don't, document-for-handoff, and implement-and-test guidance
- **Zero build step** — single HTML file, no install; works offline except Figma frame export (loads a small CDN library on demand)
- **Figma export** — paste editable frames (criterion, title, review chips per card, all roles) or a compact text fallback; CSV download also available
- **Build journal** — [how the app was made](how-i-built-this.html)

---

## Running locally

No build step needed. Open the file directly or serve it:

```bash
npx serve .
# open http://localhost:3000/index.html

# or open index.html directly in a browser
```

---

## WCAG coverage

Covers all **55 WCAG 2.2 AA criteria** including the six criteria new in 2.2:

| New in 2.2 | Criterion |
|---|---|
| 2.4.11 | Focus Not Obscured (Minimum) |
| 2.4.12 | Focus Not Obscured (Enhanced) |
| 2.5.7 | Dragging Movements |
| 2.5.8 | Target Size (Minimum) |
| 3.2.6 | Consistent Help |
| 3.3.7 | Redundant Entry |
| 3.3.8 | Accessible Authentication (Minimum) |

---

## Built by

Vibe Coded with ❤️ by [Abhimanyu Sirothia](https://x.com/whizabz)
