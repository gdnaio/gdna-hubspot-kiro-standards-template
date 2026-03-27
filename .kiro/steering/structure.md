---
title: Project Structure
inclusion: always
---

# Project Structure

## HubSpot Theme Layout

```
project-root/
├── src/
│   └── theme/
│       ├── theme.json
│       ├── fields.json
│       ├── templates/
│       │   ├── layouts/
│       │   │   └── base.html
│       │   ├── pages/
│       │   │   ├── landing-page.html
│       │   │   └── [project-specific].html
│       │   ├── system/
│       │   │   ├── 404.html
│       │   │   └── 500.html
│       │   └── partials/
│       │       ├── header.html
│       │       ├── footer.html
│       │       └── navigation.html
│       ├── modules/
│       │   ├── hero-banner/
│       │   ├── feature-grid/
│       │   ├── cta-section/
│       │   ├── testimonial-carousel/
│       │   ├── faq-accordion/
│       │   ├── form-section/
│       │   └── [project-specific-modules]/
│       ├── css/
│       │   ├── main.css
│       │   └── _variables.css
│       ├── js/
│       │   └── main.js
│       └── images/
├── .kiro/
│   ├── steering/
│   ├── specs/
│   ├── hooks/
│   └── settings/
├── hubspot.config.yml.example
├── .gitignore
├── package.json
└── README.md
```

## Module Naming

- kebab-case directory names: `hero-banner/`, `feature-grid/`
- Each module is self-contained: `module.html`, `module.css`, `module.js`, `meta.json`, `fields.json`
- Group related modules by page type when the project grows large

## Template Naming

- Page templates: descriptive kebab-case — `landing-page.html`, `partner-signup.html`
- System templates: match HubSpot conventions — `404.html`, `500.html`, `search-results.html`
- Partials: component-style — `header.html`, `footer.html`, `navigation.html`
