---
inclusion: manual
---

# OData Data Sharing Between Mendix Apps

Use this when publishing or consuming data between Mendix apps via OData services.

## When to Use

- Exposing app data to other Mendix apps without direct database access
- Creating a stable API contract that survives internal schema changes
- Building read-only or read-write inter-app integrations

## Architecture: Producer / Consumer

```
[Producer App]
  Persistent entities
       ↓
  View entities (stable API layer)
       ↓
  Published OData service
       ↓
[Consumer App]
  External entities (imported from OData)
       ↓
  Pages / Microflows
```

**Why view entities?** They decouple your internal domain model from the external contract. Rename a column, add a table, or restructure associations — consumers don't break.

## Module Naming Convention

| Module | Purpose |
|--------|---------|
| `Shop` | Internal domain model |
| `ShopApi` | OData producer — view entities + published service |
| `ShopClient` | OData consumer — external entities + client |

## MCP Tools

```
read_skill([{skillName: "view-entities"}])                    → mandatory before any view entity work
ped_read_document("DomainModels$DomainModel", moduleName)     → inspect domain model
ped_get_schema(["DomainModels$Entity"], kind: "constructor")  → get entity schema (for view entities)
ped_update_document(documentType, documentName, operations)   → write the OQL onto the source document
ped_check_errors([{documentType, documentName}])              → validate
```

There are no `oql_*` tools. They were removed. See `oql-queries.md`.

## Building the Producer

### Step 1: Create view entities

View entities flatten joins, filter datasets, and compute fields. Always back them with an OQL query. Load the `view-entities` skill, then write the query through `ped_update_document`. See `oql-queries.md` for the full sequence and OQL syntax rules.

Example OQL for a flattened order view:
```sql
SELECT o.OrderId AS OrderId, o.OrderDate AS OrderDate,
       c.Name AS CustomerName, o.TotalAmount AS TotalAmount
FROM Shop.Order AS o
INNER JOIN o/Shop.Order_Customer/Shop.Customer AS c
WHERE o.Status = 'Confirmed'
```

### Step 2: Publish OData service

Create a published OData service in the `ShopApi` module and expose the view entities. Set access permissions to a dedicated module role (e.g., `ShopApi.DataConsumer`).

### Step 3: Grant security

Create a user role that includes `ShopApi.DataConsumer` module role. Only grant this to the integration user.

## Building the Consumer

1. Create an OData client in the `ShopClient` module pointing to the producer URL
2. Import external entities from the OData contract
3. Use the external entities in pages (`DataSource: DATABASE`) and microflows

## Read-Write Extension

For write-back, the OData service can expose microflow handlers for INSERT, UPDATE, DELETE that delegate to the persistent entity domain.

## Security Checklist

- [ ] View entities expose only needed attributes — no internal IDs or sensitive data
- [ ] Published service has authentication (Basic auth or token)
- [ ] Access restricted to a dedicated module role
- [ ] Consumer app has the correct credentials configured
- [ ] Run `ped_check_errors` on all view entities and the OData service after changes
