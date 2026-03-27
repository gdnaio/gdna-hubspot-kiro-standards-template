---
title: HubSpot CMS Development Standards
inclusion: always
---

# HubSpot CMS Development Standards

## HubSpot CLI Workflow

All HubSpot development uses the official CLI. Never edit directly in the HubSpot Design Manager for production assets.

```bash
# Install CLI
pnpm add -D @hubspot/cli

# Auth (creates hubspot.config.yml)
pnpm dlx hs init

# Upload theme/module to portal
pnpm dlx hs upload src/theme theme

# Watch for local changes → auto-upload
pnpm dlx hs watch src/theme theme

# Fetch existing assets from portal
pnpm dlx hs fetch theme src/theme

# Create a new module scaffold
pnpm dlx hs create module src/theme/modules/new-module

# Preview
pnpm dlx hs sandbox create
```

## hubspot.config.yml

```yaml
defaultPortal: dev
portals:
  - name: dev
    portalId: 00000000
    authType: personalaccesskey
    personalAccessKey: >-
      [NEVER COMMIT — use env var or hs auth]
  - name: staging
    portalId: 00000001
    authType: personalaccesskey
  - name: prod
    portalId: 00000002
    authType: personalaccesskey
```

- Never commit `personalAccessKey` values — use `hs auth` or environment variables
- Add `hubspot.config.yml` to `.gitignore`
- Provide `hubspot.config.yml.example` with placeholder portal IDs

## Theme Structure

```
src/theme/
├── theme.json                 # Theme metadata and settings
├── fields.json                # Global theme fields (colors, fonts, spacing)
├── templates/
│   ├── layouts/
│   │   └── base.html          # Base layout (header/footer/scripts)
│   ├── pages/
│   │   ├── landing-page.html
│   │   ├── site-page.html
│   │   └── knowledge-base.html
│   ├── blog/
│   │   ├── blog-listing.html
│   │   └── blog-post.html
│   ├── system/
│   │   ├── 404.html
│   │   ├── 500.html
│   │   ├── password-prompt.html
│   │   └── search-results.html
│   └── partials/
│       ├── header.html
│       ├── footer.html
│       └── navigation.html
├── modules/
│   └── [module-name]/
│       ├── module.html         # HubL + HTML template
│       ├── module.css           # Scoped styles
│       ├── module.js            # Client-side behavior
│       ├── meta.json            # Module config (label, icon, categories)
│       └── fields.json          # Module field definitions
├── css/
│   ├── main.css
│   └── _variables.css
├── js/
│   └── main.js
└── images/
```

## HubL Standards

### Variable Output
```html
<!-- ✅ Good — escaped by default -->
{{ module.heading }}

<!-- ✅ Good — explicit filter when needed -->
{{ module.rich_text|safe }}

<!-- ❌ Bad — never use |safe on user-submitted content -->
{{ contact.custom_field|safe }}
```

### Conditionals
```html
{% if module.show_cta and module.cta_link %}
  <a href="{{ module.cta_link.url.href }}"
     {% if module.cta_link.open_in_new_tab %}target="_blank" rel="noopener noreferrer"{% endif %}>
    {{ module.cta_link.url.text }}
  </a>
{% endif %}
```

### Loops
```html
{% for item in module.features %}
  <div class="feature-card">
    <h3>{{ item.title }}</h3>
    <p>{{ item.description }}</p>
  </div>
{% endfor %}
```

### Macros (Reusable Snippets)
```html
{% macro render_button(link, style) %}
  {% if link.url.href %}
    <a href="{{ link.url.href }}"
       class="btn btn--{{ style|default('primary') }}"
       {% if link.open_in_new_tab %}target="_blank" rel="noopener noreferrer"{% endif %}>
      {{ link.url.text }}
    </a>
  {% endif %}
{% endmacro %}

{{ render_button(module.primary_cta, 'primary') }}
{{ render_button(module.secondary_cta, 'outline') }}
```

### Required HubL Tags in Templates
```html
{{ standard_header_includes }}
{{ standard_footer_includes }}
```
These are mandatory in every page template. Omitting them breaks HubSpot tracking, analytics, and CMS features.

## Module Development

### meta.json Pattern
```json
{
  "label": "Hero Banner",
  "css_assets": [],
  "external_js": [],
  "global": false,
  "host_template_types": ["PAGE", "BLOG_LISTING", "BLOG_POST"],
  "icon": "module_icon_banner",
  "is_available_for_new_content": true,
  "categories": ["BANNER"]
}
```

### fields.json Pattern
```json
[
  {
    "name": "heading",
    "label": "Heading",
    "type": "text",
    "required": true,
    "default": "Your Headline Here"
  },
  {
    "name": "subheading",
    "label": "Subheading",
    "type": "richtext",
    "required": false
  },
  {
    "name": "background_image",
    "label": "Background Image",
    "type": "image",
    "required": false,
    "responsive": true,
    "default": {
      "src": "",
      "alt": "",
      "loading": "lazy"
    }
  },
  {
    "name": "cta_link",
    "label": "Call to Action",
    "type": "link",
    "required": false,
    "supported_types": ["EXTERNAL", "CONTENT", "FILE", "EMAIL_ADDRESS"]
  },
  {
    "name": "style",
    "label": "Style",
    "type": "choice",
    "choices": [
      ["light", "Light"],
      ["dark", "Dark"],
      ["brand", "Brand Color"]
    ],
    "default": "light"
  }
]
```

### Field Type Reference
| Type | Use For |
|------|---------|
| `text` | Single-line text |
| `richtext` | Rich text editor (WYSIWYG) |
| `image` | Image picker with alt text |
| `link` | URL with link type options |
| `choice` | Dropdown select |
| `boolean` | Toggle switch |
| `number` | Numeric input |
| `color` | Color picker |
| `font` | Font selector |
| `group` | Field grouping container |
| `repeater` | Repeatable field groups |

## Accessibility in HubSpot Templates

- All `<img>` tags must have `alt` attributes (use `{{ module.image.alt }}`)
- All form inputs must have associated `<label>` elements
- Use semantic HTML: `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`
- Skip navigation link in base layout
- Color contrast: 4.5:1 minimum for body text
- Focus indicators on all interactive elements
- `aria-label` on icon-only buttons and links
- `lang` attribute on `<html>` tag

## SEO Requirements

- Every page template must support `<title>` and `<meta name="description">` via HubSpot page settings
- Use `<h1>` once per page, hierarchical heading structure
- Structured data (JSON-LD) for relevant page types
- Canonical URLs via `{{ content.absolute_url }}`
- Open Graph and Twitter Card meta tags in base layout
- Image `alt` text populated from module fields, never empty
- `robots` meta tag respecting HubSpot page settings

## Performance

- Lazy load images below the fold: `loading="lazy"`
- Inline critical CSS in base layout `<head>`
- Defer non-critical JS: `defer` attribute
- Minimize HubL `require_css` / `require_js` — bundle where possible
- Use HubSpot's built-in image optimization (auto-resize via `resize_image_url`)
- Avoid `{{ require_css("path") }}` in modules when styles can be in `module.css`

## Anti-Patterns

❌ Don't edit templates in HubSpot Design Manager for production code
❌ Don't use inline styles — use module.css or theme CSS
❌ Don't hardcode portal-specific IDs or URLs in templates
❌ Don't use `|safe` filter on user-submitted content
❌ Don't skip `standard_header_includes` / `standard_footer_includes`
❌ Don't create modules without `fields.json` — always define the content editor interface
❌ Don't use `<div>` for navigation — use `<nav>`
❌ Don't commit `hubspot.config.yml` with real access keys
