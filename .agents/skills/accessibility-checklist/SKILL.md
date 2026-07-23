---
name: accessibility-checklist
description: Canonical WCAG 2.1 AA accessibility checklist shared by the engineer, reviewer, and improver agents. Use when implementing UI/frontend code, when reviewing a diff that touches HTML/JSX/templates/CSS, or when scanning a codebase for accessibility findings. Skip entirely for backend-only changes.
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

## Applying this by role

**Engineer (building):** Build accessible by default — use semantic HTML elements, associate every input with a `<label>`, ensure all interactive elements are keyboard-operable, provide text alternatives for non-text content.

**Reviewer (auditing a diff):** Only apply this section if the diff touches HTML, JSX, templates, or CSS. Any checklist violation that breaks WCAG 2.1 AA is a blocking finding — `[ACCESSIBILITY] file:line — description`. Advisory improvements beyond the AA minimum are non-blocking observations.

**Improver (scanning a codebase):** This is Tier 6 of the scan — report "Skipped — no UI code in scope" if the scan scope is backend-only. Use the standard finding format (Title / Risk / Suggestion / Effort).
