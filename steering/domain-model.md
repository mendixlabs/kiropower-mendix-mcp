---
inclusion: manual
---

# Domain Model — Steering Guide

Use this when creating or modifying entities, attributes, associations, and enumerations.

> **Important:** Load the `folder-structure` MCP skill before creating any document, and `conventions.md` for entity, attribute, and association naming.
> The domain model is a singleton that always exists. Never create one, never pass it to `ped_find_document`, and use the module name alone as `documentName`.

## Reading the Domain Model

```
ped_read_document(documentName="MyFirstModule", documentType="DomainModels$DomainModel")
```

The result contains:
- `entities[]` — list of entity stubs; re-read individual paths to expand
- `associations[]` — associations within the module
- `crossAssociations[]` — associations to entities in other modules

## Creating Entities

Always get the schema first:
```
ped_get_schema(elementTypes=["DomainModels$Entity"], kind="constructor")
```

Then add via `ped_update_document` with an `add` operation on `/entities`.

### Attribute Types
`Binary`, `Boolean`, `DateTime`, `HashedString`, `String`, `Decimal`, `Integer`, `Long`, `AutoNumber`, `Enumeration`

For `Enumeration` type, set `enumerationName` to the fully qualified enum name (e.g., `"MyFirstModule.Status"`). The enumeration must already exist.

### Entity Locations
Set `location` as a point `{"x": 100, "y": 100}` to position entities on the canvas. Space entities ~200px apart to avoid overlap.

## Creating Enumerations

Check first:
```
ped_find_document(moduleName="MyFirstModule", documentType="Enumerations$Enumeration")
```

Get schema:
```
ped_get_schema(elementTypes=["Enumerations$Enumeration"], kind="constructor")
```

Create:
```
ped_create_document(documents=[{
  "documentType": "Enumerations$Enumeration",
  "moduleName": "MyFirstModule",
  "documentName": "Status",
  "documentContent": {...},
  "folderPath": "Enumerations"
}])
```

## Creating Associations

Get schema:
```
ped_get_schema(elementTypes=["DomainModels$Association"], kind="constructor")
```

Add via `ped_update_document` on `/associations`. Key properties:
- `parent` — reference to the "one" side entity (e.g., `"MyFirstModule.Customer"`)
- `child` — reference to the "many" side entity
- `type` — `"Reference"` (many-to-one) or `"ReferenceSet"` (many-to-many)
- `owner` — `"Default"` (child owns) or `"Both"` (both own, for ReferenceSet)

**Always add entities before associations** — associations reference entities by ID.

## User Associations

Never associate to `System.User`. Always use `Administration.Account`:
```json
{ "parent": "Administration.Account", "child": "MyFirstModule.MyEntity" }
```

## System Entity Inheritance

To create file/image entities, set `generalization`:
- `System.FileDocument` — for file storage
- `System.Image` — for image storage (extends FileDocument)

## Creation Order

Always create in dependency order:
1. Enumerations (no dependencies)
2. Entities (may reference enumerations)
3. Associations (reference entities)
4. Microflows (reference entities)
5. Pages (reference microflows and entities)

## After Every Change

```
ped_check_errors(documents=[{"documentType": "DomainModels$DomainModel", "documentName": "MyFirstModule"}])
```
