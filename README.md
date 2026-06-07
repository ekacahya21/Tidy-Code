# Legacy API Housekeeping

A reusable, language-agnostic skillset for Claude to audit, clean, and refactor a legacy microservice or backend API. Derived from a battle-tested housekeeping checklist.

## Contents

* `HOUSEKEEPING_CHECKLIST.md` - The original Node.js specific checklist that started it all.
* `.claude/skills/run-legacy-housekeeping/SKILL.md` - The generalized, language/framework-agnostic skill instructing Claude on how to approach refactoring and auditing dead code.
* `.claude/skills/run-legacy-housekeeping/driver.sh` - An executable CLI tool to automatically inventory endpoints, check usages against frontend code, identify leaking HTTP concerns in your model layer, and more (supports JS/TS, Python, Java, Go, Ruby).

## Usage for Claude users

If you have Claude Code, you can pull this skill directly into your projects to enforce a standard legacy code refactoring pattern. By putting the `.claude/skills/` directory in your repo root, Claude will automatically pick up the `/run-legacy-housekeeping` command.

1. Drop the `.claude` folder into the root of any repository.
2. Ask Claude Code: `/run-legacy-housekeeping` or "Please audit this codebase using the legacy housekeeping skill".

## Features

- **Endpoint Audit**: Inventory routes across all files and cross-reference them with frontend/clients to verify actual usage. YAGNI-driven approach.
- **Error Standardization**: Patterns for creating controller wrappers to enforce consistent try/catch patterns and standard `sendSuccess`/`sendError` formats.
- **De-duplication**: Strategy for consolidating combinatorial conditional queries.
- **Utility Extraction**: Identifies and extracts repeated outputs (e.g. Excel export) to shared libraries.
- **Model decoupling**: Helps identify models that accept `(req, res)` directly, decoupling them to return data so controllers can own the HTTP response lifecycle.
