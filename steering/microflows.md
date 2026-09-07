---
inclusion: manual
---

# Microflows — Steering Guide

Use this when creating or modifying microflows via the MCP server.

> **Important:** Before creating or modifying any microflow, load the MCP skills:
> `microflow-common`, `microflow-expressions`, `microflow-xpath`
> Add `validation-microflow` for `VAL_*` flows and `microflow-unit-testing` when generating tests.
> Also read the critical sub-references from `microflow-common`:
> `references/best-practices.md`, `references/flows.md`, `references/layout.md`, `references/variable-scope.md`

## Naming Conventions

| Prefix | Purpose | Example |
|---|---|---|
| `ACT_` | User-triggered actions | `ACT_Customer_Save` |
| `SUB_` | Sub-microflows called by other microflows | `SUB_SendNotification` |
| `DS_` | Data source microflows (return object or list) | `DS_Customer_GetAll` |
| `VAL_` | Validation microflows (return Boolean) | `VAL_Customer_Save` |
| `IVK_` | Integration/REST call microflows | `IVK_ExternalApi_Sync` |

## MCP Tools

```
ped_find_document(moduleName, "Microflows$Microflow")    → check if exists first
ped_get_schema(["Microflows$Microflow"], kind: "constructor")   → schema for creating
ped_get_schema(["Microflows$Microflow"], kind: "element")       → schema for reading/updating
ped_create_document([{...}])                                    → create microflow
ped_read_document("Microflows$Microflow", "Module.MyMicroflow") → read (type first, then name)
ped_update_document("Microflows$Microflow", "Module.MyMicroflow", ops) → update
ped_check_errors([{documentType, documentName}])         → validate after every change
```

Always call `ped_find_document` first — if a match exists, read and update instead of creating.

## Canvas Layout Rules

- Happy path flows left → right
- Error/exception flows top → bottom
- Connection points: `0=top`, `1=right`, `2=bottom`, `3=left`
- Connect right side of origin (`originConnectionIndex=1`) to left side of destination (`destinationConnectionIndex=3`)
- Space objects ~150px apart horizontally

## Object Types

| Type | Use |
|---|---|
| `Microflows$StartEvent` | Every microflow needs exactly one. Never connect parameters to it. |
| `Microflows$EndEvent` | Only ONE incoming flow allowed. Use ExclusiveMerge to merge paths first. |
| `Microflows$ActionActivity` | Wraps any action (retrieve, create, change, call microflow, etc.) |
| `Microflows$ExclusiveSplit` | IF/SWITCH branching. Must cover ALL cases. |
| `Microflows$ExclusiveMerge` | Merges multiple paths back to one. Required before EndEvent when multiple paths exist. |
| `Microflows$LoopedActivity` | FOR-EACH loop. Child objects and flows are nested inside. Never connect outer flows to inner objects. |
| `Microflows$MicroflowParameterObject` | Input parameter. Never a flow destination. |
| `Microflows$InheritanceSplit` | Routes by entity subtype. Requires N subtypes + base type + empty case flows. |
| `Microflows$BreakEvent` | Breaks out of a loop. |
| `Microflows$ContinueEvent` | Continues to next loop iteration. |
| `Microflows$ErrorEvent` | Only on custom error handling flows. |
| `Microflows$Annotation` | Documentation. Connect with `Microflows$AnnotationFlow`, never SequenceFlow. |

## ExclusiveSplit Rules (Critical)

**Boolean split:** MUST have exactly two flows — `caseValue.enumerationCase` = `"true"` and `"false"`.

**Enumeration split:** MUST have one flow per enum value PLUS one mandatory `"(empty)"` flow. Missing the empty case causes a validation error.

## Flow References

Use `$id(/objectCollection/objects/N)` to reference objects by position (zero-based index).

```json
{
  "$Type": "Microflows$SequenceFlow",
  "originId": "$id(/objectCollection/objects/0)",
  "destinationId": "$id(/objectCollection/objects/1)",
  "originConnectionIndex": 1,
  "destinationConnectionIndex": 3
}
```

## Common Actions (get schema for each before use)

| Action Type | Use |
|---|---|
| `Microflows$RetrieveAction` | Retrieve objects from DB or association |
| `Microflows$CreateObjectAction` | Create a new object |
| `Microflows$ChangeObjectAction` | Change object attributes |
| `Microflows$DeleteAction` | Delete an object |
| `Microflows$CommitAction` | Commit object to DB |
| `Microflows$MicroflowCallAction` | Call another microflow |
| `Microflows$ShowMessageAction` | Show a message to the user |
| `Microflows$ShowPageAction` | Open a page |
| `Microflows$ClosePageAction` | Close the current page |
| `Microflows$LogMessageAction` | Write to the Mendix log |

## Performance Rules (Never Do These)

- **Never commit inside a loop** — accumulate in a list, commit after the loop
- **Never retrieve from DB inside a loop** — retrieve before the loop with XPath
- **Never show a message inside a loop** — collect results, show once after
- **Never delete objects one-by-one in a loop** — use Delete List after the loop
- **Never call REST/email inside a loop** — batch or move outside

## Checklist

- [ ] `ped_find_document` called before creating to avoid duplicates
- [ ] Schemas fetched with `ped_get_schema` before adding elements
- [ ] `$Type` included on all elements; `$ID` never included
- [ ] `$id(/path)` used for by-id references
- [ ] ExclusiveSplits cover all cases including `(empty)` for enums
- [ ] Every microflow has exactly one StartEvent; EndEvents each have exactly one incoming flow
- [ ] No commits, retrieves, messages, or external calls inside loops
- [ ] `ped_check_errors` run after every create/update
