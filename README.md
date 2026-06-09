# Tidy Code

A reusable, language-agnostic skillset for Claude to audit, clean, and refactor a microservice or backend API. Derived from a battle-tested housekeeping checklist.

Extensible by design — the same patterns can be applied to frontend codebases as well.

## Contents

* `skills/audit-api/SKILL.md` - Skill for auditing backend APIs, finding dead code, and analyzing endpoint usage.
* `skills/refactor-api/SKILL.md` - Skill for cleaning backend code, standardizing error handling, and eliminating duplication.
* `skills/testing-api/SKILL.md` - Skill to create, complete, and execute backend tests following TDD.
* `skills/audit-ui/SKILL.md` - Skill for auditing frontend codebases, finding unexposed routes, orphan components, and unused assets.
* `skills/refactor-ui/SKILL.md` - Skill for cleaning frontend code, decoupling presentation from logic, and standardizing styles.
* `skills/testing-ui/SKILL.md` - Skill to create, complete, and execute frontend unit/component tests following TDD.
* `bin/driver.sh` - An executable CLI tool to automatically inventory endpoints, check usages against frontend code, identify leaking HTTP concerns in your model layer, and more (supports JS/TS, Python, Java, Go, Ruby).

## Installation

You can install this repository directly as a Claude Code plugin:

```shell
/plugin install tidy-code@https://github.com/ekacahya21/Tidy-Code.git
```

Alternatively, you can register it as a marketplace:

```shell
/plugin marketplace add https://github.com/ekacahya21/Tidy-Code.git
/plugin install tidy-code@tidy-code-marketplace
```

## Usage for Claude users

Once installed, the skills become available. You can run them inside any repository:

```shell
/tidy-code:audit-api
/tidy-code:refactor-api
/tidy-code:testing-api
/tidy-code:audit-ui
/tidy-code:refactor-ui
/tidy-code:testing-ui
```

Or simply ask Claude Code: 
- "Please audit this codebase using the audit-api skill".
- "Review this PR using the refactor-api skill to ensure it follows our clean code standards."
- "Create unit tests for this controller using the testing-api skill."

## Features

- **Endpoint Audit** (`audit-api`): Inventory routes across all files and cross-reference them with frontend/clients to verify actual usage. YAGNI-driven approach.
- **Error Standardization** (`refactor-api`): Patterns for creating controller wrappers to enforce consistent try/catch patterns and standard `sendSuccess`/`sendError` formats.
- **De-duplication** (`refactor-api`): Strategy for consolidating combinatorial conditional queries.
- **Unit Testing** (`testing-api`): Create, complete, and execute unit tests following TDD principles. Includes mocking and coverage compliance.
- **Utility Extraction** (`refactor-api`): Identifies and extracts repeated outputs (e.g. Excel export) to shared libraries.
- **Frontend Audit** (`audit-ui`): Find unexposed routes, orphan components, and unused assets in frontend codebases.
- **Frontend Refactor** (`refactor-ui`): Decouple logic from components, extract reusable UI components, and standardize styling.
- **Frontend Testing** (`testing-ui`): Create, complete, and execute unit, component, and E2E tests for frontend codebases.
