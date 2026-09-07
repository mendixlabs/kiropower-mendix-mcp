---
inclusion: manual
---

# Assess Mendix Project Quality

Use this when the user asks to evaluate, audit, or health-check a Mendix project.

## Assessment Workflow

Use MCP tools to explore the project:

```
list_modules()                                          → modules plus writable/fromMarketplace flags
ped_list_folder(moduleName, folderPath?)                → list documents and subfolders
ped_read_document("DomainModels$DomainModel", moduleName) → read entities and associations
ped_find_document(moduleName, documentType)             → find microflows, pages, etc.
pg_read_page(moduleName, pageName)                      → read a page
ped_check_errors(documents)                             → check for validation errors
```

---

## Quality Guidelines Reference

### A. Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Entities | Singular, PascalCase | `Customer`, `OrderLine` |
| Attributes | PascalCase | `FirstName`, `OrderDate` |
| Boolean attributes | Prefix with Is/Has/Can/Should | `IsActive`, `HasChildren` |
| Microflows | Prefix indicating purpose | `ACT_Customer_Save`, `DS_Customer_GetAll` |
| Pages | Entity + suffix | `Customer_NewEdit`, `Order_Overview` |
| Enumerations | Prefix with ENUM_ | `ENUM_OrderStatus` |
| Snippets | Prefix with SNIPPET_ | `SNIPPET_CustomerDetails` |

**Microflow prefix reference:**

| Prefix | Purpose |
|--------|---------|
| `ACT_` | UI action (button click handler) |
| `DS_` | Data source for pages/widgets |
| `NAV_` | Navigation microflow |
| `SUB_` | Reusable submicroflow |
| `VAL_` | Validation logic |
| `BCO_` | Before commit handler |
| `ACO_` | After commit handler |
| `SCH_` | Scheduled event |
| `IVK_` | Invocable (called by external systems) |

### F. Architecture

| Guideline | Details |
|-----------|---------|
| No cross-module direct data access | Access data through microflows |
| Business key on entities | Persistent entities should have a business key attribute |
| Domain model size | Max 15 persistent entities per module |
| Entity attribute count | Max 10 attributes per entity (consider splitting) |

### G. Error Handling

- All microflows should use Stop + Log + Roll back + Show user message as default
- REST calls, web service calls, and Java actions need custom error handling
- Never use "Continue" error handling — always handle explicitly
- Log with context: include status code, response body, and input parameters

---

## Assessment Report Template

```markdown
# Mendix Project Quality Assessment

**Project:** [Name]
**Date:** [Date]

## Executive Summary

| Category | Finding |
|----------|---------|
| Naming | [summary] |
| Security | [summary] |
| Quality | [summary] |
| Architecture | [summary] |
| Performance | [summary] |

## Critical Issues (must fix)
- [list]

## High Priority (should fix)
- [list]

## Recommendations

### Top 5 Actions (Highest Impact)
1. [specific, actionable recommendation]

### Quick Wins (Low Effort, High Value)
- [list]
```
