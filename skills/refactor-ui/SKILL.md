---
name: refactor-ui
description: Use when you need to improve code quality in a frontend codebase — decoupling presentation from logic, standardizing state/styling, or extracting reusable components.
---

# Frontend Refactoring & Code Quality

A framework-agnostic checklist to clean, refactor, and decouple frontend components. Use this to ensure your UI code follows clean architecture patterns (separation of concerns, DRY UI components).

All paths in this skill are relative to the **frontend repo root**.

## 0. Detect Project Stack First

Identify the project styling, state management, and framework conventions first. Never write refactored code that breaks existing project idioms.

---

### 1. Presentation vs. Logic Decoupling

Move data fetching, side effects, and state manipulation out of UI components and into custom hooks, context providers, or service classes. The component should primarily handle rendering.

**Before (Mixed logic & UI):**
```jsx
// React Example
function UserProfile() {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch(`/api/user`).then(res => res.json()).then(setUser);
  }, []);
  return <div>{user ? user.name : 'Loading...'}</div>;
}
```

**After (Decoupled):**
```jsx
// hooks/useUser.js
function useUser() {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch(`/api/user`).then(res => res.json()).then(setUser);
  }, []);
  return { user, loading: !user };
}

// UserProfile.js
function UserProfile() {
  const { user, loading } = useUser();
  if (loading) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}
```

---

### 2. Component Extraction (DRY UI)

- **Reusable UI Elements:** Extract repeated JSX layouts (e.g. repeated Tailwind button styles, custom input fields) into generic UI presentational components.
- **God Components:** Break down massive components (e.g., a dashboard page component) into small, focused sub-components.

---

### 3. Styling Standardization

Consolidate inline styles, duplicate classes, and inconsistent theme declarations:
- Replace inline style props with classes.
- Extract repeated layout classes into shared helper classes or design token variables.
- Ensure all margins, paddings, and colors align with the existing project theme.

---

### 4. Refactoring Checklist

Use this checklist to ensure refactoring does not break visual or interactive logic.

#### Scope
- [ ] Refactor scope is clearly defined
- [ ] Affected pages/components are listed
- [ ] No unrelated feature changes included

#### Visual Safety
- [ ] Layout matches the original exactly (no shifting, margins, or padding breaks)
- [ ] Responsive design still functions correctly on mobile/desktop
- [ ] Styling tokens or variables align with project conventions

#### Interaction Safety
- [ ] Click, hover, and focus states behave identically
- [ ] Forms, validation, and submissions function correctly
- [ ] Navigation and route transitions are preserved

#### State Safety
- [ ] Loading states still trigger correctly
- [ ] Error boundary behavior and empty states are preserved
- [ ] State changes do not cause infinite re-renders

#### Testing
- [ ] Component unit tests pass
- [ ] Visual or integration regression checks run cleanly
