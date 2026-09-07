---
inclusion: manual
---

# REST Integration

Use this when calling external REST APIs from Mendix microflows.

## Two Approaches

| Approach | When to Use |
|----------|-------------|
| REST Client + SEND REST REQUEST | Structured APIs with multiple reusable operations |
| Inline REST CALL | One-off calls, dynamic URLs, quick prototyping |

## Approach 1: REST Client

### Create the REST client

Use `ped_create_document` with document type `ConsumedRestServices$ConsumedRestService`. Define the base URL and authentication once. Each operation in the client specifies the HTTP method, endpoint path, request body, and response handling.

### Call it from a microflow

In the microflow, use `Microflows$SendExternalObjectAction` (get schema first). After the call, the response is in `$latestHttpResponse`:

- `$latestHttpResponse/Content` — response body as String
- `$latestHttpResponse/StatusCode` — HTTP status code as Integer

## Approach 2: Inline REST CALL

Use directly inside a microflow action. Supports GET, POST, PUT, DELETE with inline headers, authentication, body content, and error handling.

Add `ON ERROR CONTINUE` to handle failures gracefully rather than crashing the microflow.

## JSON Structures and Import Mappings

When the API returns structured JSON, use this pipeline:

```
REST API response
      ↓
JSON Structure (schema definition)
      ↓
Import Mapping (JSON → Mendix entity)
      ↓
Mendix entities (non-persistent for read-only, persistent if storing)
```

### JSON Structure

Defines the shape of the JSON payload. Mendix generates an element tree from a sample JSON. Create with `ped_create_document` using type `JsonStructures$JsonStructure`.

**Key rule:** For non-persistent result entities, use unlimited String attributes (no max length).

### Import Mapping

Maps JSON fields to entity attributes. Create with `ped_create_document` using type `Mappings$ImportMapping`. Reference the JSON structure and map each element to entity attributes.

**Nested arrays:** Require intermediate container entities matching the JSON hierarchy.

### Export Mapping

Reverses the process — serializes Mendix entities back to JSON for request bodies.

## Data Transformers (Mendix 11.12+)

Use JSLT transformers to reshape complex API responses before passing them to an import mapping. This simplifies the mapping and avoids deeply nested entity structures.

Pipeline with transformer:
```
REST API → JSLT transform → Import Mapping → Entity
```

## Common Patterns

**Check response status before processing:**
```
IF $latestHttpResponse/StatusCode != 200 THEN
  show error or return empty
```

**Parse JSON to entity in one microflow:**
1. Call REST (inline or via client)
2. Import from mapping with `$latestHttpResponse/Content`
3. Return the entity or list

**Build dynamic query strings:**
Use string concatenation in the URL for query parameters. Encode special characters if needed.

## Checklist

- [ ] GET requests have an Accept header
- [ ] POST/PUT/PATCH have a Content-Type header and request body
- [ ] Response status is checked before using the body
- [ ] Non-persistent entity attributes use unlimited String (no max length)
- [ ] Run `ped_check_errors` on microflows after adding REST call activities
