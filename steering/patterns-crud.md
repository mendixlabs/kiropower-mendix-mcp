---
inclusion: manual
---

# CRUD Patterns — Steering Guide

Use this when implementing standard Create, Read, Update, Delete patterns in Mendix.

> Load `microflow-common` and `page-gen-common` first, and `conventions.md` for the prefix meanings.

## The rule that shapes every pattern below

The `folder-structure` skill states that `ACT_` and `DS_` microflows must not contain business logic. They may only call client activities (close page, download file, show home page, show message, show page) and other microflows.

So every `ACT_` here is a thin shell. Create, commit, delete, and default-setting all live in a `SUB_` that the `ACT_` calls. Writing a `CommitAction` directly into an `ACT_` puts it in the wrong layer and the folder placement will fight you.

## Standard CRUD Microflow Set

For an entity `MyFirstModule.Customer`:

| Microflow | Purpose | Returns |
|---|---|---|
| `DS_Customer_GetList` | Retrieve all customers | List of Customer |
| `DS_Customer_GetById` | Retrieve single customer by ID | Customer object |
| `ACT_Customer_New` | Call `SUB_Customer_Create`, open the form | Void |
| `ACT_Customer_Save` | Call `SUB_Customer_Save`, show message or close | Void |
| `ACT_Customer_Delete` | Call `SUB_Customer_Delete`, close or refresh | Void |
| `SUB_Customer_Create` | Create the object and set defaults | Customer |
| `SUB_Customer_Save` | Validate, then commit | Boolean |
| `SUB_Customer_Delete` | Delete the object | Boolean |
| `VAL_Customer_Save` | Validate required fields | Boolean |

## Save Pattern

```
ACT_Customer_Save                        (UI layer, no business logic)
1. Call SUB_Customer_Save → $Success
2. ExclusiveSplit on $Success:
   - false → ShowMessage (validation error) → End
   - true  → ClosePageAction → End

SUB_Customer_Save                        (data layer)
1. Call VAL_Customer_Save → $IsValid
2. ExclusiveSplit on $IsValid:
   - false → return false
   - true  → CommitAction (with events) → return true
```

## Create Pattern

```
ACT_Customer_New
1. Call SUB_Customer_Create → $Customer
2. ShowPageAction (Customer_NewEdit, $Customer)

SUB_Customer_Create
1. CreateObjectAction (Customer, uncommitted)
2. Set defaults on the new object
3. Return the object
```

Create the object uncommitted so cancelling the form leaves nothing behind.

## Validation Pattern

```
VAL_Customer_Save
1. $IsValid = true
2. Check each required field:
   IF field empty → $IsValid = false → show validation feedback
3. Return $IsValid
```

Enumeration comparisons use fully qualified values:
```
CORRECT: $Task/Status = Module.TaskStatus.Completed
WRONG:   $Task/Status = 'Completed'
```

## Delete Pattern

```
ACT_Customer_Delete
1. Call SUB_Customer_Delete → $Success
2. ClosePageAction

SUB_Customer_Delete
1. DeleteAction (delete Customer)
2. Return true
```

## Overview Page Pattern

1. Create `Customer_Overview` with `Atlas_Core.Atlas_Default` layout
2. DataGrid2 with DatabaseSource pointing to `MyFirstModule.Customer`
3. Columns for key attributes
4. "New" button → `ACT_Customer_New`
5. Row "Edit" button → opens `Customer_NewEdit` with selected object
6. Row "Delete" button → `ACT_Customer_Delete` with selected object

## Detail/Form Page Pattern

1. Create `Customer_NewEdit` with `Atlas_Core.PopupLayout` layout
2. DataView with PageParameterSource bound to the Customer page parameter
3. Input widgets per attribute type
4. "Save" button → `ACT_Customer_Save`
5. "Cancel" button → ClosePageAction

## Best Practices

- Keep `ACT_` thin: client activities and microflow calls only
- Always validate before commit, inside the `SUB_`
- Commit WITH EVENTS so event handlers fire
- Create uncommitted, commit on save
- `SUB_` microflows return a Boolean success status; `ACT_` microflows return Void
