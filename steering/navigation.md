---
inclusion: manual
---

# Navigation — Steering Guide

Use this when setting up or modifying navigation: home pages, menu items, login pages, and role-based routing.

> **Important:** Load the `navigation` MCP skill before **any** navigation change, however trivial. Add `glyph-icons` when setting menu icons.
> The navigation document is project-level: pass `documentType: "Navigation$NavigationDocument"` and omit `documentName` entirely.

## Navigation Concepts

- **Navigation Profiles** — Responsive, Phone, Tablet, and optionally Native
- **Home Page** — The default page shown after login
- **Role-Based Home Pages** — Override the default home page per user role
- **Menu Items** — Hierarchical menu tree with captions and page/microflow targets

## Finding the Navigation Document

```
ped_find_document(moduleName="MyFirstModule", documentType="Navigation$NavigationProfile")
```

Or search in the System module:
```
ped_find_document(moduleName="System", documentType="Navigation$NavigationDocument")
```

## Navigation Document Structure

Each profile contains:
- `homePage` — the default page shown after login
- `roleBasedHomePages` — list of role-specific home page overrides
- `menuItemCollection` — the menu tree
- `loginPage` — optional custom login page

## Adding a Menu Item

Get schema:
```
ped_get_schema(elementTypes=["Navigation$MenuItem"], kind="constructor")
```

Key properties:
- `caption` — display text (use `Texts$Text` element)
- `action` — the action when clicked (typically `Pages$PageClientAction` to open a page)
- `icon` — optional icon reference
- `allowedRoles` — restrict visibility by role

## Setting the Home Page

```
ped_update_document(
  documentName="...",
  documentType="Navigation$NavigationDocument",
  operations=[{
    "path": "/profiles/0/homePage",
    "operation": { "type": "set", "value": { "$Type": "Navigation$PageSettings", "page": "MyFirstModule.Home_Web" } }
  }]
)
```

## Role-Based Navigation

Menu items can be restricted by role using `allowedRoles`. Only users with matching roles will see the item. Reference module roles as `"ModuleName.RoleName"`.

## Common Workflow: New Project Setup

1. Create the home page first
2. Read the navigation profile to get its current structure
3. Update the navigation profile to set the home page and add menu items

## After Every Change

```
ped_check_errors(documents=[{"documentType": "Navigation$NavigationDocument", "documentName": "..."}])
```
