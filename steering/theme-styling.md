---
inclusion: manual
---

# Theme & Styling — MCP Server Workflow

> **Important:** Load the `theming` MCP skill before changing styles, and `design-properties` before touching any `design-properties.json`.
> Theme files live behind the `glob` roots `/theme` and `/themesource`. Run `glob` first to get real paths, then `read_file` and `write_file`.

## When to Use This Guidance

Apply this whenever working with:
- SCSS customization (`theme/web/custom-variables.scss`, `theme/web/main.scss`)
- Design properties on widgets (DesignProperties, Class, Style)
- Module-level theme contributions (`themesource/<module>/web/`)
- Debugging styling or layout issues

---

## Directory Structure

```
<project>/
├── theme/web/                          # ✅ Project-level overrides — EDIT HERE
│   ├── main.scss                       # SCSS entry point (import chain)
│   ├── custom-variables.scss           # Project variable overrides
│   ├── exclusion-variables.scss        # Disable unwanted Atlas components
│   └── settings.json                   # Theme settings
│
├── themesource/                        # ✅ Module-level theme definitions — EDIT HERE
│   ├── atlas_core/web/
│   │   ├── design-properties.json      # Widget design property definitions
│   │   ├── _variables.scss             # Color/spacing/font variables
│   │   └── main.scss                   # Atlas core SCSS entry point
│   ├── atlas_web_content/web/
│   ├── datawidgets/web/
│   ├── productionfloormonitor/web/     # Custom module styles
│   └── <module_name>/web/
│       └── design-properties.json
│
├── theme-cache/web/                    # ❌ Build artifact — DO NOT EDIT
└── deployment/                         # ❌ Generated output — DO NOT EDIT
```

**Critical rule: Never edit files in `deployment/` or `theme-cache/`. These are generated.**

---

## Where to Make Changes

| Goal | File to edit |
|---|---|
| Override Atlas variables (colors, spacing, fonts) | `theme/web/custom-variables.scss` |
| Add project-wide custom CSS | `theme/web/main.scss` |
| Disable Atlas components | `theme/web/exclusion-variables.scss` |
| Add module-specific styles | `themesource/<module>/web/*.scss` |
| Define widget design properties | `themesource/<module>/web/design-properties.json` |

---

## SCSS Compilation Chain

`themesource/atlas_core/web/main.scss` imports in order:
1. Default variables (`atlas_core`)
2. Exclusion variables
3. Project custom variables (`theme/web/custom-variables.scss`)
4. Bootstrap framework
5. MXUI components
6. Core styles (base, animations, spacing, flex)
7. Widget-specific styles
8. Module-specific styles from `themesource/*/web/*.scss`

Variables with `!default` are overridden by later declarations. This means `theme/web/custom-variables.scss` overrides `themesource/atlas_core/web/_variables.scss`.

---

## Applying Styles via MCP Server

When setting styles on widgets through the MCP server tools, use these properties:

- **Class** — space-separated CSS class names (e.g., `"spacing-outer-top-large"`)
- **Style** — inline CSS string (e.g., `"color: red; font-weight: bold;"`)
- **DesignProperties** — key/value pairs from `design-properties.json`

### Design Property Keys Are Case-Sensitive

Keys must exactly match the `name` field in `design-properties.json`. Always check `themesource/atlas_core/web/design-properties.json` or the relevant module's `design-properties.json` to verify exact key names and allowed values.

---

## Caveats

### Never Apply Style Directly to a Text/Dynamic Text Widget

Applying `Style` directly to a text widget can cause build errors. Wrap it in a container instead — style the container, not the text widget.

### deployment/ Is Fully Generated

The `deployment/sass/` folder (including `main.scss`, `Atlas_Core.Atlas.scss`, etc.) is compiled output. Any manual edits will be overwritten on the next build. Always make changes in `theme/web/` or `themesource/` instead.

### theme-cache/ Is Also Generated

`theme-cache/web/` contains compiled CSS. Do not edit it directly.

---

## Checklist

- [ ] Never edit files under `deployment/` or `theme-cache/`
- [ ] Variable overrides go in `theme/web/custom-variables.scss`
- [ ] Custom module styles go in `themesource/<module>/web/`
- [ ] Design property keys are case-sensitive — verify against `design-properties.json`
- [ ] Never apply `Style` directly to text/dynamic text widgets — wrap in a container
- [ ] After SCSS changes, the app needs to be redeployed/rebuilt for changes to take effect
