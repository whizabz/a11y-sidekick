# A11ySidekick — Build Spec
## For Claude Code

---

## What this is

**A11ySidekick** is a single-file HTML tool that helps design teams check WCAG 2.2 AA compliance per Figma screen. It has two modes:

1. **Screen Checklist** — filter 56 criteria by what's on a screen, get a focused checklist, check items off
2. **Full Reference** — browse all criteria by role (Design / Design→Dev / Dev Only / Content)

The existing file `wcag-guidelines.html` has the full reference content already built. The task is to add the filter + checklist system on top of it.

---

## Design system

**Kami design system** — apply consistently throughout.

```
--bg:           #f5f4ed      (page background)
--surface:      #faf9f4      (card background)
--surface-alt:  #eeede4      (subtle alt background)
--accent:       #1B365D      (primary blue)
--accent-soft:  #d6dfe8      (light blue tint)
--ink:          #1a1917      (primary text)
--ink-mid:      #3d3c38      (secondary text)
--ink-mute:     #7a7870      (muted text)
--border:       #d9d8d0
--border-soft:  #e8e7df

Role colours:
--do-bg/#do-border/#do:       green  (#eef6f1 / #b3d4bf / #1a4a2e)
--dont-bg/#dont-border/#dont: red    (#fdf0ef / #e8b4b0 / #5c1c1c)
--doc-bg/#doc-border/#doc-ink: amber (#f5f0e8 / #d4c4a0 / #5a4a28)
--impl-bg/#impl-border/#impl-ink: green-grey (#f0f4f0 / #a8c4a8 / #2a4a2a)
--new-bg/#new-border/#new-ink: blue  (#f0f4fa / #a8bcd8 / #1B365D)
--split-bg/#split-border/#split-ink: purple (#f8f4ff / #c4aee8 / #4a2a7a)

Fonts: Newsreader serif (headings) + Inter sans (body) via Google Fonts
Shadows: ring 0 0 0 1px rgba(27,54,93,0.07), 0 2px 8px rgba(27,54,93,0.06)
```

---

## Architecture

Single HTML file. No build step, no dependencies, no external JS. Everything inline.

```
index.html
├── <style>          Kami tokens + all component styles
├── <body>
│   ├── .page-header
│   ├── .mode-bar    [Screen Checklist] [Full Reference]
│   ├── #panel-filter   (Screen Checklist mode)
│   │   ├── Screen name input
│   │   ├── Layer 1 callout (always-on, informational)
│   │   ├── Layer 2 chips (screen structure)
│   │   ├── Layer 3 chips (elements)
│   │   ├── [Generate checklist] [Clear] buttons
│   │   └── #checklist-output (dynamically rendered)
│   └── #panel-reference  (Full Reference mode)
│       └── tabs: Design / Design→Dev / Dev Only / Content
└── <script>         All logic inline
```

---

## The data model

Every criterion is an object. The JS array `CRITERIA` holds all 56. Structure:

```js
{
  id:       '1.4.3',           // WCAG criterion number (string)
  title:    'Contrast (Minimum)',
  layer:    'element',         // 'always' | 'structure' | 'element'
  role:     'design',          // 'design' | 'ddev' | 'dev' | 'content'
  triggers: ['text','button'], // which chips surface this criterion
                               // empty array [] for layer:'always'
  isNew:    true,              // optional, WCAG 2.2 additions only
  rule:     '...',             // one-paragraph plain-language explanation
  do:       '...',             // Do this
  dont:     '...',             // Don't do this
  note:     '...',             // optional exception/clarification
  // role-specific extras (only one per criterion):
  doc:      '...',             // Design→Dev: what to document for handoff
  impl:     '...',             // Dev: what to implement and test
  owns:     '...',             // Content: your specific responsibility
}
```

---

## Layer model

### Layer 1 — Always On (7 criteria)
Apply to every screen regardless of content. Always included in any checklist.

| ID | Title | Role |
|----|-------|------|
| 2.4.2 | Page Titled | dev |
| 3.1.1 | Language of Page | dev |
| 3.2.1 | On Focus | dev |
| 3.2.2 | On Input | dev |
| 2.4.1 | Bypass Blocks | dev |
| 3.2.3 | Consistent Navigation | design |
| 3.2.4 | Consistent Identification | design |

### Layer 2 — Screen Structure (12 criteria)
Triggered by yes/no structural properties of the screen.

| Chip label | chip value | Triggers these criterion IDs |
|------------|------------|-------------------------------|
| Has navigation | `navigation` | 2.4.5, 3.2.6, 2.4.4, 3.3.4 |
| Has sticky element | `sticky` | 2.4.11, 1.3.2, 1.4.10 |
| Non-linear layout | `nonlinear` | 1.3.2, 2.4.3, 1.4.10 |
| Has overlay (modal, drawer, tooltip) | `overlay` | 1.4.13, 2.1.2, 2.4.3, 3.3.8, 4.1.2 |
| Has timeout / session | `timeout` | 2.2.1 |
| Orientation-sensitive | `orientation` | 1.3.4 |

Note: 1.3.2 and 2.4.3 appear in multiple structure triggers — deduplicate.

### Layer 3 — Elements (37 criteria)
Triggered by element types present on the screen.

| Chip label | chip value | Triggers these criterion IDs |
|------------|------------|-------------------------------|
| Text | `text` | 1.4.3, 1.4.1, 1.3.3, 1.3.1, 1.3.2, 2.4.6, 3.1.2, 1.4.4, 1.4.12 |
| Button | `button` | 1.4.3, 1.4.1, 1.4.11, 2.4.7, 2.5.8, 1.3.3, 2.1.1, 2.4.3, 2.5.3, 4.1.2, 1.4.4, 1.4.12, 2.5.2 |
| Link | `link` | 1.4.3, 1.4.1, 2.4.7, 2.5.8, 1.3.3, 2.4.4, 2.1.1, 2.4.3, 2.5.3, 4.1.2, 1.4.4, 1.4.12, 2.5.2, 3.1.2 |
| Form Field | `formfield` | 1.4.3, 1.4.1, 1.4.11, 2.4.7, 2.5.8, 3.3.2, 1.3.3, 1.3.1, 1.3.5, 2.1.1, 2.4.3, 2.4.6, 2.5.3, 3.3.1, 3.3.3, 3.3.4, 3.3.7, 4.1.2, 1.4.4, 1.4.12 |
| Selection Control | `selection` | 1.4.1, 1.4.11, 2.4.7, 2.5.8, 3.3.2, 1.3.3, 1.3.1, 2.1.1, 2.4.3, 2.5.3, 3.3.1, 3.3.3, 3.3.7, 4.1.2, 2.5.2 |
| Image / Illustration | `image` | 1.4.5, 1.3.3, 1.1.1 |
| Icon | `icon` | 1.4.11, 2.5.8, 1.3.3, 1.1.1, 2.5.3, 4.1.2 |
| Media | `media` | 2.3.1, 1.4.2, 2.2.2, 1.1.1, 1.2.1, 1.2.2, 1.2.3, 1.2.4, 1.2.5 |
| Notification | `notification` | 1.4.1, 1.4.3, 3.3.1, 3.3.3, 4.1.3, 1.4.4, 1.4.12 |
| Data Display | `datadisplay` | 1.3.1, 1.3.2, 2.4.6, 1.4.3, 1.4.11, 1.1.1 |
| Interactive Gesture | `gesture` | 2.3.1, 2.5.8, 2.1.1, 2.1.4, 2.2.2, 2.5.1, 2.5.4, 2.5.7, 2.5.2, 2.4.7, 4.1.2 |

---

## All 56 criteria — full content

Use this as the source of truth for the `CRITERIA` array. Every field is specified.

### Layer: always

**2.4.2 — Page Titled** · role: dev
- rule: Every page must have a descriptive title that identifies its purpose. Screen readers announce the page title on load — it's how users confirm they've landed in the right place.
- do: Set a unique, descriptive title for every page — format: "Page Name — Product Name"
- dont: Use the same title on every page or leave it as just the app name
- impl: In SPAs, update document.title on every route change. Format: specific context first, then product name.

**3.1.1 — Language of Page** · role: dev
- rule: The primary language of each page must be declared in the HTML lang attribute. Screen readers use this to select the correct pronunciation engine — the wrong setting produces garbled output.
- do: Set the correct lang attribute on the html element of every page — e.g. lang="en", lang="hi", lang="de"
- dont: Leave the lang attribute missing or default to lang="en" on non-English pages
- impl: Every HTML document must have <html lang="xx"> with a valid BCP 47 language tag. In multi-language apps, set lang dynamically on route change to match the current locale.

**3.2.1 — On Focus** · role: dev
- rule: When a UI element receives focus, it must not automatically trigger a context change — no navigation, no form submission, no content shifting. Focus means "I'm here," not "go."
- do: Trigger context changes only on explicit user actions — click, Enter key, or Space bar
- dont: Attach navigation or modal-open logic to focus events alone
- impl: Audit all focus and blur event listeners. None should trigger navigation, form submission, or significant layout changes.

**3.2.2 — On Input** · role: dev
- rule: Changing a form control's value must not automatically trigger a context change unless the user was warned in advance.
- do: Require an explicit submit action after form input changes; if auto-advance is needed, warn the user in the label
- dont: Auto-submit forms or auto-navigate when a user selects a radio button or changes a dropdown
- impl: Audit all change and input event listeners on form controls. None should trigger navigation or submission.

**2.4.1 — Bypass Blocks** · role: dev
- rule: Keyboard users must be able to skip past repeated content blocks to reach the main content directly. Without this, every page requires dozens of Tab presses before reaching actual content.
- do: Implement a "Skip to main content" link as the very first focusable element on every page
- dont: Hide or remove skip links because they look odd visually — make them visible on focus only
- impl: Add <a href="#main-content">Skip to main content</a> as first element in <body>. Style off-screen by default, visible on :focus.

**3.2.3 — Consistent Navigation** · role: design
- rule: Navigation that appears across multiple screens must appear in the same place and order every time. Inconsistency forces users to relearn the interface on every screen.
- do: Lock navigation components in your design system; don't reorder items between screens
- dont: Move the back button or rearrange nav items based on what "feels right" for a specific screen

**3.2.4 — Consistent Identification** · role: design
- rule: Components that do the same thing must look the same and be named the same across the product. A search icon that changes between screens creates unnecessary confusion.
- do: Build a design system where every component has one visual form and one consistent name
- dont: Use "Submit," "Send," and "Go" interchangeably for the same action across different screens

---

### Layer: structure

**2.4.5 — Multiple Ways** · role: design · triggers: navigation
- rule: Users should be able to reach any screen through more than one path. Relying on a single navigation route creates dead ends for users who navigate differently.
- do: Design search, breadcrumbs, sitemaps, or related links as additional pathways to key screens
- dont: Design a flow where the only way to reach a screen is through one specific sequence

**3.2.6 — Consistent Help** · role: design · triggers: navigation · isNew: true
- rule: If help is available (chat, FAQ, support link), it must appear in the same location on every screen. Users who need help shouldn't have to hunt for it differently each time.
- do: Fix the position of help entry points in your layout grid — consistent across all screens
- dont: Show a help icon in the header on some screens and in the footer on others

**2.4.4 — Link Purpose (In Context)** · role: content · triggers: navigation, link
- rule: Link labels must make sense in context. Screen reader users navigate by scanning a list of all links — "Read more" repeated six times tells them nothing.
- do: Write link labels that describe the destination — "Download the Q3 report," "Book a 30-minute call"
- dont: Use "Read more," "Click here," or "Learn more" as standalone link text
- owns: Audit all CTAs and inline links. Every link must be self-describing. If context is needed to understand a link, write it into the link text itself.

**3.3.4 — Error Prevention** · role: ddev · triggers: navigation, formfield
- rule: For legal, financial, or irreversible actions — design must include a review step, confirmation dialog, or undo mechanism.
- do: Design a confirmation step for every high-stakes or irreversible action
- dont: Let destructive or financial actions trigger immediately on a single tap without confirmation
- doc: Confirmation dialog or review screen design for every irreversible or high-stakes action; specify what can be changed at the review step

**2.4.11 — Focus Not Obscured (Minimum)** · role: design · triggers: sticky · isNew: true
- rule: When an element receives focus, it must not be completely hidden by other content. Sticky headers, floating toolbars, cookie banners, and modals are common culprits.
- do: Check that focused elements remain at least partially visible when sticky UI is present on screen
- dont: Design a sticky header that covers the top row of interactive elements when the user scrolls

**1.3.2 — Meaningful Sequence** · role: ddev · triggers: sticky, nonlinear, text, datadisplay
- rule: Screen readers read in DOM order, not visual order. If your layout has columns, overlapping elements, or non-linear flow — dev needs the intended reading sequence documented.
- do: Add numbered reading order annotations to any layout that isn't strictly top-left to bottom-right
- dont: Design multi-column layouts without specifying the intended reading order
- doc: Numbered reading order annotation on any non-linear layout — every screen with columns, cards, or overlapping elements

**1.4.10 — Reflow** · role: dev · triggers: sticky, nonlinear
- rule: At 400% browser zoom, content must reflow into a single column without requiring horizontal scrolling.
- do: Implement responsive layouts that work at 320px wide; use flexbox or CSS grid with wrapping
- dont: Use fixed-width containers or overflow-x scroll as a layout solution
- impl: Test at 400% zoom on a 1280px screen. All content should be readable in a single column without horizontal scrolling.

**2.4.3 — Focus Order** · role: ddev · triggers: nonlinear, overlay, button, link, formfield, selection, navigation, gesture
- rule: The order in which a keyboard user tabs through elements must match the logical reading and interaction order. You define the sequence; dev implements it in the DOM.
- do: Add numbered tab order annotations to every screen with modals, multi-column layouts, or custom flows
- dont: Assume tab order will match visual order — CSS layout often doesn't match DOM order
- doc: Numbered tab order annotation on every screen with non-linear layouts, modals, or multi-step flows

**1.4.13 — Content on Hover or Focus** · role: ddev · triggers: overlay
- rule: Tooltips and hover cards must be dismissable without moving focus, hoverable without disappearing, and persistent until dismissed.
- do: Specify dismissal method, hover persistence, and how long content stays visible
- dont: Design tooltips without specifying their behavior — "appears on hover" is not enough
- doc: For every hover/focus-triggered element: trigger condition, dismissal method, hover persistence

**2.1.2 — No Keyboard Trap** · role: dev · triggers: overlay
- rule: If keyboard focus can move into a component, it must be able to move out. Modal dialogs are the classic trap. Focus must be managed correctly and returned to the trigger on close.
- do: Implement focus trapping inside open modals, Escape to close, and focus return to trigger on close
- dont: Open modals or drawers without managing focus — keyboard users will be stuck
- impl: Use a focus trap utility for all modals. On open: move focus to first focusable element. On close: return focus to trigger. Tab/Shift+Tab must cycle only within the open modal.

**3.3.8 — Accessible Authentication (Minimum)** · role: ddev · triggers: overlay · isNew: true
- rule: Login flows must not rely solely on cognitive tests. Design must include at least one path that doesn't require memorisation or transcription.
- do: Design authentication flows that allow copy-paste, biometrics, magic links, or password managers
- dont: Design a login that forces users to memorise and retype an OTP without allowing paste
- doc: Full authentication flow with every available path; confirm at least one non-cognitive alternative and that copy-paste is not blocked

**2.2.1 — Timing Adjustable** · role: ddev · triggers: timeout
- rule: If any screen has a timeout or session expiry, a visible UI to extend or dismiss it must be designed.
- do: Design a timeout warning dialog with clear options to extend, save, or dismiss — before the session expires
- dont: Rely on dev to design the timeout warning — if it's not in your frames, it probably won't exist
- doc: Timeout warning UI design, placement, trigger timing, and the available actions (extend/save/dismiss)

**1.3.4 — Orientation** · role: dev · triggers: orientation
- rule: Content must not be restricted to a single display orientation unless that restriction is essential to the function.
- do: Support both portrait and landscape orientations; if locking is unavoidable, display a clear explanation to the user
- dont: Lock orientation as a convenience or stylistic choice — only lock when content genuinely cannot work otherwise
- impl: Never use screen.orientation.lock() to restrict orientation. Test layouts in both orientations at common breakpoints.

---

### Layer: element

**1.4.3 — Contrast (Minimum)** · role: design · triggers: text, button, link, formfield, selection, notification, datadisplay
- rule: Text must have a contrast ratio of at least 4.5:1 against its background. Small text that's hard to read isn't just a style problem — it's a barrier.
- do: Check every text color + background combination with a contrast checker before finalising
- dont: Use light grey body copy on a white background because it "looks clean"
- note: Large text (18pt regular or 14pt bold and above) only needs a 3:1 ratio.

**1.4.1 — Use of Color** · role: design · triggers: text, button, link, formfield, selection, notification
- rule: Never use color as the only way to communicate something. Strip all color — every piece of information should still be readable.
- do: Pair color with an icon, label, pattern, or underline to convey meaning
- dont: Use a red border alone to indicate an error field — without a label or icon

**1.4.11 — Non-text Contrast** · role: design · triggers: button, formfield, selection, icon, navigation, datadisplay
- rule: UI components and icons must have a contrast ratio of at least 3:1 against their background — buttons, inputs, checkboxes, icons, dividers.
- do: Check border of input fields, stroke of icons, and outline of buttons against their background
- dont: Use a light grey button border on white just because the fill colour looks fine

**2.4.7 — Focus Visible** · role: design · triggers: button, link, formfield, selection, navigation, overlay, gesture
- rule: Every interactive element must show a visible focus indicator when selected via keyboard. If a keyboard user can't see where they are, they're effectively lost.
- do: Design a distinct focus style for every interactive component — buttons, links, inputs, tabs
- dont: Remove the default browser focus ring without replacing it with something equally visible

**2.5.8 — Target Size (Minimum)** · role: design · triggers: button, link, formfield, selection, icon, navigation, overlay, gesture · isNew: true
- rule: Interactive targets must be at least 24×24px. Aim for 44×44px for touch interfaces. Small targets are hard to hit for anyone using a finger, stylus, or with limited motor control.
- do: Set minimum tap/click target sizes in your design system; add invisible padding around small icons
- dont: Design icon-only buttons at 16px and assume the visual size is the tap size

**1.3.3 — Sensory Characteristics** · role: design · triggers: text, button, link, formfield, selection, image, icon
- rule: Never design UI that can only be understood by its appearance, position, or sound. "Tap the blue icon" breaks down for users who can't perceive those characteristics.
- do: Design labels and instructions that name the element directly ("tap Save")
- dont: Design UI that requires copy like "see the panel on the left" to be understood

**1.4.5 — Images of Text** · role: design · triggers: image
- rule: Use real text instead of text baked into images. Images of text can't be resized, translated, or read by screen readers.
- do: Set all headlines, labels, and body copy as live text in Figma
- dont: Drop in a PNG of a stylised heading because the font isn't available
- note: Exception: logos and wordmarks where text is part of brand identity.

**2.3.1 — Three Flashes** · role: design · triggers: media, gesture
- rule: Nothing on screen should flash more than 3 times per second. Flashing content can trigger seizures in users with photosensitive epilepsy.
- do: Keep animations smooth and below threshold; test any looping motion before handoff
- dont: Use rapid strobe effects, fast-cycling highlights, or attention-grabbing flashes

**3.3.2 — Labels or Instructions** · role: design · triggers: formfield, selection
- rule: Every form field needs a persistent visible label. Placeholder text alone doesn't count — it disappears when the user starts typing.
- do: Design visible labels above or beside every input; use placeholder text only as supplementary guidance
- dont: Use placeholder text as the only label — users lose context the moment they start typing
- doc: Confirm every input has a persistent visible label; note any fields with additional instructions or format hints

**1.1.1 — Non-text Content** · role: ddev · triggers: image, icon, media, datadisplay
- rule: Every image, icon, and illustration needs declared intent — decorative or meaningful. Dev can't make this call from looking at the design.
- do: Annotate every non-text element as "decorative" or provide suggested alt text
- dont: Leave icons and illustrations unannotated and assume dev will figure out the intent
- doc: Annotation on each image/icon — mark as "decorative" or provide alt text copy

**1.3.1 — Info and Relationships** · role: ddev · triggers: text, formfield, selection, navigation, datadisplay
- rule: The visual hierarchy you create needs to be encoded semantically. Dev can't guess your intended structure from visual styling alone.
- do: Annotate heading levels (H1/H2/H3), table structures, and logical form groupings on every screen
- dont: Assume that because something looks like a heading, dev will automatically code it as one
- doc: Heading level map per screen (H1–H4); table structure; form field groupings with labels

**1.3.5 — Identify Input Purpose** · role: ddev · triggers: formfield
- rule: Every input field needs its purpose declared so browsers can offer autofill correctly. You define what the field is for; dev implements the correct autocomplete attribute.
- do: Label the purpose of every form field explicitly — name, email, phone, street address, city, etc.
- dont: Use vague labels like "Field 1" or "Enter details"
- doc: Input purpose annotation for every form field using standard names: name, email, tel, street-address, postal-code, etc.

**1.4.2 — Audio Control** · role: ddev · triggers: media
- rule: If any screen has auto-playing audio, a visible pause/stop control must be designed. Dev implements the behavior, but the control must exist in your design.
- do: Design a visible, clearly labelled pause/stop control for any screen with auto-playing audio
- dont: Hand off screens with auto-playing media and leave the control placement undefined
- doc: Placement, size, and label of pause/stop controls for every screen containing auto-playing media

**2.1.1 — Keyboard** · role: ddev · triggers: button, link, formfield, selection, navigation, overlay, gesture
- rule: Every interactive element must be reachable and operable via keyboard alone. Confirm all interactions are covered in your design — dev implements the keyboard handlers.
- do: Create an inventory of every interactive element per screen; flag any custom interactions
- dont: Design interactions that only work with a mouse without specifying a keyboard alternative
- doc: Full inventory of interactive elements per screen; keyboard alternative for any custom or gesture-based interaction

**2.2.2 — Pause, Stop, Hide** · role: ddev · triggers: media, gesture
- rule: Carousels, animations, auto-scrolling content need visible controls. You define where and how; dev implements the behavior.
- do: Include pause, stop, or hide controls for every piece of moving or auto-updating content
- dont: Design auto-playing carousels or tickers without visible controls in the frame
- doc: Placement and visual design of pause/stop/hide controls for all moving content

**2.4.6 — Headings and Labels** · role: ddev · triggers: text, formfield, datadisplay
- rule: The visual heading hierarchy needs to be explicitly mapped to heading levels. "Big text" and "H1" are not the same thing.
- do: Annotate every heading with its intended level (H1, H2, H3, H4) on each screen
- dont: Use heading size as a purely visual decision
- doc: Heading level annotation (H1–H4) for every screen; label text for all form controls and UI regions

**2.5.1 — Pointer Gestures** · role: ddev · triggers: gesture
- rule: Any multi-touch gesture must also be achievable with a single pointer. You design both paths; dev implements them.
- do: For every multi-touch gesture, design an equivalent single-tap or button-based alternative
- dont: Design interactions that only work with two fingers without a single-pointer alternative
- doc: For every multi-touch gesture: name the gesture and specify its single-pointer alternative

**2.5.3 — Label in Name** · role: ddev · triggers: button, link, formfield, selection, icon, navigation
- rule: The visible text label on a button or link must be contained within its accessible name. If a button says "Save," the screen reader cannot call it "Submit."
- do: Ensure the accessible name includes the exact visible label text
- dont: Use an aria-label that replaces or contradicts the visible label
- doc: Accessible name for every interactive component — confirm it contains the visible label verbatim

**2.5.4 — Motion Actuation** · role: ddev · triggers: gesture
- rule: If any feature is triggered by device motion, an equivalent on-screen UI control must be designed.
- do: Design an on-screen button or control equivalent for every motion-triggered feature
- dont: Ship motion-triggered interactions without an on-screen alternative
- doc: On-screen UI alternative for every motion-triggered interaction; note which motion it replaces

**2.5.7 — Dragging Movements** · role: ddev · triggers: gesture · isNew: true
- rule: Any drag interaction needs a single-pointer alternative that doesn't require dragging. You design both; dev implements both.
- do: For every drag-based UI element, design an equivalent interaction using taps, clicks, or keyboard controls
- dont: Design sortable lists or drag-to-resize panels without a non-drag alternative
- doc: For every drag interaction: name the element, describe the drag, and specify the single-pointer alternative

**3.3.1 — Error Identification** · role: ddev · triggers: formfield, selection, notification
- rule: Error states must be designed for every form field that can fail validation. You design the error state; dev implements detection and triggers the right state.
- do: Design an explicit error state for every input — with color, icon, and a descriptive error message
- dont: Hand off forms without designed error states
- doc: Error state design for every input; error message copy for each failure scenario (empty, wrong format, too long, etc.)

**3.3.3 — Error Suggestion** · role: ddev · triggers: formfield, selection, notification
- rule: "Invalid date" tells a user what's wrong. "Enter a date in DD/MM/YYYY format" tells them how to fix it. That's the difference this criterion requires.
- do: Write specific, actionable error suggestion copy for every common error scenario
- dont: Use generic messages like "Invalid input"
- doc: Error suggestion copy and placement for each common error scenario per field

**3.3.7 — Redundant Entry** · role: ddev · triggers: formfield, selection · isNew: true
- rule: Don't ask users for information they've already provided in the same session. Multi-step forms and checkout flows are common places this breaks down.
- do: Audit every multi-step flow — pre-fill previously entered information rather than asking again
- dont: Ask for name, email, or address a second time in the same session
- doc: Flag every field in a multi-step flow that duplicates previously collected data; specify which earlier field it maps to

**4.1.2 — Name, Role, Value** · role: ddev · triggers: button, link, formfield, selection, icon, navigation, overlay, gesture
- rule: Every interactive component needs its name, role, and all possible states documented so dev can implement correct ARIA markup.
- do: Annotate every component with its name, role, and all interactive states
- dont: Hand off custom components without naming their role and states
- doc: For every interactive component: accessible name, ARIA role, and all states — default, hover, focus, active, disabled, selected, expanded, error

**4.1.3 — Status Messages** · role: ddev · triggers: notification
- rule: Success banners, loading states, error alerts need to be announced to screen readers without moving keyboard focus. You define where and when; dev implements the live region.
- do: Design every status message as a distinct component with a defined trigger — mark that focus must not shift to it
- dont: Design toast notifications without specifying they should not steal keyboard focus
- doc: For every status message: trigger condition, message copy, placement, duration — note focus must not shift

**1.4.4 — Resize Text** · role: dev · triggers: text, button, link, formfield, notification
- rule: Text must be resizable up to 200% without losing content or functionality. This is purely a CSS implementation concern — use relative units, not fixed pixels.
- do: Use rem or em for all font sizes; use min-height on text containers
- dont: Set font sizes in px — pixel values ignore the user's browser font size preference
- impl: Set all font sizes in rem. Test by setting browser default font size to 32px and verify no content is clipped.

**1.4.12 — Text Spacing** · role: dev · triggers: text, button, link, formfield, notification, datadisplay
- rule: When users override text spacing, no content or functionality should break. Write CSS that doesn't snap under spacing pressure.
- do: Use min-height instead of height on text containers; allow containers to grow naturally
- dont: Set fixed heights on text containers or use overflow: hidden on elements with text
- impl: Test by injecting spacing overrides: line-height: 1.5; letter-spacing: 0.12em; word-spacing: 0.16em — verify no text is clipped.

**2.1.4 — Character Key Shortcuts** · role: dev · triggers: gesture
- rule: If single-character keyboard shortcuts are implemented, users must be able to turn them off or remap them. They conflict with screen reader and speech input commands.
- do: Require at least one modifier key for all keyboard shortcuts; or provide a settings toggle to disable single-key shortcuts
- dont: Implement shortcuts that fire on a single letter key — they clash with screen reader navigation commands
- impl: Audit all keydown/keyup listeners. Any responding to a single character key without a modifier needs either a modifier added or a toggle to disable.

**2.5.2 — Pointer Cancellation** · role: dev · triggers: button, link, selection, overlay, gesture
- rule: Actions should fire on the up-event, not the down-event. This gives users a chance to cancel by moving the pointer away before releasing.
- do: Use click events — they fire on pointer up and include built-in cancellation behaviour
- dont: Use mousedown or pointerdown to trigger irreversible or destructive actions
- impl: Audit all mousedown/pointerdown/touchstart listeners that trigger actions. Replace with click events.

**3.1.2 — Language of Parts** · role: content · triggers: text, link
- rule: When content switches language mid-page, the change must be flagged so dev can mark it up correctly with a lang attribute.
- do: Clearly mark any non-primary-language content in your copy deck so dev knows to apply the correct lang attribute
- dont: Silently include foreign language phrases without flagging them
- owns: Flag all language switches in your copy with a note to dev. Doesn't apply to proper nouns or brand names.

**1.2.1 — Audio-only / Video-only (Prerecorded)** · role: content · triggers: media
- rule: Prerecorded audio-only content needs a text transcript. Silent video needs a transcript or audio description track.
- do: Write accurate transcripts for all audio-only content; write descriptive text alternatives for silent video
- dont: Publish audio or video content without a text alternative

**1.2.2 — Captions (Prerecorded)** · role: content · triggers: media
- rule: Every prerecorded video with audio must have synchronised, reviewed captions. Auto-generated captions alone are not sufficient.
- do: Review and correct auto-captions; include speaker names and meaningful non-speech sounds
- dont: Publish auto-generated captions without review

**1.2.3 — Audio Description or Media Alternative** · role: content · triggers: media
- rule: Prerecorded video must have an audio description track or full text alternative describing both audio and visual content.
- do: Write audio description scripts narrating visual information not conveyed in the dialogue
- dont: Assume the dialogue alone conveys everything — key visual information is invisible to some users

**1.2.4 — Captions (Live)** · role: content · triggers: media
- rule: Live audio-video content must have real-time captions. Pre-prepared transcripts are not a substitute.
- do: Arrange CART or equivalent live captioning for any streamed or broadcast event
- dont: Rely on auto-captions for live events or assume a post-event transcript satisfies this

**1.2.5 — Audio Description (Prerecorded)** · role: content · triggers: media
- rule: All prerecorded video must have an audio description track narrating meaningful visual information not conveyed in the main audio.
- do: Script descriptions that fit natural pauses; describe actions, scene changes, on-screen text
- dont: Describe what's already said in the dialogue — audio descriptions fill the visual gaps, not repeat the speech

---

## Filter logic

```js
function getApplicableCriteria(selectedStructure, selectedElements) {
  const seen = new Set();
  const result = [];
  CRITERIA.forEach(c => {
    if (seen.has(c.id)) return;
    let include = false;
    if (c.layer === 'always') include = true;
    if (c.layer === 'structure' && c.triggers.some(t => selectedStructure.has(t))) include = true;
    if (c.layer === 'element'   && c.triggers.some(t => selectedElements.has(t)))  include = true;
    if (include) { seen.add(c.id); result.push(c); }
  });
  return result;
}
```

Key rule: **deduplicate by criterion ID**. A criterion that appears in multiple trigger lists (e.g. 1.3.2 triggered by both `sticky` and `nonlinear`) should appear only once in the checklist.

---

## Checklist UI behaviour

### Item states
Each criterion in the checklist has one of three states:
- `open` — default, unchecked
- `checked` — done, shown with green tint + checkmark active
- `na` — not applicable, shown at reduced opacity + dash active

Clicking a state button that's already active returns the item to `open` (toggle behaviour).

### Item card structure
```
[✓] [–]   1.4.3  Contrast (Minimum)          [Element tag]
          Rule text (one paragraph)
          [Show guidance ▾]
            ┌─────────────────┬─────────────────┐
            │ ✓ Do            │ ✗ Don't         │
            └─────────────────┴─────────────────┘
            [Document / Implement / Your part block if applicable]
            [Note if applicable]
```

### Expand/collapse
"Show guidance" toggles the Do/Don't + extra block open/closed. Default: collapsed. This keeps the checklist scannable.

### Progress
- Progress bar at top of checklist — fills based on checked / total ratio
- Stats row: X done · X N/A · X open
- Re-renders on every state change

### Grouping
Items are grouped by role within the checklist:
1. Design
2. Design → Dev
3. Dev Only
4. Content

Always-on criteria distribute into their role group — they don't get a separate group.

---

## Reference tabs

The Full Reference mode has four tabs — Design, Design→Dev, Dev Only, Content — with all criteria shown in full card format including Do/Don't, Document/Implement/Your part blocks, and notes. POUR section headers within each tab. This content already exists in the current `wcag-guidelines.html` file and should be preserved as-is.

---

## What to build

Start from the existing `wcag-guidelines.html` which has the full reference tab content already. The work is:

1. **Add the mode switcher** — [Screen Checklist] [Full Reference] buttons at the top
2. **Add the filter panel** — screen name input, layer callout, structure chips, element chips, generate + clear buttons
3. **Build the CRITERIA data array** — all 56 criteria as JS objects per the spec above
4. **Implement the filter logic** — union of triggers, deduplication by ID
5. **Implement the checklist renderer** — grouped by role, with state management, progress bar, expand/collapse
6. **Wire up state management** — checked/na/open toggle, re-render on change, state preserved in memory during session
7. **Wrap the existing reference content** in the Full Reference panel

The existing CSS tokens, card styles, tab styles, and reference card content should be preserved and reused.

---

## Out of scope for this build

- Persistence across sessions (localStorage) — in-memory only for now
- Multiple saved screens / screen history
- Export or print view
- Figma plugin integration

These are next-phase features.

---

## File output

Single file: `a11y-sidekick.html`
No external dependencies except Google Fonts (already in the existing file).
No build step.
