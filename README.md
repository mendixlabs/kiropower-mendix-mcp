# Mendix MCP Power

This Kiro Power enables AI-assisted Mendix development by connecting directly to the **Studio Pro MCP Server** — Mendix's built-in MCP server that exposes the same capabilities as Maia, Mendix's own AI assistant.

Instead of using a CLI binary or scripting language, all model changes (entities, microflows, pages, security, OQL, and more) are made through structured MCP tool calls against the live `.mpr` project file, with changes reflected in real time inside Studio Pro.

---

## How It Works

```
Kiro ──MCP tools──▶ Studio Pro MCP Server (localhost:7782) ──▶ .mpr project file
                                                               ◀── live updates in Studio Pro
```

1. Studio Pro runs an MCP server locally on a configurable port (`7782` unless taken; the status bar shows the active port)
2. Kiro connects to it via the MCP configuration in `.kiro/settings/mcp.json`
3. Kiro uses the MCP tools to read and write directly to the Mendix model
4. Every change appears live in Studio Pro — no restart or sync needed

---

## MCP or MxCLI?

There are two Mendix Kiro Powers — pick the one that matches how your project talks to Mendix:

| | This power — MCP | [Mendix MxCLI Power](https://github.com/Rvthof/awskiro-mxcli-power) |
|---|---|---|
| **Approach** | Studio Pro MCP server | CLI binary + MDL scripts |
| **How it works** | Kiro calls Mendix Studio Pro directly through MCP | Write `.mdl` files, validate and execute via `./mxcli` |
| **Setup** | Studio Pro MCP server running | `mxcli` binary in project root |

If you're not sure which to use: if Studio Pro is running with its MCP server enabled, you're in the right place. If your project root contains an `mxcli` binary, use the MxCLI power instead.

---

## Prerequisites

### 1. Use Studio Pro 11.10 or later

The Studio Pro MCP Server shipped in **11.10**. Earlier versions have no server to connect to. Page-editing tools arrived in 11.11, so 11.11+ is recommended.

Studio Pro must be signed in and have an internet connection for the MCP server to work.

### 2. Enable the MCP Server in Studio Pro

Navigate to **Preferences → AI → MCP Server** and check **Enable MCP Server**.

You can configure the port in the same tab. From 11.13 onward Studio Pro picks a free port automatically when the configured one is taken, so two instances can run side by side. **Read the active port from the Studio Pro status bar** rather than assuming `7782`.

For full details, see the official docs: [Studio Pro MCP Server](https://docs.mendix.com/refguide/studio-pro-mcp-server/)

### 3. Open your project in Studio Pro

The MCP server only runs while Studio Pro is open with your project loaded. Kiro cannot connect if Studio Pro is closed.

### 4. Verify the MCP config in this workspace

The power ships an `mcp.json` at its root. Kiro merges it into your workspace config at `.kiro/settings/mcp.json`, which is the file Kiro actually reads. Both hold the same content:

```json
{
  "mcpServers": {
    "localhost-7782": {
      "type": "http",
      "url": "http://localhost:7782/mcp",
      "disabled": false,
      "autoApprove": [
        "list_modules",
        "read_skill",
        "ped_list_folder",
        "ped_find_document",
        "ped_read_document",
        "ped_get_schema",
        "ped_check_errors",
        "pg_read_page",
        "glob",
        "read_file",
        "search_mendix_knowledge_base"
      ]
    }
  }
}
```

`autoApprove` covers read-only tools only. Writes still prompt: `ped_create_document`, `ped_create_module`, `ped_update_document`, `pg_patch_page`, `write_file`, `install_marketplace_module`.

If Studio Pro reports a different port in its status bar, update the `url` in `.kiro/settings/mcp.json` to match.

---

## What You Can Build

Tool names below are verified against Studio Pro 11.14.

| Capability | Tools Used |
|---|---|
| Domain model — entities, attributes, associations, enumerations | `ped_read_document`, `ped_update_document` |
| Microflows and nanoflows | `ped_create_document`, `ped_update_document` |
| Workflows | `ped_create_document`, `ped_update_document` |
| Pages and widgets | `pg_read_page`, `pg_patch_page` |
| Security — access rules, module roles | `ped_update_document` |
| Navigation — menus, home pages | `ped_update_document` |
| View entities and OQL | `read_skill` (`view-entities`) plus `ped_*` |
| REST API integration | `ped_create_document`, `ped_update_document` |
| OData inter-app data sharing | `ped_create_document`, `ped_update_document` |
| Business Events (Kafka pub/sub) | `ped_create_document`, `ped_update_document` |
| JavaScript actions | `glob`, `read_file`, `write_file` |
| Theme and design properties | `glob`, `read_file`, `write_file` |
| Java actions | Kiro file tools (`Read`, `Edit`, `Write`) |
| Mendix AI agents | `ped_create_document`, `ped_get_schema` |
| Custom pluggable widgets | npm / widget build tools |
| Marketplace module install | `install_marketplace_module` |
| Version control history | `glob`, `read_file` on `/version-control` |
| Mendix knowledge lookup | `search_mendix_knowledge_base` |

**Pages use their own tools.** `pg_read_page` and `pg_patch_page` work on a "LightPage" structure with JSON Patch (RFC 6902), not the PED document format. The `ped_*` tools do not handle pages.

**There are no OQL tools.** `oql_generate` and `oql_read` existed in earlier releases and are gone. OQL now runs through the `view-entities` server skill plus the `ped_*` tools.

### Newer document types (server-side support, no steering guide yet)

Studio Pro added these to Maia Make after this power was written. The MCP tools reach them, but there is no dedicated steering file, so expect to hand-hold Kiro more:

| Document type | Available from |
|---|---|
| Data Transformers | 11.12 |
| JSON Structures | 11.13 |
| Change Data Capture | 11.14 |

See [Maia Make Capabilities](https://docs.mendix.com/refguide/maia-make/) for the current list.

### Studio Pro features this power does not cover

- **Agent Skills** (11.11+): `SKILL.md` files under `skillssource/`, loaded by Maia inside Studio Pro
- **`AGENTS.md` instructions** (11.12+): project-level and module-level custom instructions Maia reads automatically
- **AI Agent Task workflow element** (11.14)
- **Custom LLM providers** (11.12, public beta): OpenAI-compatible or AWS Bedrock, configured per project

---

## Steering Guides

This power includes steering files that Kiro loads based on what you're working on:

| File | When it's used |
|---|---|
| `steering/domain-model.md` | Creating entities, attributes, associations, enumerations |
| `steering/microflows.md` | Building microflows and nanoflows |
| `steering/patterns-crud.md` | CRUD microflow and page patterns |
| `steering/xpath-constraints.md` | XPath syntax for retrieves and access rules |
| `steering/pages.md` | Creating and modifying pages, widgets, and layouts |
| `steering/theme-styling.md` | SCSS/CSS theme customization |
| `steering/create-custom-widget.md` | Building custom pluggable widgets (React/TypeScript) |
| `steering/security.md` | Configuring access rules and module roles |
| `steering/navigation.md` | Managing navigation profiles and menus |
| `steering/oql-queries.md` | Writing OQL queries for view entities |
| `steering/odata-data-sharing.md` | OData inter-app data sharing |
| `steering/rest-integration.md` | Calling external REST APIs from microflows |
| `steering/rest-sparql-integration.md` | Complex REST integrations (SPARQL, special auth, nested JSON) |
| `steering/business-events.md` | Event-driven messaging via Business Events / Kafka |
| `steering/javascript-actions.md` | Creating and editing JavaScript actions |
| `steering/java-actions.md` | Creating and editing Java actions |
| `steering/agents.md` | Setting up Mendix AI agents (Mendix 11.9+) |
| `steering/system-module.md` | System module entity reference |
| `steering/assess-quality.md` | Auditing Mendix project quality |

---

## Key Rules Kiro Follows

- **Always checks before creating** — runs `ped_find_document` to avoid duplicates
- **Always gets schemas** — runs `ped_get_schema` before constructing any new element
- **Always validates** — runs `ped_check_errors` after every change; stops and reports if errors persist
- **Never uses `System.User`** — always targets `Administration.Account` for user associations
- **Reads before writing** — understands current model state before making changes

---

## Troubleshooting

**Kiro can't connect to the MCP server**
- Make sure Studio Pro is open with your project loaded
- Check that the MCP server is enabled: Preferences → AI → MCP Server
- Confirm you are on Studio Pro 11.10 or later. Earlier versions have no MCP server
- Verify the port in `.kiro/settings/mcp.json` matches the port shown in the Studio Pro status bar

**The port keeps changing**
- From 11.13 Studio Pro takes the next free port when `7782` is busy, usually because another Studio Pro instance already holds it. Close the other instance or point `.kiro/settings/mcp.json` at the port in the status bar.

**Checking which tools the server actually exposes**
- Tool names can change between Studio Pro releases. To list what your version offers, with `<port>` from the status bar:

```bash
curl -s -X POST http://localhost:<port>/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

**Changes aren't appearing in Studio Pro**
- The MCP server reflects changes in real time — if you don't see them, try clicking into the affected document in Studio Pro to refresh the view

**"Module not found" errors**
- Module names are case-sensitive. Use `ped_list_folder` to confirm the exact module name

---

## Further Reading

- [Studio Pro MCP Server — Official Docs](https://docs.mendix.com/refguide/studio-pro-mcp-server/)
- [Maia Make Capabilities](https://docs.mendix.com/refguide/maia-make/)
- [Mendix AI Assistance (Maia)](https://docs.mendix.com/refguide/mendix-ai-assistance/)
- [Studio Pro Release Notes](https://docs.mendix.com/releasenotes/studio-pro/)
