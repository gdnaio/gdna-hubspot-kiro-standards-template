---
title: Technology Stack
inclusion: always
---

# Technology Stack

## Platform

- HubSpot CMS Hub (Professional or Enterprise)
- HubSpot CLI (`@hubspot/cli`) for local development
- HubSpot Design Manager for preview/QA only

## Templating

- HubL (HubSpot Markup Language)
- HTML5 semantic markup
- CSS3 (no preprocessor required — HubSpot handles bundling)
- Vanilla JS (minimal — defer/async, no heavy frameworks)

## Key Tools

- `@hubspot/cli` — Local dev, upload, watch, sandbox
- HubSpot Sandbox — Preview and staging environments
- HubSpot Forms API — Lead capture and workflow triggers
- HubSpot CRM — Contact/deal pipeline integration

## Common Commands

```bash
# Install HubSpot CLI
pnpm add -D @hubspot/cli

# Authenticate with portal
pnpm dlx hs init

# Upload theme to portal
pnpm dlx hs upload src/theme theme

# Watch and auto-upload on save
pnpm dlx hs watch src/theme theme

# Fetch existing theme from portal
pnpm dlx hs fetch theme src/theme

# Create new module
pnpm dlx hs create module src/theme/modules/[module-name]

# Create sandbox for preview
pnpm dlx hs sandbox create
```

## Environment Variables

Required in `hubspot.config.yml` (never committed — see `.gitignore`):
- `portalId` — HubSpot portal ID per environment (dev/staging/prod)
- `personalAccessKey` — Auth token (use `hs auth` to generate)

## Deployment Flow

1. Local development with `hs watch`
2. Upload to sandbox/dev portal for preview
3. Stakeholder review in HubSpot preview
4. Upload to staging portal for QA
5. Upload to production portal for go-live
