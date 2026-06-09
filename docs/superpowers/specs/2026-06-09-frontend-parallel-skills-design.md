# Frontend Parallel Skills Design Spec

**Date:** 2026-06-09
**Author:** Claude Code

## 1. Overview
The `Tidy-Code` toolkit currently provides backend/API auditing, refactoring, and testing skills. To make this plugin compatible with frontend development workflows, we are adding three parallel frontend skills: `audit-ui`, `refactor-ui`, and `testing-ui`.

These skills will be framework-agnostic ("repo-adaptive"), inspecting the repository first to determine the styling system, router, testing library, and package manager before providing advice.

---

## 2. Architecture & Directory Structure
The frontend skills will reside in the `skills/` directory of the `Tidy-Code` repository, parallel to the existing API skills:

```
skills/
├── audit-api/
│   └── SKILL.md
├── refactor-api/
│   └── SKILL.md
├── testing-api/
│   └── SKILL.md
├── audit-ui/            # NEW
│   └── SKILL.md
├── refactor-ui/          # NEW
│   └── SKILL.md
└── testing-ui/          # NEW
    └── SKILL.md
```

---

## 3. Detailed Design

### 3.1 `audit-ui`
A guide to inspect a frontend codebase, detect the tech stack, inventory routes/components, check for unused files or assets, and detect hygiene issues.

#### Stack Detection Flow
1. Check `package.json` for frameworks: React (Next.js, Vite/CRA), Vue (Nuxt), Angular, Svelte (SvelteKit), Solid.
2. Check `package.json` for routing: `react-router`, `vue-router`, file-based router.
3. Check `package.json` for styling: `tailwindcss`, `styled-components`, `sass`, `css-modules`.

#### Routing & Component Audit Flow
- Pages: Check the route tree or pages folder. Find components in page directories that are not defined in route paths or navigable by links (unexposed routes).
- Components: Search exports across `components/` to find orphans (never imported).
- Assets: Cross-reference image files, SVGs, and fonts in `public/` and `assets/` with source files.

---

### 3.2 `refactor-ui`
A guide to clean, reorganize, and improve frontend code without altering user-visible behavior. Focuses on separation of concerns (decoupling business logic from presentational logic) and UI deduplication.

#### Reusable Code Quality Patterns
- **Presentation vs. Logic Decoupling:** Extract raw HTTP calls, stateful calculations, and side-effects from JSX files into custom hooks, context providers, or helper services.
- **UI Deduplication:** Identify repeated DOM layouts (e.g. repeated CSS grids or forms) and extract them into clean, presentational reusable UI components.
- **Styling Standardization:** Consolidate duplicate style classes (Tailwind or CSS classes) into config files, variables, or custom component wrappers.

---

### 3.3 `testing-ui`
A guide to write and complete unit, integration, and component tests on the frontend. Promotes TDD loop (Red-Green-Refactor) and proper mocking of global state, routers, and API networks.

#### Key Testing Guidelines
- **TDD loop:** Ensure tests are written and run in a failing state (RED) before changing components.
- **Mocking:** Setup mock networks using tools like MSW (Mock Service Worker) to mock raw fetch/axios calls rather than calling live endpoints during unit tests.
- **Mocking Providers:** Wrap components under test with required providers (e.g. Redux Store, AuthProvider, ThemeProvider, RouterProvider) to isolate component tests.

---

## 4. Checklists

### 4.1 `audit-ui` Checklist
- [ ] **Routing Audit:** Route tree mapped, unexposed route paths identified.
- [ ] **Orphan Components:** Components defined but never imported found.
- [ ] **Unused Assets:** Unused images, SVGs, and fonts listed.
- [ ] **Formatting & Lint:** Linter clean, formatter applied.
- [ ] **Accessibility:** Alt tags present, form labels mapped, ARIA roles used correctly.

### 4.2 `refactor-ui` Checklist
- [ ] **Logic Decoupled:** Business logic/fetching moved to custom hooks or services.
- [ ] **UI Deduplicated:** Repeated JSX structures extracted into reusable UI elements.
- [ ] **Styling Standardized:** Layout tokens, Tailwind config, or CSS modules aligned.
- [ ] **Visual Safety:** Visual design looks identical before and after.
- [ ] **Interaction Safety:** Navigation, buttons, forms behave identically.

### 4.3 `testing-ui` Checklist
- [ ] **TDD Loop:** Red-Green-Refactor cycle followed.
- [ ] **Happy Path:** Correct rendering under typical inputs.
- [ ] **Interaction:** Clicking, typing, submitting triggers expected events.
- [ ] **Error/Empty States:** Loading flags and error boundaries verified.
- [ ] **Isolation:** API calls mocked cleanly (no real server hits).
- [ ] **Accessibility:** Elements queried by accessible role.

---

## 5. Next Steps
Once this design is approved by the user, we will transition to writing the individual subskills in their respective directories.
