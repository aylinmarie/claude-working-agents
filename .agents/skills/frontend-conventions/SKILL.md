---
name: frontend-conventions
description: Canonical frontend stack, styling, and design-token conventions shared by the engineer, reviewer, and improver agents. Use when implementing, reviewing, or scanning any frontend/UI code — choosing a stack for a new frontend project, writing component styles, or setting up theming.
---

This is the single source of truth for frontend build conventions across the engineer → tester → reviewer pipeline and the standalone improver scan. All three agents apply the rules below — only the framing of what to do with a violation differs by role (see "Applying this by role").

## 1. Stack selection

Default to **Next.js (App Router) + React + TypeScript** for any frontend project.

The only exception is a genuinely static, single-purpose site: a marketing/brochure page with no routing beyond anchors, no client-side state, no auth, no data fetching, and no forms with validation or multi-step state. Those may stay plain HTML/CSS (+ minimal vanilla JS if needed).

The moment a project needs any of the following, it's Next.js/React/TS, not plain HTML — even if it currently "looks simple":
- More than one route/page
- Client-side interactivity or state
- Data fetching (client or server)
- Auth or user sessions
- Forms with validation or multi-field state

If the user explicitly names a different stack, follow that instead — this default only applies when the stack is unspecified.

### Tooling defaults (Next.js projects)
- **Router:** App Router (`app/`), not Pages Router, unless the project already uses Pages Router.
- **Language:** TypeScript, strict mode on.
- **Testing:** Vitest + React Testing Library for unit/component tests. Add Playwright only for e2e, and only when explicitly requested or already present in the repo.
- **Linting/formatting:** ESLint (`next/core-web-vitals`) + Prettier. Add Stylelint for CSS Modules files.
- **State management:** Local component state and React Context first — do not add a state management library by default. If a project genuinely needs shared client state across a wide component tree (state Context/prop-drilling can't handle cleanly), reach for **Zustand** first — it's the lightest option (~1KB, no boilerplate, no provider wrapping) and covers the vast majority of cases. Only consider something heavier (Redux Toolkit, Jotai, etc.) if the project's requirements specifically call for it (e.g., atomic state, time-travel debugging, an existing team convention).
- **Data fetching:** Server Components + `fetch`/Server Actions by default. Reach for a client data library (SWR/TanStack Query) only when client-side caching, polling, or optimistic-mutation UX genuinely requires it.

## 2. Styling: CSS Modules + CSS custom-property tokens

- One `.module.css` file per component, co-located with the component (`Button.tsx` + `Button.module.css` in the same folder). No CSS-in-JS, no Tailwind, no inline `style` props, no global stylesheets beyond resets and token definitions — unless the user explicitly asks for a different approach.
- Theming runs entirely on CSS custom properties (variables), in **two tiers**: primitive and semantic. No third component-level tier — component styles reference semantic tokens directly.

### Primitive tokens (raw values)
Undecorated design values with no meaning attached — just `base` + `modifier` (scale/index). Defined once, global, in `tokens/primitives.css`:

```css
:root {
  --color-blue-500: #2563eb;
  --color-gray-50: #f9fafb;
  --color-gray-900: #111827;
  --space-1: 4px;
  --space-4: 16px;
  --font-size-100: 1rem;
  --radius-md: 8px;
}
```

### Semantic tokens (purpose-based, theme-aware)
Map a primitive to *what it's for*, not what it looks like. Defined in `tokens/semantic.css`, referencing primitives via `var()`:

```css
:root {
  --color-bg-canvas: var(--color-gray-50);
  --color-bg-surface: var(--color-white);
  --color-text-primary: var(--color-gray-900);
  --color-text-muted: var(--color-gray-600);
  --color-border-default: var(--color-gray-200);
  --color-border-focus: var(--color-blue-500);
  --color-action-primary-bg: var(--color-blue-600);
  --color-action-primary-bg-hover: var(--color-blue-700);
}

[data-theme='dark'] {
  --color-bg-canvas: var(--color-gray-900);
  --color-bg-surface: var(--color-gray-800);
  --color-text-primary: var(--color-gray-50);
  --color-text-muted: var(--color-gray-400);
  --color-border-default: var(--color-gray-700);
}
```

**Theming rule:** only semantic tokens get redefined per theme (`[data-theme="dark"]`, `prefers-color-scheme`, brand variants, etc.). Primitives are the constant, global palette and are never redefined per theme.

**Component rule:** component CSS Modules reference semantic tokens only — never a primitive directly, and never a hardcoded literal (hex color, raw px value, etc.):

```css
/* Button.module.css */
.primary {
  background: var(--color-action-primary-bg);
  color: var(--color-text-inverse);
  border-radius: var(--radius-md);
  padding: var(--space-2) var(--space-4);
}
```

### Naming taxonomy
Follows the namespace/object/base/modifier structure from Nathan Curtis's ["Naming Tokens in Design Systems"](https://medium.com/eightshapes-llc/naming-tokens-in-design-systems-9e86c7444676) (EightShapes), adapted to two tiers:

- **Primitive names** = `base` + `modifier` only — e.g. `color-blue-500`, `space-4`, `font-size-100`. The name describes the value itself; no purpose is implied.
- **Semantic names** = `object` + `base` + `modifier` — e.g. `color-bg-surface`, `color-text-muted`, `color-action-primary-hover`. The name describes what the token is *for*, never what it looks like (don't name a token `color-text-blue` — name it by role, e.g. `color-text-link`).
- Keep segment order consistent across every token (`object-base-modifier`, always in that order) so names stay predictable and sortable.
- Keep the taxonomy shallow — 2 segments for primitives, 3 (rarely 4) for semantic. If a name needs a 5th segment to stay unambiguous, the taxonomy has grown too deep; split into a new object category instead.
- Prefer relational spacing names over raw scale references where they add clarity (`space-inset-md`, `space-stack-lg`) so a token communicates intent, not just a number.
- Never encode a specific value into a semantic name (no `color-bg-white`) — semantic names must survive a value change (light-to-dark remap, rebrand) without becoming misleading.

## 3. Component conventions
- Co-locate: `ComponentName/ComponentName.tsx`, `ComponentName.module.css`, and `ComponentName.test.tsx` in the same folder.
- Component folders and files: PascalCase. CSS Module class names: camelCase (`.primaryButton`, not `.primary-button`) so they read naturally as `styles.primaryButton` in TSX.
- Build to the `accessibility-checklist` skill for all UI work — this skill covers stack/styling, that one covers WCAG compliance; both apply to every frontend change.

## Applying this by role

**Engineer (building):** Apply the stack-selection rule before scaffolding a new frontend project. Use CSS Modules co-located with components, define new design values as primitives first, then a semantic alias before using them in a component. Never reach into a primitive token or a hardcoded literal from component CSS. Don't add a state management library unless local state/Context is genuinely insufficient — reach for Zustand first when one is warranted.

**Reviewer (auditing a diff):** Flag any component style that references a primitive token directly, or hardcodes a color/spacing/typography literal that should be a token — `[FRONTEND-CONVENTIONS] file:line — description`. Flag a stack choice that ignores the Next.js/React/TS default without justification, or a new state management dependency added where local state/Context/Zustand would suffice. These are non-blocking observations unless they represent a significant, hard-to-undo architectural choice (e.g., introducing Redux for a small app, using CSS-in-JS instead of CSS Modules) — then treat it as blocking.

**Improver (scanning a codebase):** Fold findings into Tier 4 (Maintainability) or Tier 5 (Style & Convention) as appropriate — e.g., hardcoded literals bypassing tokens, primitive tokens used directly in components, inconsistent token naming, unnecessary state-management dependencies. Use the standard finding format (Title / Risk / Suggestion / Effort).
