---
name: "mendix-mcp"
displayName: "Mendix MCP Development"
description: "AI-assisted Mendix development against the Studio Pro MCP server (Studio Pro 11.10+, default port 7782). Build domain models, microflows, pages, workflows, security, navigation, view entities, and JavaScript actions through the ped_*, pg_*, and file tools."
keywords: ["mendix", "mcp", "microflow", "nanoflow", "entity", "domain model", "page", "datagrid", "gallery", "module", "enumeration", "association", "xpath", "oql", "atlas", "mpr", "security", "navigation", "crud", "javascript action"]
author: "Workspace Power"
---

# Mendix MCP Development Power

This power gives Kiro deep knowledge of Mendix development via the MCP server running at `localhost:7782`. It covers the full development lifecycle: domain modelling, microflows, pages, security, navigation, OQL queries, and JavaScript actions — all through structured MCP tool calls against the live `.mpr` model.

> **Using Mendix via MxCLI instead?** If your project uses the MxCLI binary and MDL scripts rather than the Studio Pro MCP server, use the [Mendix MxCLI Power](https://github.com/Rvthof/awskiro-mxcli-power) instead. This power is specifically for MCP-based development.

---

# MCP Server Connection

The MCP server is pre-configured at `localhost:7782` via `.kiro/settings/mcp.json`. No binary or CLI setup is required. All model changes are made directly through MCP tool calls.

**Requires Studio Pro 11.10 or later** (11.11+ recommended, since page-editing tools landed in 11.11). Studio Pro must be open, signed in, and online.

From 11.13 onward Studio Pro falls back to a free port when `7782` is taken, so two instances can run side by side. If the connection fails, check the port shown in the Studio Pro status bar and update the `url` in `.kiro/settings/mcp.json`.

**Do not hardcode the module list.** Call `list_modules` at the start of a session. It returns every module plus two flags: `writable` (user modules are writable, System and Marketplace modules are not) and `fromMarketplace`. Never attempt writes into a non-writable module.

## Read the server's system prompt first

The server exposes one resource, `mendix://studio-pro/system-prompt`, and the handshake instructs clients to read it before calling any tool. It carries the current PED mental model, schema conventions, reference-kind rules, reserved words, and the error-fixing protocol. Read it once per session before the first tool call.

---

# MCP Tool Reference

Verified against Studio Pro 11.14, which exposes 17 tools. Tool names change between releases. To re-check, POST a `tools/list` request to the server (see README troubleshooting).

## Skills — Load Before Building

| Tool | Purpose | Key Parameters |
|---|---|---|
| `read_skill` | Load a server-side skill, or a reference file inside one | `skills[]` of `{skillName, resourcePath?}` |

Skills carry mandatory constraints, not optional tips. Load the relevant one **before** you build, never after. The authoritative list lives in the `read_skill` tool description. Core skills in 11.14:

| Skill | Load when |
|---|---|
| `folder-structure` | Any document creation. Decides which folder the document belongs in |
| `microflow-common` | Any microflow work |
| `microflow-expressions` | Writing microflow expressions |
| `microflow-xpath` | Any XPath constraint |
| `microflow-unit-testing` | Generating unit test microflows |
| `validation-microflow` | `VAL_*` microflows, before-commit validation |
| `workflow-common` / `workflow-update` | Creating or modifying workflows |
| `page-gen-common` | Any page generation |
| `glyph-icons` | Glyph icons on pages or menus |
| `navigation` | Any navigation document change, however trivial |
| `theming` | Atlas UI theming, CSS variables, SCSS |
| `design-properties` | `design-properties.json` work |
| `view-entities` | View entities and all OQL |
| `javascript-action` | JavaScript action documents |
| `send-email-common` | Send Email activities |
| `data-importer-common` | Data Importer templates |
| `database-connector-common` | DatabaseConnection documents, external SQL |
| `version-control` | Commits, branches, history. Always re-read the files fresh |

Modules can ship custom skills, so re-read the `read_skill` description per project rather than trusting this table.

## Project Editor (PED) Tools — Core Model Manipulation

| Tool | Purpose | Key Parameters |
|---|---|---|
| `list_modules` | List modules with `writable` and `fromMarketplace` flags | none |
| `ped_list_folder` | Browse module contents (documents + subfolders) | `moduleName`, `folderPath?` |
| `ped_find_document` | Search for documents by type in a module | `moduleName`, `documentType` |
| `ped_read_document` | Read a document or nested elements via JSON Pointer paths | `documentType`, `documentName?`, `paths?` |
| `ped_get_schema` | Get TypeScript-like schemas for element types | `elementTypes[]`, `kind?` |
| `ped_create_document` | Create one or more documents in one call | `documents[]` |
| `ped_update_document` | Apply set/add/remove/call operations in one call | `documentType`, `documentName?`, `operations[]` |
| `ped_check_errors` | Validate documents. Mandatory after the final change | `documents[]` |
| `ped_create_module` | Create a new module (only when explicitly requested) | `moduleName` |
| `install_marketplace_module` | Install a Marketplace module into the open project | `versionId`, `moduleName`, `conflictResolution` |

### `ped_get_schema` takes a `kind`

Easy to miss, and getting it wrong produces silent payload failures.

- `kind: "constructor"` (the default) is the flattened shape for **creating or adding**. It never includes methods. Property names can differ from what a read returns, for example `objects` instead of `objectCollection.objects`.
- `kind: "element"` is the full **read/update** shape, including callable methods. Use it to set properties the constructor does not expose, and before any `call` operation.

Because the two shapes differ, `$id(/path)` references differ too. Creation payloads use the constructor structure (`$id(/objects/0)`); update payloads use the read structure (`$id(/objectCollection/objects/0)`).

### `documentName` rules

| Document | What to pass |
|---|---|
| Domain model | Module name only: `"MyFirstModule"` |
| Project-level (`Navigation$NavigationDocument`, `Security$ProjectSecurity`, `Settings$ProjectSettings`) | Omit `documentName` entirely |
| Everything else | Fully qualified: `"MyFirstModule.ACT_Order_Save"` |

New document names must match `^[a-zA-Z_][a-zA-Z0-9_]*$`.

### Document types

`DomainModels$DomainModel`, `Microflows$Microflow`, `Workflows$Workflow`, `Enumerations$Enumeration`, `Navigation$NavigationDocument`, `Security$ProjectSecurity`, `Security$ModuleSecurity`, `Settings$ProjectSettings`, plus others discoverable through `ped_list_folder`.

Two are singletons that always exist in every module: `DomainModels$DomainModel` and `Security$ModuleSecurity`. Read them directly. Do not pass them to `ped_find_document`, and never create a domain model.

Pages are **not** handled by the `ped_*` tools. See below.

## Page Tools — Pages Only

| Tool | Purpose | Key Parameters |
|---|---|---|
| `pg_read_page` | Read a page or sub-sections via JSON Pointers (RFC 6901) | `moduleName`, `pageName`, `paths?`, `depth?` |
| `pg_patch_page` | Create a page, or patch one with JSON Patch (RFC 6902) | `moduleName`, `pageName`, `patches` |

Pages use their own LightPage structure, separate from PED. To create a page, send a single `replace` operation with an empty path and the full LightPage as its value. To edit an existing page, send targeted operations instead of a root replace, and make sure every path points at an element that already exists.

## File System Tools

| Tool | Purpose | Key Parameters |
|---|---|---|
| `glob` | List files matching a pattern. The pattern must start with a registered root | `pattern` |
| `read_file` | Read a file at a virtual path | `path`, `startLine?`, `endLine?` |
| `write_file` | Create or update a file. Omit `span` to replace the whole file | `path`, `newContent`, `span?` |

**Accessible file domains.** Read the `glob` tool description for the authoritative list, since it varies per project:

| Root | Contents |
|---|---|
| `/jsactions` | `<module_lowercase>/actions/<action>.js` |
| `/theme` | `/theme/web/` app overrides, `/theme/themesource/<module>/web/` module sources |
| `/themesource` | Atlas UI module SCSS/CSS, `design-properties.json`, `settings.json` |
| `/version-control` | `status.json`, `commits.json`, `changes.json` |
| `/edc-schemas` | Read-only DatabaseConnection schema metadata |
| `/pagegen/appearanceVFSPlugin` | Merged design property definitions per element type |
| `/pagegen/customWidgetsVFS` | Available custom widgets and their schemas |

Always run `glob` first to discover real paths, then `read_file` or `write_file`.

## Knowledge Base Tool

| Tool | Purpose | Key Parameters |
|---|---|---|
| `search_mendix_knowledge_base` | Search Mendix docs and marketplace knowledge | `query` |

Never use this to look up element schemas. It cannot answer schema questions. Use `ped_get_schema`.

## OQL and view entities

There are no dedicated OQL tools. `oql_generate` and `oql_read` were removed. Load the `view-entities` skill and work through the `ped_*` tools instead.

---

# Mandatory Workflow Rules

These rules MUST be followed on every operation to avoid errors and duplicates. They match what the Studio Pro 11.14 server enforces.

### 1. Load the relevant skill before building
Call `read_skill` for every skill matching the task before the first create or update. Skills carry hard constraints. Loading one afterwards is too late. `folder-structure` applies to every document creation.

### 2. Always check before creating
Run `ped_find_document` before `ped_create_document`. If a matching document exists, read and update it instead. Skip this for `DomainModels$DomainModel` and `Security$ModuleSecurity`: they are singletons that always exist, so read them directly.

### 3. Always get schemas before adding elements
Run `ped_get_schema` with all needed element types before constructing `documentContent` or `operations`. Pass `kind: "constructor"` for creates and adds, `kind: "element"` for reads, property updates the constructor does not expose, and `call` operations. Never guess schema structure from prior reads.

### 4. Validate once, at the end
Run `ped_check_errors` after **all** changes for the task are complete, not between intermediate steps. If errors appear, you get exactly one `ped_update_document` fix, then re-check. If errors persist, report and stop. Do not try a third time.

### 5. Pick the right `documentName`
Domain model: module name only (`"MyFirstModule"`). Project-level documents (`Navigation$NavigationDocument`, `Security$ProjectSecurity`, `Settings$ProjectSettings`): omit `documentName`. Everything else: fully qualified (`"MyFirstModule.ACT_Order_Save"`).

### 6. Use `$id(/path)` for cross-references
When referencing elements within the same document, use `"$id(/objects/0)"`. Indices are zero-based. Creation payloads follow the constructor structure; update payloads follow the read structure. Only use `$id` for `'id'`-kind references. Qualified-name and local-name references take names.

### 7. User associations always target Administration.Account
Never use `System.User` as an association target. Always use `Administration.Account`.

### 8. Batch operations in one call and never adjust indices
`ped_update_document` reorders same-path add and remove operations into descending-index order internally. Every `index` you pass refers to the array state you read. Do not offset indices to compensate for other operations in the same call.

### 9. Order matters when elements reference each other
Add the referenced element before the element that references it, for example entities before associations.

### 10. Additions and overwrites only
Supported operations are additions, creations, removals, and property overwrites. Remove only the specific element named in an error or by the user. Restructuring, recreating from scratch, or removing anything the user did not mention requires explicit approval first.

### 11. Avoid reserved words in names
Java keywords, plus `type`, `MendixObject`, `changedby`, `changeddate`, `context`, `createddate`, `currentUser`, `empty`, `guid`, `id`, `object`, `owner`, `submetaobjectname`, `con`, and predefined variables like `currentDeviceType`, `currentIndex`, `currentSession`, `latestError`, `latestHttpResponse`. Case does not matter, so `Type` and `TYPE` are also reserved. Use an alternative even when the user asks for a reserved word.

### 12. Respect module writability
Call `list_modules` and only write into modules flagged `writable`. System and Marketplace modules are read-only.

---

# Steering Instructions

## When to Load Steering Files

Two layers of guidance apply, and both are needed:

1. **Server skills** via `read_skill`. These ship with Studio Pro, track the current model API, and carry hard constraints. Load these first.
2. **Steering files** in this power. These add Kiro-side conventions, naming patterns, and integration recipes the server does not cover.

Where the two disagree, the server skill wins. It is generated from the running product.

| Working on... | `read_skill` first | Then load steering |
|---|---|---|
| Entities, attributes, associations, enumerations | `folder-structure` | `domain-model.md` |
| Microflows, nanoflows, ACT_, SUB_, DS_ | `microflow-common`, `microflow-expressions`, `folder-structure` | `microflows.md` |
| Validation microflows (VAL_) | `validation-microflow`, `microflow-common` | `microflows.md` |
| CRUD patterns | `microflow-common`, `page-gen-common` | `patterns-crud.md` |
| XPath constraints | `microflow-xpath` | `xpath-constraints.md` |
| Pages, widgets, layouts, master-detail | `page-gen-common`, `glyph-icons` | `pages.md` |
| SCSS/CSS theme and styling | `theming`, `design-properties` | `theme-styling.md` |
| Custom pluggable widgets (React/TypeScript) | none | `create-custom-widget.md` |
| Security roles, access rules | none | `security.md` |
| Navigation profiles, menus | `navigation`, `glyph-icons` | `navigation.md` |
| OQL queries, view entities | `view-entities` | `oql-queries.md` |
| OData inter-app data sharing | `view-entities` | `odata-data-sharing.md` |
| REST API calls from microflows | `microflow-common` | `rest-integration.md` |
| Complex REST (SPARQL, special auth, nested JSON) | `microflow-common` | `rest-sparql-integration.md` |
| Business Events / Kafka messaging | `microflow-common` | `business-events.md` |
| JavaScript actions | `javascript-action` | `javascript-actions.md` |
| Java actions | none | `java-actions.md` |
| Mendix AI agents | none | `agents.md` |
| Workflows | `workflow-common`, `workflow-update` | none |
| Data Importer templates | `data-importer-common` | none |
| External database connections | `database-connector-common` | none |
| Send Email activities | `send-email-common` | none |
| Unit tests for microflows | `microflow-unit-testing` | none |
| Commits, branches, history | `version-control` | none |
| System module entities | none | `system-module.md` |
| Project quality audit | none | `assess-quality.md` |

Rows with `none` in the skill column have no server-side equivalent. Rows with `none` in the steering column are handled entirely by the server skill.

---

# Core Principles (Always Apply)

0. **Confirm the approach: MCP vs MxCLI** — This power is for MCP-based workflows only. If the user mentions MxCLI, MDL scripts, or wants to use the `./mxcli` binary, stop and point them to the [Mendix MxCLI Power](https://github.com/Rvthof/awskiro-mxcli-power) instead. Do not use MCP tools for MxCLI-based workflows.

1. **Orient once per session** — read `mendix://studio-pro/system-prompt`, then call `list_modules`. Do not carry a module list over from a previous project.
2. **Read before writing** — `ped_read_document` for model documents, `pg_read_page` for pages, `glob` then `read_file` for theme and JS action files
3. **Describe changes in plain language** — never show raw JSON document structures in chat unless the user explicitly asks
4. **Batch everything independent** — schemas, document reads, and `read_skill` calls all go in the same turn. This is the single biggest speed win.
5. **Stop on persistent errors** — after one failed fix attempt, report the error and suggested solution to the user and stop

## License and support

This power integrates with the [Mendix Studio Pro MCP Server](https://docs.mendix.com/refguide/studio-pro-mcp-server/) (Apache-2.0).
- [Privacy Policy](https://www.mendix.com/trust/mendix-and-data-privacy/)
- [Support](https://mendix.com/support)