---
name: audit-ui
description: Use when you need to audit a frontend codebase to find unexposed routes, orphan components, unused assets, or general frontend hygiene issues.
---

# Frontend Auditing & Code Health

A framework-agnostic checklist to audit and inventory a frontend codebase (React, Vue, Svelte, Angular). Use this to identify dead code, unused assets, and routing issues before beginning a refactor.

All paths in this skill are relative to the **frontend repo root**.

## 0. Detect Project Stack First

Before auditing, inspect the repository to identify the current stack. Do NOT suggest framework-specific fixes until you confirm the stack.

1. **Framework & Package Manager:** Check `package.json` for React/Next.js/Vite, Vue/Nuxt, SvelteKit, etc.
2. **Routing System:** Is routing file-based (`app/`, `pages/`) or configuration-based (`react-router`, `vue-router`)?
3. **Styling Approach:** Check for Tailwind, CSS Modules, Styled Components, Emotion, or plain Sass/CSS.
4. **Component Structure:** Does the project use `components/`, `features/`, `views/`, or atomic design?

---

## 1. Routing & Component Audit

### 1.1 Route Inventory & Unexposed Routes
- **Inventory:** List all declared routes by inspecting the router config or the file-system router directory.
- **Unexposed Routes:** Check the main navigation, sidebars, and `<Link>`/`<a>` tags. Identify pages that exist in the codebase but are never linked to (unreachable by users).

### 1.2 Orphan Component Detection
Find components that are defined and exported but never actually imported or used by any other file.

```bash
# Example logic (adjust based on tool available):
# Search for the component name export, then grep the codebase for imports of that name.
```

### 1.3 Asset Audit
Identify images, SVGs, and fonts in the `public/` or `src/assets/` directories that are never referenced in the source code.

---

## 2. Frontend Housekeeping Checklist

> This checklist is about **detecting** issues, not fixing them. Escalate identified issues to the `refactor-ui` skill for resolution.

#### Cleanup
- [ ] No unexposed routes detected
- [ ] No orphan components detected
- [ ] No unused assets (images, fonts, SVGs) detected
- [ ] No unused imports or variables detected
- [ ] No commented-out JSX/HTML detected
- [ ] No debug `console.log` noise detected

#### Formatting & Linting
- [ ] Linter passes cleanly
- [ ] Formatter (e.g. Prettier) has been applied
- [ ] Type check passes (if TypeScript is used)
- [ ] CSS class naming is consistent

#### Dependencies
- [ ] No unused npm packages found in `package.json`
- [ ] No double-installed libraries (e.g. `lodash` vs `lodash-es`)
- [ ] Bundle size check: Identify unusually heavy dependencies

#### Accessibility (A11y)
- [ ] Basic scan complete: `alt` tags present on images
- [ ] Form inputs have associated labels
- [ ] Interactive elements have appropriate ARIA roles or semantic HTML tags

---

## 3. Track Progress

Create `ui_review.md` in the repo root to track component/page status:

```markdown
# UI Audit Review
**Branch:** `current-branch`
**Date:** YYYY-MM-DD

## Audit Findings
- ❌ Unexposed Route: `/src/pages/old-dashboard.tsx`
- ❌ Orphan Component: `<LegacyButton />`
- ❌ Unused Asset: `hero-bg-v1.png`
```