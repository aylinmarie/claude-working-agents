---
name: accessibility-checklist
description: Canonical WCAG 2.2 AA accessibility checklist shared by the engineer, reviewer, and improver agents. Use when implementing UI/frontend code, when reviewing a diff that touches HTML/JSX/templates/CSS, or when scanning a codebase for accessibility findings. Skip entirely for backend-only changes.
---

This is the single source of truth for accessibility criteria across the engineer → tester → reviewer pipeline and the standalone improver scan. All three agents apply the same list below — only the framing of what to do with a hit differs by role (see "Applying this by role"). Skip this checklist entirely if the code in scope contains no HTML, JSX, templates, or CSS.

## Checklist

- Do all images and icons have meaningful `alt` text (or `alt=""` for decorative ones)?
- Are interactive elements (buttons, links, inputs) reachable and operable by keyboard alone — no click handlers on non-interactive elements (`div`/`span`) without a role and keyboard support?
- Is focus order logical, and does focus never become trapped (except in intentional modals)?
- Are ARIA roles, labels, and `aria-*` attributes used correctly — not redundantly or incorrectly?
- Do form inputs have associated `<label>` elements (via `for`/`id` or `aria-label`)?
- Do color and contrast ratios meet AA minimums (4.5:1 for normal text, 3:1 for large text and UI components)?
- Is information conveyed by color also conveyed by another mechanism (text, pattern, icon)?
- Are dynamic content changes (toasts, modals, errors) announced to screen readers via live regions or focus management?
- Are error messages associated with their fields via `aria-describedby` or equivalent?
- Do interactive components that are not native HTML elements implement the correct ARIA pattern (e.g., listbox, combobox, dialog)?
- **(2.4.11 Focus Not Obscured)** When an element receives keyboard focus, is it left at least partially visible — not fully hidden behind a sticky header, sticky footer, cookie banner, or other author-created overlay content?
- **(2.5.7 Dragging Movements)** For any functionality operated by dragging (reordering, sliders, drag-and-drop), is there a single-pointer alternative that doesn't require a drag gesture (e.g., up/down buttons, tap-to-select-then-tap-to-place), unless dragging is essential to the function?
- **(2.5.8 Target Size Minimum)** Are pointer/touch targets at least 24×24 CSS pixels, or — if smaller — do they have enough spacing from adjacent targets, sit inline in a sentence, have an equivalent same-function control elsewhere on the page that meets the minimum, or is the small size essential to the function?
- **(3.2.6 Consistent Help)** If the same help mechanism (contact link, chat widget, FAQ link, help menu) appears on multiple pages, does it appear in the same relative order/location on each?
- **(3.3.7 Redundant Entry)** In a multi-step process, is information the user already entered auto-populated or made selectable again, rather than requiring them to re-type it — unless re-entry is essential (e.g., re-entering a password for security) or the original information is no longer valid?
- **(3.3.8 Accessible Authentication)** Does any authentication step avoid requiring a cognitive function test (remembering a password, solving a puzzle, transcribing a code) with no alternative — is pasting/password-manager autofill allowed, is there an alternative auth method, or is object/personal-content recognition used instead?

## Applying this by role

**Engineer (building):** Build accessible by default — use semantic HTML elements, associate every input with a `<label>`, ensure all interactive elements are keyboard-operable, provide text alternatives for non-text content. For newer criteria: keep focused elements clear of sticky/overlay content, give drag interactions a non-drag alternative, size tap targets to at least 24×24px, keep help mechanisms in a consistent place, avoid forcing redundant re-entry of previously supplied data, and never gate authentication behind a cognitive test alone.

**Reviewer (auditing a diff):** Only apply this section if the diff touches HTML, JSX, templates, or CSS. Any checklist violation that breaks WCAG 2.2 AA is a blocking finding — `[ACCESSIBILITY] file:line — description`. Advisory improvements beyond the AA minimum are non-blocking observations.

**Improver (scanning a codebase):** This is Tier 6 of the scan — report "Skipped — no UI code in scope" if the scan scope is backend-only. Use the standard finding format (Title / Risk / Suggestion / Effort).
