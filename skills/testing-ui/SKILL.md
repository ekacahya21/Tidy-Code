---
name: testing-ui
description: Use when you need to write, complete, or fix unit, component, and E2E tests for a frontend application to ensure maximum coverage and reliability.
---

# Frontend Testing & TDD

A framework-agnostic checklist to create and complete tests for frontend components and utilities. Follow the TDD (Red-Green-Refactor) pattern and ensure proper isolation.

All paths in this skill are relative to the **frontend repo root**.

## 0. Detect Project Stack First

Check `package.json` to identify:
- **Test Runner:** Jest, Vitest
- **Component Testing:** React Testing Library (RTL), Vue Test Utils
- **E2E Testing:** Cypress, Playwright
- **Mocking:** MSW (Mock Service Worker), fetch-mock

---

### 1. Test-Driven Development (TDD) Loop

**NO component logic or markup changes without a test verifying the change first.**

1. **RED:** Write a test that mounts the component and queries for expected elements or behaviors. Run and watch it fail.
2. **GREEN:** Write the minimal implementation markup/logic to pass the test.
3. **REFACTOR:** Clean up the component and the test code.

---

### 2. Component Testing Best Practices

#### 2.1 Mock External Data & APIs
Never make real HTTP requests in component tests.
- Use **MSW** to intercept network calls.
- Mock local service functions or API client wrappers if MSW is not configured.

#### 2.2 Wrapper Providers
Frontend components often rely on global context (Theme, Auth, Redux, Router). Ensure your test setup renders the component inside the necessary wrapper providers.

```javascript
// Example RTL custom render pattern
import { render } from '@testing-library/react';
import { ThemeProvider } from './theme';
import { MemoryRouter } from 'react-router-dom';

const renderWithProviders = (ui, options) =>
  render(ui, { wrapper: ({ children }) => (
    <ThemeProvider>
      <MemoryRouter>{children}</MemoryRouter>
    </ThemeProvider>
  ), ...options });
```

#### 2.3 Query by Accessibility
Prefer testing library queries that mimic user interaction and accessibility:
- Prefer `getByRole`, `getByLabelText`, `getByText` over `getByTestId` or `querySelector`.

---

### 3. Component Testing Checklist

#### TDD Compliance
- [ ] Test written *before* implementation
- [ ] Test verified failing (RED)
- [ ] Minimal code added to pass test (GREEN)

#### State & Interaction Coverage
- [ ] Happy path (renders with correct default props/data)
- [ ] Conditional UI (Loading spinner, Empty state, Error message)
- [ ] User events (Clicks, form inputs) trigger correct state changes or callbacks

#### Environment Isolation
- [ ] Component is wrapped in required providers (Router, Context, Store)
- [ ] API calls are mocked (MSW or fetch mocks)
- [ ] Timers (`setTimeout`, `setInterval`) are mocked/faked if applicable

#### Quality & Cleanliness
- [ ] Elements queried using accessible roles (e.g., `getByRole('button')`)
- [ ] No `console.error` or `act()` warnings in the test runner output
- [ ] Test coverage meets project target and no regressions introduced

---

## Red Flags - STOP and Start Over

- Component markup written before the test
- "I'll just test the hook and skip the UI render test"
- Relying on real API network calls inside tests
- Using brittle CSS selectors (`document.querySelector('.btn-blue')`) instead of role queries

**All of these mean: Stop. Revert. Start over with TDD and proper testing library principles.**