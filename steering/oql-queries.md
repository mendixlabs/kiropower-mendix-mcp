---
inclusion: manual
---

# OQL Queries & View Entities — Steering Guide

Use this when creating or updating view entities with OQL queries.

> **Load the `view-entities` skill first** for the OQL syntax reference, tool sequences, and constraints.
> **Then ignore its `oql_generate` sections.** See below.

## The `oql_generate` trap

The `view-entities` skill has sections titled "When to Call `oql_generate`" and "Tool Inputs / `oql_generate`". That tool is **not available over MCP**. Calling it fails:

```
MCP error -32602: Tool oql_generate not found
```

The skills are written for Maia running inside Studio Pro. The MCP server exposes a subset of Maia's tools, so a skill can describe something `tools/list` does not carry. `oql_generate` is the only known case as of 11.14. When a skill names a tool you have not seen in `tools/list`, trust `tools/list`.

**What this means in practice:** you write the OQL yourself. There is no natural-language generation step over MCP. Everything else in the skill (syntax, sequences, constraints) still applies.

## Working sequence

```
read_skill([{skillName: "view-entities"}])                             → syntax reference
ped_find_document(moduleName, "DomainModels$ViewEntitySourceDocument") → check if exists
ped_get_schema(["DomainModels$ViewEntitySourceDocument"], kind: "element")
ped_read_document("DomainModels$ViewEntitySourceDocument", name)       → read existing OQL
ped_create_document([{...}])                                           → create the source document
ped_update_document(documentType, documentName, operations)            → write the OQL
ped_check_errors([{documentType, documentName}])                       → validate
```

## Creating a View Entity

1. Create the `ViewEntitySourceDocument`:

```
ped_create_document(documents=[{
  "documentType": "DomainModels$ViewEntitySourceDocument",
  "moduleName": "MyFirstModule",
  "documentName": "CustomerSummaryView",
  "documentContent": {"$Type": "DomainModels$ViewEntitySourceDocument", "name": "CustomerSummaryView"}
}])
```

2. Add the entity to the domain model referencing it:

```json
{
  "$Type": "DomainModels$Entity",
  "name": "CustomerSummaryView",
  "source": {
    "$Type": "DomainModels$OqlViewEntitySource",
    "sourceDocument": "MyFirstModule.CustomerSummaryView"
  },
  "attributes": []
}
```

3. Write the OQL onto the source document with `ped_update_document`. Get the exact property path from `ped_get_schema` with `kind: "element"`. Do not guess it.

## OQL Syntax Rules

The `view-entities` skill carries the full reference. These are the rules that most often bite:

### All SELECT columns MUST have explicit AS aliases
```sql
-- WRONG
SELECT fl.ForecastDate, fl.ProjectedIncome FROM Finance.ForecastLine AS fl

-- CORRECT
SELECT fl.ForecastDate AS ForecastDate, fl.ProjectedIncome AS ProjectedIncome
FROM Finance.ForecastLine AS fl
```

### No ORDER BY or LIMIT at the view level
Both are only valid inside correlated subqueries.

### Aggregate functions must be lowercase
```sql
sum(o.Amount)   avg(o.Amount)   count(t.ID)   max(o.Date)   min(o.Price)
```

### Use count(entity.ID) not count(*)
```sql
count(t.ID)   -- CORRECT
count(*)      -- NOT SUPPORTED
```

### Division uses colon, not slash
```sql
amount : quantity AS price    -- CORRECT
amount / quantity AS price    -- WRONG
```

### Enumeration comparisons use string literals
```sql
WHERE t.Status = 'Active'               -- CORRECT
WHERE t.Status = Finance.Status.Active  -- WRONG
```

### Use != not <>
```sql
WHERE t.Status != 'Voided'    -- CORRECT
WHERE t.Status <> 'Voided'    -- WRONG
```

## Common Patterns

**Date aggregation:**
```sql
SELECT datepart(YEAR, t.Date) AS Year, sum(t.Amount) AS Total
FROM Finance.Transaction AS t
GROUP BY datepart(YEAR, t.Date)
```

**Association navigation:**
```sql
SELECT o.OrderId AS OrderId, c.Name AS CustomerName
FROM Shop.Order AS o
INNER JOIN o/Shop.Order_Customer/Shop.Customer AS c
```

**Correlated subquery (latest price):**
```sql
SELECT p.Name AS Name,
  (SELECT pr.Price FROM Shop.Price AS pr
   WHERE pr/Shop.Price_Product = p.ID
   ORDER BY pr.StartDate DESC LIMIT 1) AS LatestPrice
FROM Shop.Product AS p
```

## After Every Change

```
ped_check_errors(documents=[
  {"documentType": "DomainModels$DomainModel", "documentName": "MyFirstModule"},
  {"documentType": "DomainModels$ViewEntitySourceDocument", "documentName": "MyFirstModule.CustomerSummaryView"}
])
```
