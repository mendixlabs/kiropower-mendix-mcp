---
inclusion: manual
---

# Security — Steering Guide

Use this when working with module roles, entity access rules, and page/microflow access.

> **Important:** `Security$ModuleSecurity` is a singleton in every module. Read it directly instead of searching for it.
> Project security is project-level: pass `documentType: "Security$ProjectSecurity"` and omit `documentName`.
> Load the `microflow-xpath` skill before writing any `xpathConstraint`.

## Security Concepts

- **Module Roles** — permissions within a single module (e.g., `Shop.Admin`, `Shop.Viewer`)
- **User Roles** — aggregate module roles from multiple modules (e.g., `Administrator`)
- **Access Rules** — control CRUD rights on entities per module role
- **Project Security Level** — `Off`, `Prototype`, or `Production`

## Reading Security Configuration

```
ped_read_document(documentName="MyFirstModule", documentType="DomainModels$DomainModel", paths=["/entities/0/accessRules"])
```

## Entity Access Rules

Get schema:
```
ped_get_schema(elementTypes=["DomainModels$AccessRule"], kind="constructor")
```

Key properties:
- `moduleRoles` — list of module role references (e.g., `["MyFirstModule.User"]`)
- `allowCreate` — boolean
- `allowDelete` — boolean
- `memberAccesses` — per-attribute read/write permissions
- `xpathConstraint` — optional XPath filter

### Quoting inside `xpathConstraint`

The `microflow-xpath` skill covers XPath syntax. One thing it does not cover is the quoting, and it is the most common source of broken access rules.

The constraint is a string, and single quotes inside it must be **doubled**:

```
'[System.owner = ''[%CurrentUser%]'']'
```

Tokens are quoted in XPath and unquoted in Mendix expressions. In a constraint you always want the quoted form, `'[%CurrentUser%]'`.

The same property name, `xpathConstraint`, is used on `Microflows$RetrieveAction`, where no doubling applies because the value is a plain string:

```json
"xpathConstraint": "[Status = 'MyFirstModule.Status.Active']"
```

## Page Access

Set `allowedRoles` on the page document:
```
ped_update_document(
  documentName="MyFirstModule.MyPage",
  documentType="Pages$Page",
  operations=[{
    "path": "/allowedRoles",
    "operation": { "type": "set", "value": ["MyFirstModule.User", "MyFirstModule.Administrator"] }
  }]
)
```

## Microflow Access

Set `allowedModuleRoles` on the microflow:
```
ped_update_document(
  documentName="MyFirstModule.ACT_DoSomething",
  documentType="Microflows$Microflow",
  operations=[{
    "path": "/allowedModuleRoles",
    "operation": { "type": "set", "value": ["MyFirstModule.User"] }
  }]
)
```

## Common XPath Constraints for Access Rules

- `[System.owner = '[%CurrentUser%]']` — user can only access records they created
- `[Status != 'Closed']` — write access only on non-closed records
- `[MyModule.Entity_Account = '[%CurrentUser%]']` — filter by associated account

## Security Best Practices

| Guideline | Details |
|---|---|
| All persistent entities must have access rules | No entity should be accessible without explicit rules |
| Use XPath constraints | Every READ/WRITE rule on business entities should be constrained |
| Strict security mode | Enable in Production security level |
| No demo users in production | Remove or disable demo users before go-live |

## Common Mistakes

1. Creating module roles before the module exists — create the module first
2. Forgetting to grant microflow access — pages can call microflows the user can't execute
3. Entity access without proper member rights — use `READ *` for all members
4. User roles without System module roles — in Production security, user roles need at least one System module role (CE0156)
5. `Administration.Account` entity access is managed by the Administration module — do not modify it directly

## After Every Change

```
ped_check_errors(documents=[{"documentType": "DomainModels$DomainModel", "documentName": "MyFirstModule"}])
```
