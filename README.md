# Backend API Housekeeping

A reusable, language-agnostic skillset for Claude to audit, clean, and refactor a microservice or backend API. Derived from a battle-tested housekeeping checklist.

## Contents

* `HOUSEKEEPING_CHECKLIST.md` - The original Node.js specific checklist that started it all.
* `skills/audit-api/SKILL.md` - Skill for auditing codebases, finding dead code, and analyzing endpoint usage.
* `skills/refactor-api/SKILL.md` - Skill for cleaning code, standardizing error handling, and eliminating duplication.
* `bin/driver.sh` - An executable CLI tool to automatically inventory endpoints, check usages against frontend code, identify leaking HTTP concerns in your model layer, and more (supports JS/TS, Python, Java, Go, Ruby).

## Installation

You can install this repository directly as a Claude Code plugin:

```shell
/plugin install legacy-housekeeping@https://github.com/ekacahya21/Legacy-Housekeeping.git
```

Alternatively, you can register it as a marketplace:

```shell
/plugin marketplace add https://github.com/ekacahya21/Legacy-Housekeeping.git
/plugin install legacy-housekeeping@legacy-housekeeping-marketplace
```

## Usage for Claude users

Once installed, the skills become available. You can run them inside any repository to trigger the housekeeping flow:

```shell
/legacy-housekeeping:audit-api
/legacy-housekeeping:refactor-api
```

Or simply ask Claude Code: 
- "Please audit this codebase using the audit-api skill".
- "Review this PR using the refactor-api skill to ensure it follows our clean code standards."

## Features

- **Endpoint Audit** (`audit-api`): Inventory routes across all files and cross-reference them with frontend/clients to verify actual usage. YAGNI-driven approach.
- **Error Standardization** (`refactor-api`): Patterns for creating controller wrappers to enforce consistent try/catch patterns and standard `sendSuccess`/`sendError` formats.
- **De-duplication** (`refactor-api`): Strategy for consolidating combinatorial conditional queries.
- **Utility Extraction** (`refactor-api`): Identifies and extracts repeated outputs (e.g. Excel export) to shared libraries.
- **Model decoupling** (`refactor-api`): Helps identify models that accept `(req, res)` directly, decoupling them to return data so controllers can own the HTTP response lifecycle.
