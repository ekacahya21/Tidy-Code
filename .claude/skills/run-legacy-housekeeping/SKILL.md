---
name: run-legacy-housekeeping
description: Use when asked to clean up, audit, or refactor a legacy backend API service — removing dead code, standardizing error handling, eliminating duplication, and extracting shared utilities. Applies to any language/framework.
---

# Legacy API Housekeeping & Refactoring

A reusable, language-agnostic checklist to audit, clean, and refactor a legacy microservice or backend API. Drive the audit via `.claude/skills/run-legacy-housekeeping/driver.sh` and follow the patterns below.

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
.claude/skills/run-legacy-housekeeping/driver.sh inventory ./src
# Replace ./src with the path to your source code
```

## Process

Run each section below **in order**. Every claim ("this endpoint is dead", "this condition is duplicated") must be verified — never infer.

---

### 1. Endpoint Audit (Dead Code Removal)

#### 1.1 Inventory All Routes

Use the driver to find all HTTP method definitions:

```bash
.claude/skills/run-legacy-housekeeping/driver.sh endpoints ./src
```

This produces a list of every endpoint and the file it lives in.

#### 1.2 Cross-reference with Consumers

For each endpoint, check two things:

**A. API Gateway (if present):** Find what prefix maps to this service:

```bash
# e.g., KrakenD, Kong, Nginx, Envoy
grep -R "<service-name>" /path/to/gateway/config*
```

Identify the gateway prefix (e.g., `webform-read/`, `izin-api/`).

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

### 2. Standardize Controller Error Handling

#### 2.1 Create a Controller Wrapper

**Language-agnostic pattern:** Extract a reusable wrapper that:
1. Catches all errors from handler functions
2. Logs the error
3. Returns a consistent error response

```pseudocode
FUNCTION wrap(handler):
    RETURN FUNCTION(request, response, next):
        TRY:
            AWAIT handler(request, response, next)
        CATCH error:
            logger.error("Controller error", error)
            response.status(500).json({
                status: 500,
                message: error.message OR "Internal server error"
            })

FUNCTION sendSuccess(response, data):
    IF data has a "status" field THEN
        // Legacy model response — pass through verbatim
        response.status(data.status).json(data)
    ELSE
        response.status(200).json({
            status: 200,
            message: "Success",
            data: data
        })
```

#### 2.2 Apply to All Controllers

**Before — every handler has its own try/catch:**

```pseudocode
// In every single handler, duplicated N times
HANDLER getData(request, response):
    TRY:
        result = model.getData(request.params.id)
        response.status(result.status).json(result)
    CATCH error:
        response.status(500).json({ status: 500, error: error.message })
```

**After — one wrapper eliminates all duplication:**

```pseudocode
HANDLER getData(request, response):
    result = model.getData(request.params.id)
    sendSuccess(response, result)

// Register with wrapper:
router.get("/data/:id", wrap(getData))
```

#### 2.3 Fix Models That Accept (request, response)

If a model function previously received both `request` and `response` and wrote directly to `response`, change it to **accept only request parameters and return the result**. Controllers should own the response.

**Before:**

```pseudocode
FUNCTION legacyImport(request, response):
    data = parse(request.body)
    response.status(200).json({ status: 200, data: data })
```

**After:**

```pseudocode
FUNCTION legacyImport(request):
    data = parse(request.body)
    RETURN { status: 200, data: data }
// Controller calls sendSuccess(response, model.legacyImport(request))
```

---

### 3. Eliminate Duplicated Code

#### 3.1 Combinatorial Conditionals

Look for functions that write every permutation of N booleans as separate branches:

**Before (N booleans = 2^N branches):**

```pseudocode
IF a AND b AND c THEN
    conditions = [condA, condB, condC]
ELSE IF a AND b AND NOT c THEN
    conditions = [condA, condB]
ELSE IF a AND NOT b AND c THEN
    conditions = [condA, condC]
ELSE IF a AND NOT b AND NOT c THEN
    conditions = [condA]
...  // 2^3 = 8 branches
```

**After (2 branches, N conditions):**

```pseudocode
conditions = []
IF a THEN conditions.PUSH(condA)
IF b THEN conditions.PUSH(condB)
IF c THEN conditions.PUSH(condC)
where = conditions.NOT_EMPTY ? buildConditions(conditions) : default
```

#### 3.2 Runtime Verification

After de-duplication, run the test suite and verify output:

```bash
# Language-specific syntax check
node -c path/to/file.js    # Node/JS
python -m py_compile file.py  # Python
go vet ./...               # Go
javac File.java            # Java

# Run tests
npm test                   # Node
pytest                     # Python
go test ./...              # Go
mvn test                   # Java
```

---

### 4. Extract Shared Utilities

If the same output generation (Excel, CSV, PDF, JSON) or pagination logic appears in multiple controllers, extract it to a single shared module.

**Before:**

```pseudocode
// In controller A
FUNCTION exportAsExcel(request, response):
    data = model.getDataA()
    workbook = NEW_EXCEL_WORKBOOK()
    worksheet = workbook.ADD_SHEET("A")
    // ... 20 lines of setup
    workbook.WRITE(response)

// In controller B — same 20 lines duplicated
FUNCTION exportAsExcel(request, response):
    data = model.getDataB()
    workbook = NEW_EXCEL_WORKBOOK()
    worksheet = workbook.ADD_SHEET("B")
    // ... same 20 lines
    workbook.WRITE(response)
```

**After:**

```pseudocode
// utils/exporter.js
FUNCTION exportToExcel(response, sheetName, columns, rows, filename):
    workbook = NEW_EXCEL_WORKBOOK()
    worksheet = workbook.ADD_SHEET(sheetName)
    worksheet.HEADERS = columns
    rows.FOREACH(row => worksheet.ADD_ROW(row))
    workbook.WRITE(response)

// In any controller:
exporter.exportToExcel(response, "A", headers, data, "report.xlsx")
```

---

### 5. YAGNI Decision Framework

Before refactoring any endpoint, ask these questions **in order**:

1. **Is the frontend/client actually calling this?** If **No** → **Remove**
2. **Is it called with a different prefix?** If **Yes** → **Remove** (belongs to another service)
3. **Does the refactor change the JSON response shape?** If **Yes** → **Stop** (verify consumer expectations first)
4. **Are there extra keys alongside `data`?** (metadata, flags, computed fields) If **Yes** → **Preserve** or test thoroughly

---

### 6. Track Progress

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

---

## Checklist Template (copy to your task list)

- [ ] **Inventory:** List all routes across all route files (`driver.sh inventory`)
- [ ] **Audit:** Search all client repos for each endpoint name + prefix
- [ ] **Remove:** Delete routes, controllers, and models for dead endpoints
- [ ] **Standardize:** Create controller wrapper and apply to all handlers
- [ ] **De-dup:** Consolidate duplicated query branches into dynamic builders
- [ ] **Extract:** Create shared utilities for repeated patterns (Excel, CSV, pagination)
- [ ] **Document:** Write/update `api_review.md` with endpoint-by-endpoint status
- [ ] **Verify:** Run linter/syntax check on all modified files and run the test suite

## Gotchas

- **Response shape changes are invisible to you but break the frontend.** A model that returns `{ status, data, flag_rkl_rpl, keterangan }` has extra keys the frontend depends on. Changing to just `{ status, data }` breaks them. Always verify.
- **"Remove dead endpoints" means removing 3 layers** — route, controller, model. Miss one and the code lingers.
- **API gateway prefix matters.** An endpoint at `/users/export` that the frontend calls as `legacy-api/users/export` but the gateway routes `webform-read/users/export` to this service — the frontend endpoint IS dead even though the code path exists.
- **Don't run one-shot search-replace on model files.** The model layer accumulates the most tacit knowledge about response shapes. Trace each one manually.

## Troubleshooting

- **"I can't find the API gateway config":** Look for `krakend.json`, `kong.yml`, `nginx.conf`, `envoy.yaml`, or any reverse proxy config in CI deployment scripts or infra repos.
- **"The route file is enormous":** It's likely a legacy monolith with hundreds of endpoints. Focus on endpoints with obvious migration comments first.
- **"I don't know which frontend repos exist":** Check monorepo `clients/` directory, GitHub org repos, or README for published SDKs / frontend packages.
