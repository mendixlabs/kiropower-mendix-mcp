---
inclusion: manual
---

# Navigation — Steering Guide

Use this when setting up or modifying navigation.

> **Load the `navigation` skill before any navigation change, however trivial.** Add `glyph-icons` when setting menu icons.
> That skill covers profiles (add, update, remove), menus, menu item grouping, and page parameters in menu item actions. This file only adds what it does not: role-based routing.

## The navigation document is project-level

Pass `documentType: "Navigation$NavigationDocument"` and **omit `documentName` entirely**. Do not search for it in a module.

## Role-Based Home Pages

The skill does not cover these. Each navigation profile has a `roleBasedHomePages` list alongside its default `homePage`. Entries override the default per user role, and the first matching role wins, so order matters.

```
ped_read_document("Navigation$NavigationDocument", paths=["/profiles/0"])
```

Setting the default home page:

```
ped_update_document(
  documentType="Navigation$NavigationDocument",
  operations=[{
    "path": "/profiles/0/homePage",
    "operation": { "type": "set", "value": { "$Type": "Navigation$PageSettings", "page": "MyFirstModule.Home_Web" } }
  }]
)
```

Get the schema with `ped_get_schema(["Navigation$HomePage"], kind="constructor")` before adding a role-based entry. Do not assume the shape matches `homePage`.

## Menu Item Visibility by Role

Menu items carry `allowedRoles`, which the skill also does not cover. Only users holding a listed role see the item. Reference module roles as `"ModuleName.RoleName"`.

Hiding a menu item is not security. The target page still needs its own page-level access, or a user can reach it by URL.

## Common Workflow: New Project Setup

1. Create the home page first
2. Read the navigation profile to get its current structure
3. Set the home page, then add menu items
4. Add role-based overrides last, once the roles exist
