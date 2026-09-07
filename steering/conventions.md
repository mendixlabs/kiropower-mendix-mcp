---
inclusion: manual
---

# Naming Conventions — Steering Guide

Load this before creating any document in a module that has no established pattern.

The server skills route on these prefixes but never define them. `folder-structure` sends `VAL_` and `SUB_` to the entity folder and `ACT_` and `DS_` to `<Entity>/Pages`, yet nothing tells you what to name a new microflow in the first place. The system prompt only says "follow existing naming patterns," which gives you nothing on a new module.

## Microflow Prefixes

| Prefix | Purpose | Layer | Example |
|---|---|---|---|
| `ACT_` | User-triggered action | UI | `ACT_Customer_Save` |
| `DS_` | Data source, returns object or list | UI | `DS_Customer_GetAll` |
| `NAV_` | Navigation-only flow | UI | `NAV_Customer_Overview` |
| `SUB_` | Sub-microflow called by other microflows | Data | `SUB_SendNotification` |
| `VAL_` | Validation, returns Boolean | Data | `VAL_Customer_Save` |
| `OCH_` | On-change handler | Data | `OCH_Customer_Name` |
| `OLE_` | Object event handler (before/after commit or delete) | Data | `OLE_Customer_BeforeCommit` |

This is the set `folder-structure` recognises. Do not invent new prefixes; a prefix the skill does not know gets placed wrong.

**`ACT_` and `DS_` must not contain business logic.** They may only call client activities (close page, download file, show home page, show message, show page) and microflow calls. Put the logic in a `SUB_` and call it.

## Entities and Attributes

- Entities: singular, PascalCase (`Customer`, `OrderLine`)
- Attributes: PascalCase (`FirstName`, `OrderDate`)
- Boolean attributes: prefix with `Is`, `Has`, or `Can` (`IsActive`, `HasChildren`)
- Associations: `{FromEntity}_{ToEntity}` (`Order_Customer`)

## Pages

| Pattern | Example |
|---|---|
| Overview page | `Customer_Overview` |
| Create/edit page | `Customer_NewEdit` |
| Detail page | `Customer_Detail` |
| Popup dialog | `Customer_Confirm` |

## Reserved Words

Document and element names must match `^[a-zA-Z_][a-zA-Z0-9_]*$`, so the underscore style above is safe.

Separately, a set of words can never be used for any element name, in any casing: Java keywords, plus `type`, `MendixObject`, `changedby`, `changeddate`, `context`, `createddate`, `currentUser`, `empty`, `guid`, `id`, `object`, `owner`, `submetaobjectname`, `con`, and predefined variables like `currentDeviceType`, `currentIndex`, `currentSession`, `latestError`, `latestHttpResponse`.

Use an alternative even when the user asks for a reserved word. A `Type` attribute on `Customer` becomes `CustomerType`.
