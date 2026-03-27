---
title: HubSpot CMS Development Standards
inclusion: always
---

# HubSpot CMS Development Standards

## HubSpot CLI Setup

```bash
# Install CLI as project dep
pnpm add -D @hubspot/cli

# Auth — get a personal access key from Anvi or Will
pnpm --package=@hubspot/cli dlx hs auth

# Upload to portal
pnpm --package=@hubspot/cli dlx hs cms upload src/[project] [project]

# Watch for changes
pnpm --package=@hubspot/cli dlx hs cms watch src/[project] [project]

# Fetch from portal
pnpm --package=@hubspot/cli dlx hs cms fetch [project] src/[project]

# Delete remote directory
pnpm --package=@hubspot/cli dlx hs cms delete [path]

# List remote contents
pnpm --package=@hubspot/cli dlx hs cms list [path]
```

### CLI Gotchas
- Use `hs cms upload` NOT `hs upload` — the bare `upload` command doesn't exist in newer CLI versions
- Use `pnpm --package=@hubspot/cli dlx hs` NOT `pnpm dlx hs` — the package has multiple binaries (`hs`, `hscms`) and pnpm can't auto-resolve
- `hs init` requires an interactive terminal — won't work in Kiro's terminal. Run in your own terminal or use `hs auth`
- If `hs init` says config already exists, use `hs auth` instead

## hubspot.config.yml

```yaml
defaultPortal: gdnaio-hs
portals:
  - name: gdnaio-hs
    env: prod
    authType: personalaccesskey
    accountType: STANDARD
    portalId: [GET FROM ANVI OR WILL]
    personalAccessKey: [GENERATED VIA hs auth]
```

- NEVER commit `hubspot.config.yml` — it contains access keys
- It's already in `.gitignore`
- Provide `hubspot.config.yml.example` with placeholder values only
- Only include portals you actually have credentials for — placeholder entries with fake IDs cause validation errors

## Project Structure (NOT a Theme)

DO NOT use HubSpot themes (no `theme.json`, no theme-level `fields.json`). Themes trigger strict field validation that rejects `text`, `richtext`, `link`, and other common module field types. Instead, upload as plain CMS directories.

```
src/[project-name]/
├── templates/
│   ├── layouts/
│   │   └── base.html
│   ├── pages/
│   │   └── [page-name].html
│   ├── system/
│   │   ├── 404.html
│   │   └── 500.html
│   └── partials/
│       ├── header.html
│       ├── footer.html
│       └── navigation.html
├── modules/
│   └── [module-name].module/      ← .module suffix REQUIRED
│       ├── module.html
│       ├── module.css
│       ├── module.js
│       ├── meta.json
│       └── fields.json
├── css/
│   ├── main.css
│   └── _variables.css
├── js/
│   └── main.js
└── images/
```

## Critical: Module Directory Naming

Module directories MUST end with `.module` suffix:
- ✅ `hero-banner.module/`
- ❌ `hero-banner/`

Without the `.module` suffix, HubSpot won't recognize the directory as a module.

## Critical: No theme.json or Theme-Level fields.json

- ❌ DO NOT include `theme.json` in the project root
- ❌ DO NOT include a `fields.json` at the project root
- These trigger HubSpot's theme validation which rejects `text`, `richtext`, `link`, and other standard module field types
- Module-level `fields.json` inside `.module` directories are fine — they use module validation rules

## Upload Strategy

Upload CSS/JS/templates as a batch, then modules individually or as a batch. If module upload fails, upload each module directory separately:

```bash
# Upload everything except modules
pnpm --package=@hubspot/cli dlx hs cms upload src/[project]/css [project]/css
pnpm --package=@hubspot/cli dlx hs cms upload src/[project]/js [project]/js
pnpm --package=@hubspot/cli dlx hs cms upload src/[project]/templates [project]/templates

# Upload modules
pnpm --package=@hubspot/cli dlx hs cms upload src/[project]/modules [project]/modules
```

## meta.json — Valid Module Categories

Only these categories are valid in `meta.json`:
- `TEXT`
- `COMMERCE`
- `DESIGN`
- `FUNCTIONALITY`
- `FORMS_AND_BUTTONS`
- `BODY_CONTENT`
- `MEDIA`
- `BLOG`
- `SOCIAL`

❌ `BANNER` is NOT valid — use `DESIGN` instead

```json
{
  "label": "Hero Banner",
  "css_assets": [],
  "external_js": [],
  "global": false,
  "host_template_types": ["PAGE"],
  "is_available_for_new_content": true,
  "categories": ["DESIGN"]
}
```

## fields.json — Module Field Rules

### Reserved Field Names
- ❌ `body` — reserved by HubSpot. Use `body_content` or `body_text` instead

### Link Fields
- Use `url` type with `supported_types` for simple URL fields
- Avoid `link` type with complex default objects — they cause validation errors
- For CTA buttons, use separate `text` + `url` fields instead of a single `link` field

```json
{
  "name": "cta_text",
  "label": "Button Text",
  "type": "text",
  "default": "Get Started"
},
{
  "name": "cta_url",
  "label": "Button URL",
  "type": "url",
  "supported_types": ["EXTERNAL", "CONTENT"]
}
```

### Field Type Reference (Module fields.json)
| Type | Use For |
|------|---------|
| `text` | Single-line text |
| `richtext` | Rich text editor (WYSIWYG) |
| `image` | Image picker with alt text |
| `url` | URL field (use instead of `link` for simplicity) |
| `choice` | Dropdown select |
| `boolean` | Toggle switch |
| `number` | Numeric input |
| `color` | Color picker |
| `font` | Font selector |
| `group` | Field grouping container (use `occurrence` for repeaters) |

## HubL Standards

### Required Tags in Every Page Template
```html
{{ standard_header_includes }}
{{ standard_footer_includes }}
```

### Variable Output
```html
{{ module.heading }}
{{ module.rich_text_field|safe }}
```
Never use `|safe` on user-submitted content.

### Template Annotation (Required for Page Templates)
```html
<!--
  templateType: page
  label: My Page Template
  isAvailableForNewContent: true
-->
```

## Previewing Your Work

After uploading to HubSpot:
1. Go to your portal: `https://app.hubspot.com/portal/[PORTAL_ID]`
2. Marketing → Website → Website Pages → Create page
3. Select your template (e.g., "PC3 Migration Landing Page")
4. Click Preview to see it rendered
5. Or use Design Manager to preview individual templates/modules

## Accessibility

- All `<img>` tags must have `alt` attributes
- All form inputs must have `<label>` elements
- Semantic HTML: `<nav>`, `<main>`, `<article>`, `<section>`
- Skip navigation link in base layout
- Focus indicators on all interactive elements
- `aria-label` on icon-only buttons
- `lang` attribute on `<html>` tag

## Anti-Patterns

❌ Don't include `theme.json` — triggers strict theme field validation
❌ Don't include root-level `fields.json` — same issue
❌ Don't use `BANNER` category in meta.json — not valid
❌ Don't name fields `body` — reserved word
❌ Don't use `link` type with complex defaults — use `url` + `text` instead
❌ Don't use `pnpm dlx hs` — use `pnpm --package=@hubspot/cli dlx hs`
❌ Don't commit `hubspot.config.yml`
❌ Don't use `hs upload` — use `hs cms upload`
❌ Don't forget `.module` suffix on module directories
❌ Don't edit in HubSpot Design Manager for production code
❌ Don't hardcode portal-specific IDs in templates
