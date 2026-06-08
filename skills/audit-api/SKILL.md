---
name: audit-api
description: Use when you need to audit an API service to find dead code, inventory endpoints, or cross-reference usage with consumer applications. Applies to any language/framework.
---

# API Auditing & Dead Code Removal

A reusable, language-agnostic checklist to audit and inventory a backend API or microservice. Drive the audit via `driver.sh` (available on `$PATH`) and follow the patterns below.

All paths in this skill are relative to the **service repo root** (the project being audited).

## Prerequisites

```bash
# Language-specific syntax checkers — use whatever applies:

# JavaScript / Node
sudo apt-get install -y nodejs

# Python
sudo apt-get install -y python3 python3-pip

# Go
sudo apt-get install -y golang-go

# Java
sudo apt-get install -y default-jdk
```

## Setup

```bash
# Run the audit driver to inventory all endpoints, controllers, and models:
driver.sh inventory ./src
# Replace ./src with the path to your source code
```

## Process

Run each section below **in order**. Every claim ("this endpoint is dead", "this condition is duplicated") must be verified — never infer.

---

### 1. Endpoint Audit (Dead Code Removal)

#### 1.1 Inventory All Routes

Use the driver to find all HTTP method definitions:

```bash
driver.sh endpoints ./src
```

This produces a list of every endpoint and the file it lives in.

#### 1.2 Cross-reference with Consumers

For each endpoint, check two things:

**A. API Gateway (if present):** Find what prefix maps to this service:

```bash
# e.g., KrakenD, Kong, Nginx, Envoy
grep -R "<service-name>" /path/to/gateway/config*
```

Identify the gateway prefix (e.g., `prefix1/`, `prefix2/`).

**B. Frontend / Clients:** Search all consumer repos for each endpoint name or path:

```bash
# Search all frontend and client repos
for endpoint in $(cut -d' ' -f1 endpoints.txt); do
  echo "--- $endpoint ---"
  grep -r "$endpoint" /path/to/frontend1 /path/to/frontend2 2>/dev/null || echo "NOT FOUND"
done
```

**Decision matrix:**

| FE calls with prefix | Gateway prefix maps to this service | Verdict |
|---|---|---|
| Correct prefix | Correct prefix | **Keep** — actively used |
| Wrong prefix | Correct prefix | **Remove** — calls another service, this is dead |
| No match found | any | **Audit** — check for service-to-service calls |
| Not found anywhere | Not found | **Remove** — truly dead |

#### 1.3 Remove Dead Code

For each confirmed dead endpoint, remove in three layers:

```
1. Remove route/endpoint declaration   (route file / router config)
2. Remove handler/controller function   (controller file)
3. Remove data-access / model function  (model / DAL file)
```

**⚠️ Critical: Never change response shapes without verifying the consumer first.** Before removing or refactoring any model/controller:
- Trace the full response shape
- Check for extra keys alongside `data` (e.g., metadata, flags, computed fields)
- If the consumer depends on extra keys, preserve them

---

### 2. YAGNI Decision Framework

Before touching any endpoint, ask these questions **in order**:

1. **Is the frontend/client actually calling this?** If **No** → **Remove**
2. **Is it called with a different prefix?** If **Yes** → **Remove** (belongs to another service)
3. **Does the refactor change the JSON response shape?** If **Yes** → **Stop** (verify consumer expectations first)
4. **Are there extra keys alongside `data`?** (metadata, flags, computed fields) If **Yes** → **Preserve** or test thoroughly

---

### 3. General Code Health Audit

Beyond endpoint dead code, audit the broader codebase for hygiene issues organized by priority.

#### 🔴 High Priority

Items with high impact — address these first:

- [ ] **Remove dead code**: Delete functions, modules, and conditionals that are never reached
- [ ] **Remove unused imports**: Strip out unused dependencies and import statements
- [ ] **Remove unused variables**: Delete declared but never-used variables and parameters
- [ ] **Remove duplicated logic**: Consolidate repeated code blocks into shared functions
- [ ] **Remove commented-out code**: Delete blocks left as comments — git history preserves them
- [ ] **Fix lint and formatting**: Run linter and auto-formatter across the codebase
- [ ] **Clean error handling**: Eliminate bare catches, unhandled rejections, and silent failures
- [ ] **Clean logging**: Remove debug/console.log noise, standardize structured logging
- [ ] **Clean config and env usage**: Centralize scattered config, validate env vars at startup
- [ ] **Check dependency security**: Run `npm audit`, `pip-audit`, or equivalent for known vulnerabilities

#### 🟡 Medium Priority

Structural improvements that reduce friction in daily development:

- [ ] **Rename unclear variables/functions**: Replace single-letter names and jargon with descriptive names
- [ ] **Split large functions**: Break god functions into focused, testable units
- [ ] **Reorganize folder structure**: Align directory layout with the domain boundaries
- [ ] **Update README/internal docs**: Keep setup, architecture, and API documentation current
- [ ] **Standardize response helpers**: Unify how controllers return success/error responses
- [ ] **Standardize validation pattern**: Adopt a consistent input validation approach across all endpoints
- [ ] **Standardize repository/service pattern**: Ensure data-access and business-logic layers follow a consistent contract

#### 🟢 Low Priority

Larger-scale changes that require careful planning and full regression testing:

- [ ] **Upgrade major package versions**: Bump major dependency versions (framework, database driver, etc.)
- [ ] **Change ORM structure**: Migrate to a different data-access approach or ORM version
- [ ] **Change framework architecture**: Adopt a new framework version or architectural pattern
- [ ] **Rewrite module boundaries**: Re-align module responsibilities after the domain has been understood
- [ ] **Change API versioning**: Adopt a new API versioning strategy
- [ ] **Replace logging library**: Switch logging libraries after evaluating alternatives

> **🔄 Circular reference:** Items above the dashed line can be escalated to the `refactor-api` skill for implementation. Items below involve structural changes best managed as separate initiatives.

---

### 4. Track Progress

Create `api_review.md` in the repo root to track endpoint-by-endpoint status:

```markdown
# API Endpoints Review
**Branch:** `current-branch`
**Date:** YYYY-MM-DD

## Refactor Status
- ✅ Endpoint completed
- ❌ Endpoint pending
- ℹ️ Endpoint audited and kept

## Rating Table
| Rank | Original Score | Endpoint | Controller | Status | New Score | Details |
```

| Rating | Score | Criteria |
|---|---|---|
| 🚨 WORST | 1-3 | God functions, over-engineering, DRY violations |
| ⚠️ POOR | 4-5 | HTTP leaking, magic numbers, poor REST design |
| 🟡 FAIR | 6-7 | Acceptable but structural improvements remain |
| 🟢 BEST | 8-10 | Clean, simple, well-separated |

## Gotchas

- **Response shape changes are invisible to you but break the frontend.** A model that returns `{ status, data, flag_rkl_rpl, keterangan }` has extra keys the frontend depends on. Changing to just `{ status, data }` breaks them. Always verify.
- **"Remove dead endpoints" means removing 3 layers** — route, controller, model. Miss one and the code lingers.
- **API gateway prefix matters.** An endpoint at `/users/export` that the frontend calls as `legacy-api/users/export` but the gateway routes `prefix/users/export` to this service — the frontend endpoint IS dead even though the code path exists.

## Troubleshooting

- **"I can't find the API gateway config":** Look for `krakend.json`, `kong.yml`, `nginx.conf`, `envoy.yaml`, or any reverse proxy config in CI deployment scripts or infra repos.
- **"The route file is enormous":** Focus on endpoints with obvious migration comments first.
- **"I don't know which frontend repos exist":** Check monorepo `clients/` directory, GitHub org repos, or README for published SDKs / frontend packages.