# Legacy API Housekeeping

A reusable, language-agnostic skillset for Claude to audit, clean, and refactor a legacy microservice or backend API. Derived from a battle-tested housekeeping checklist.

## Contents

* `HOUSEKEEPING_CHECKLIST.md` - The original Node.js specific checklist that started it all.
* `skills/run-legacy-housekeeping/SKILL.md` - The generalized, language/framework-agnostic skill instructing Claude on how to approach refactoring and auditing dead code.
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

Once installed, the `/legacy-housekeeping:run-legacy-housekeeping` skill command becomes available. You can run it inside any repository to trigger the housekeeping flow:

```shell
/legacy-housekeeping:run-legacy-housekeeping
```

Or simply ask Claude Code: "Please audit this codebase using the legacy housekeeping skill".

## Features

- **Endpoint Audit**: Inventory routes across all files and cross-reference them with frontend/clients to verify actual usage. YAGNI-driven approach.
- **Error Standardization**: Patterns for creating controller wrappers to enforce consistent try/catch patterns and standard `sendSuccess`/`sendError` formats.
- **De-duplication**: Strategy for consolidating combinatorial conditional queries.
- **Utility Extraction**: Identifies and extracts repeated outputs (e.g. Excel export) to shared libraries.
- **Model decoupling**: Helps identify models that accept `(req, res)` directly, decoupling them to return data so controllers can own the HTTP response lifecycle.
