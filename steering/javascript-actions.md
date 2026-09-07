---
inclusion: manual
---

# JavaScript Actions — Steering Guide

Use this when creating or modifying JavaScript actions via the MCP server.

> **Load the `javascript-action` skill first.** It carries the metamodel-to-file sync rules, parameter and type rules, return types, platform constraints, the code file structure, and how to expose an action as a nanoflow action.

This file only adds what the skill does not: a catalogue of actions that already ship in a default project, so you check before you write.

## Check before you write

Most common client-side needs are already covered by a Marketplace module. Run `glob("/jsactions/*/actions/*.js")` to see what this project actually has, then check this list.

**`nanoflowcommons`**

| Action | Use |
|---|---|
| `Base64Encode` / `Base64Decode` | Encode and decode base64 strings |
| `GenerateUniqueID` | Generate a UUID |
| `GetGuid` | Get the GUID of a Mendix object |
| `ShowProgress` / `HideProgress` | Show and hide the loading indicator |
| `OpenURL` | Open a URL in the browser |
| `NavigateTo` | Navigate to a page |
| `Wait` | Pause execution for N milliseconds |
| `GetPlatform` | Detect web or native platform |

**`webactions`**

| Action | Use |
|---|---|
| `ScrollTo` | Scroll to a widget |
| `SetFocus` | Focus a widget |
| `ReadCookie` / `SetCookie` | Browser cookie management |

**`datawidgets`**

| Action | Use |
|---|---|
| `Export_To_Excel` | Export a data grid to Excel |
| `Reset_Filter` / `Reset_All_Filters` | Reset data grid filters |
| `Set_Filter` | Set a filter programmatically |
| `Clear_Selection` | Clear the data grid selection |

These modules are not writable. Call the action, do not edit it.

## File location

```
/jsactions/<module_name_lowercase>/actions/<ActionName>.js
```

Example: `/jsactions/myfirstmodule/actions/MyAction.js`
