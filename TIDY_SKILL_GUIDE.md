# Backend API Housekeeping & Refactoring Checklist

> A reusable guide to audit, clean, and refactor a microservice. Derived from cleanup work on legacy backend systems.

> **ℹ️ Now part of the `/tidy-code` plugin as two independent subskills:**
> - **`audit-api`** — Inventory, endpoint audit, dead code detection, YAGNI framework
> - **`refactor-api`** — Error standardization, de-duplication, utility extraction
>
> Use them individually as needed depending on your current phase.

## 1. Endpoint Audit (Dead Code Removal)

### 1.1 Inventory All Routes
Read all route files and compile a complete endpoint list:
```bash
cat routes/*_routes.js | grep -E "(get|post|put|patch|delete)\('"
```

### 1.2 Cross-reference with API Gateway & Frontend
For each endpoint, check the API gateway (e.g., KrakenD) configuration to understand how frontend paths map to this microservice:

```bash
# Example: Find all endpoints mapped to this service
grep -B2 -A5 "http://back-service-name:3000" /path/to/krakend/krakend.stg.json
```

Once the prefix is identified (e.g., `users-api/`), search all frontend repos:
```bash
grep -r "endpointName" /path/to/frontend1 /path/to/frontend2
```
Determine:
- Is it called? (Yes/No)
- Does the call use the correct gateway prefix? (`users-api/v1/...`)
- Is the call active or commented out?

**Decision matrix:**
| FE Prefix | Gateway Prefix | Verdict |
|-----------|--------------------|---------|
| `users-api/...` | `users-api/...` | **Keep** — frontend hits this service |
| `orders-api/...` | `users-api/...` | **Remove** — frontend hits another service entirely |
| `payments-api/...` | `users-api/...` | **Remove** — frontend hits another service entirely |
| No match | any | **Audit** — confirm if service-to-service or truly dead |

### 1.3 Remove Dead Code
For confirmed dead endpoints:
1. Remove route line from route file
2. Remove controller export from controller file
3. Remove model function from model file
4. Verify syntax: `node -c path/to/file.js`

> **⚠️ Caution:** Avoid one-shot script refactors on model files. The model returns may include extra keys alongside `data` (like custom metadata, pagination flags, computed fields) that the frontend depends on. Always trace the full response shape before changing.

---

## 2. Standardize Controller Error Handling

### 2.1 Create a Controller Helper
File: `utils/controllerHelper.js`

```javascript
const logger = require('./logger');

exports.wrap = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch((err) => {
      logger.error('Controller error', err);
      res.status(500).json({
        status: 500,
        message: err.message || 'Internal server error',
      });
    });
  };
};

exports.sendSuccess = (res, data) => {
  // Handle model responses that include their own status: { status: 200, data: [...] }
  if (data && typeof data === 'object' && data.status) {
    const statusCode = data.status;
    return res.status(statusCode).json(data);
  }

  // Handle raw data
  return res.status(200).json({
    status: 200,
    message: 'Success',
    data,
  });
};
```

### 2.2 Apply to All Controllers
Pattern — before:
```javascript
exports.getData = async (req, res) => {
  try {
    let result = await model.getData(req.params.id);
    res.status(result.status).json(result).end();
  } catch (err) {
    res.status(500).json({ status: 500, error: err });
  }
};
```

Pattern — after:
```javascript
const { wrap, sendSuccess } = require('../utils/controllerHelper');

exports.getData = wrap(async (req, res) => {
  let result = await model.getData(req.params.id);
  return sendSuccess(res, result);
});
```

### 2.3 Update Models to Accept `req` instead of `(req, res)`
If a model function previously accepted `(req, res)` and wrote to `res` directly, change it to accept only `req` (or extracted params) and `return` the result.

---

## 3. Eliminate Duplicated Code

### 3.1 Look for Combinatorial If/Else
Common pattern — 8-10 duplicated query branches for every permutation of 3 booleans.

**Before:**
```javascript
if (a && b && c) {
  where = { [Op.and]: [condA, condB, condC] };
} else if (a && b && !c) {
  where = { [Op.and]: [condA, condB] };
} // ... 8 more branches
```

**After:**
```javascript
const conditions = [];
if (a) conditions.push(condA);
if (b) conditions.push(condB);
if (c) conditions.push(condC);
const where = conditions.length > 0 ? { [Op.and]: conditions } : {};
```

### 3.2 Extract Shared Utilities
If the same Excel/CSV/PDF generation logic appears in multiple controllers:
```javascript
// utils/excel.js
exports.exportToExcel = (res, sheetName, columns, rows, filename) => {
  const workbook = new ExcelJS.Workbook();
  const worksheet = workbook.addWorksheet(sheetName);
  // ... build headers from columns, populate rows
  return workbook.xlsx.write(res).then(() => res.end());
};
```

---

## 4. Refactor & Document

### 4.1 Create api_review.md
For tracking progress across a full codebase cleanup, create an audit file:

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

### 4.2 Track by Priority
| Rating | Score | Criteria |
|--------|-------|----------|
| 🚨 WORST | 1-3 | God functions, over-engineering, DRY violations |
| ⚠️ POOR | 4-5 | HTTP leaking, magic numbers, poor REST design |
| 🟡 FAIR | 6-7 | Acceptable but structural improvements remain |
| 🟢 BEST | 8-10 | Clean, simple, well-separated |

---

## 5. YAGNI Decision Framework
Before refactoring any endpoint, ask:
1. Is the frontend actually calling this? If no → **Remove**
2. Is it called with a different prefix? If yes → **Remove** (belongs to another service)
3. Does the refactor change the JSON response shape? If yes → **Stop** (verify frontend expectations first)
4. Are there extra keys (metadata, flags) alongside `data`? If yes → **Preserve** or test thoroughly

---

## 6. Housekeeping Audit Checklist
*(For use with `audit-api`)*

- [ ] **Inventory:** List all routes across all route files
- [ ] **Cross-reference:** Search all frontend repos for each endpoint name + prefix
- [ ] **Remove dead endpoints:** Delete routes, controllers, and models for confirmed dead endpoints
- [ ] **Cleanup check:** Verify no unused imports, unused variables, dead code, commented-out code, debug logs, or unsafe duplication
- [ ] **Formatting check:** Confirm formatter applied, linter passes, type check passes, naming consistent
- [ ] **Dependencies check:** Ensure no unused deps, deprecated deps reviewed, unused config removed, sample env/docs current
- [ ] **Docs check:** Verify outdated comments removed, README/docs current, TODO/FIXME reviewed
- [ ] **Safety check:** Confirm no business logic change, no API contract change, no DB schema change, no auth change
- [ ] **Verify:** Build and tests pass

## 7. Refactoring Checklist
*(For use with `refactor-api`)*

- [ ] **Scope:** Scope defined, affected modules listed, no unrelated feature changes
- [ ] **Contract Safety:** Request/response structure, status codes, error format unchanged
- [ ] **Business Logic:** Logic, validation, permissions, DB behavior, external services preserved
- [ ] **Code Structure:** Clear responsibilities per layer (controller/service/repository), large functions split, module boundaries clear
- [ ] **Standardize error handling:** Create controller wrapper, apply `wrap()` + `sendSuccess()` to all handlers
- [ ] **De-dup:** Consolidate duplicated query branches into dynamic builders
- [ ] **Extract:** Create shared utilities for repeated patterns (Excel, CSV, pagination)
- [ ] **Testing:** Unit, integration, API, and regression tests pass
- [ ] **Review:** Risk areas documented, rollback plan available, CI/CD passes
