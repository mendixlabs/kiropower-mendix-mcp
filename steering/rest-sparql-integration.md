---
inclusion: manual
---

# REST Integration Tips for Complex APIs

Practical guidance for REST integrations that go beyond standard JSON APIs — including SPARQL endpoints, APIs requiring special authentication, and complex nested JSON responses.

## Use Inline REST CALL for Special Auth

When the REST API uses Basic Auth with a password containing special characters (`@`, `#`, `$`, etc.), the REST Client document may silently fail to authenticate. Use an inline REST CALL action inside the microflow instead:

- Inline REST CALL lets you construct the `Authorization` header manually
- Build the header value: `"Basic " + base64encode(username + ":" + password)`
- Avoids the REST Client's credential escaping issues

Use the REST Client approach only for standard username/password credentials without special characters.

## Use Persistent Entities for DataGrid Data Sources

When displaying REST results in a DataGrid2:
- Import data into **persistent entities** (stored in the database)
- DataGrid2 with `DataSource: DATABASE` queries the database — it cannot query in-memory non-persistent objects

**Pattern:**
1. Microflow calls REST API → imports into persistent entities (commit with events)
2. Page shows a DataGrid2 with `DatabaseSource` XPath pointing to those entities

Use non-persistent entities only for forms and DataViews where you control the data source via a microflow.

## JSLT Transformers for Nested JSON

SPARQL, GraphQL, and many RPC-style APIs return deeply nested JSON. Flattening with import mappings alone creates complex entity hierarchies. Use a JSLT transformer to reshape the JSON before mapping:

```jslt
// Flatten SPARQL results array to a flat list
{
  "items": [
    for (.results.bindings) {
      "name": .name.value,
      "value": .value.value,
      "timestamp": .timestamp.value
    }
  ]
}
```

Then create a simple JSON structure and import mapping for the flat `items` array.

## Watch Out for Timestamp Detection

When a JSON structure is created from a sample payload, Mendix auto-detects ISO 8601 strings (e.g., `"2024-01-15T10:30:00Z"`) as `DateTime` attributes. If your import mapping entity uses `String` for that field, the mapping will fail.

**Fix:** After creating the JSON structure, check auto-detected types and change any `DateTime` fields to `String` if your entity models them as strings.

## SPARQL POST Request Body with Braces

SPARQL queries contain `{` and `}` which Mendix treats as template placeholders. To pass literal braces in a REST CALL body template, use parameter substitution:

- Pass `{` as the value of a placeholder parameter
- Example: `WITH ({1} = '{')`

This avoids the template engine interpreting your SPARQL braces as variable slots.

## Avoid Rapid Drop/Recreate of Entities

If you drop an entity and immediately recreate it with the same name:
- Old association GUIDs in the domain model may still reference the deleted entity
- This can corrupt the MPR and cause Studio Pro errors

Instead: modify the entity in place using `ped_update_document` to add/remove attributes. Only drop and recreate if you need to change the entity's generalization.

## MCP Tools for REST Microflows

```
ped_get_schema(["Microflows$HttpConfiguration", "Microflows$CallRestAction"], kind: "constructor")
                                                   → batch both schemas in one call
ped_create_document([{documentType: "Microflows$Microflow", ...}]) → create microflow
ped_update_document("Microflows$Microflow", "Module.MF", ops)      → add REST action
ped_check_errors([...])                                            → validate
```

## Checklist

- [ ] APIs with special-character credentials use inline REST CALL, not REST Client
- [ ] REST results displayed in DataGrid use persistent entities committed to DB
- [ ] Complex nested JSON is flattened with JSLT before import mapping
- [ ] ISO timestamp fields in JSON are checked — set to String in entities if not used as dates
- [ ] SPARQL/template brace conflicts resolved via parameter substitution
- [ ] Entities are modified in place rather than dropped and recreated
