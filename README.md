# g/d/n/a HubSpot Kiro Standards Template

AIDLC-driven development standards for HubSpot CMS projects using Kiro IDE. Clone this template for any HubSpot engagement — landing pages, support portals, email templates, blog themes.

## What's Included

### HubSpot-Specific Steering

| Document | Scope |
|----------|-------|
| `hubspot-development.md` | HubL standards, module patterns, CLI workflow, theme structure |
| `product.md` | Project template — fill in per engagement |
| `structure.md` | HubSpot theme directory layout and naming |
| `tech.md` | HubSpot CMS toolchain, CLI commands, deployment flow |

### g/d/n/a Universal Standards (inherited)

| Document | Scope |
|----------|-------|
| `coding-standards.md` | Core philosophy, error handling, DRY |
| `git-standards.md` | Conventional commits, branch strategy, PR requirements |
| `security-standards.md` | Secret management, input validation |
| `grc-compliance.md` | Data classification, audit logging, WCAG 2.1 AA |
| `testing-standards.md` | Testing approach (adapted for HubL/HTML context) |

### Spec Templates

Ready-to-use AIDLC spec templates for common HubSpot deliverables:

| Spec | Files |
|------|-------|
| `landing-page/` | requirements.md, design.md, tasks.md |
| `support-portal/` | requirements.md, design.md, tasks.md |

Copy a spec folder, fill in the requirements, iterate on design, then work through tasks.

### Hooks (20 files)

Pre-configured quality gates — accessibility audits, lint-on-save, security scanning, etc.

### Scripts

- `analyze-standards.js` — Compliance checker

## Quick Start

### New Engagement

1. Use this repo as a GitHub template (click "Use this template")
2. Clone your new repo
3. Customize the 3 project files:
   - `.kiro/steering/product.md` — What you're building
   - `.kiro/steering/structure.md` — Project-specific modules
   - `.kiro/steering/tech.md` — Portal IDs, deployment targets
4. Copy a spec template from `.kiro/specs/` and fill it in
5. Install HubSpot CLI: `pnpm add -D @hubspot/cli`
6. Auth: `pnpm dlx hs init`
7. Start building

### HubSpot CLI Commands

```bash
pnpm dlx hs upload src/theme theme    # Upload to portal
pnpm dlx hs watch src/theme theme     # Watch + auto-upload
pnpm dlx hs fetch theme src/theme     # Pull from portal
pnpm dlx hs create module src/theme/modules/[name]  # New module
```

### Deployment Flow

```
local dev → dev portal → staging portal → production portal
            (hs watch)    (hs upload)      (hs upload)
```

## Project Structure

```
project-root/
├── src/
│   └── theme/
│       ├── theme.json
│       ├── fields.json
│       ├── templates/
│       │   ├── layouts/
│       │   ├── pages/
│       │   ├── system/
│       │   └── partials/
│       ├── modules/
│       ├── css/
│       ├── js/
│       └── images/
├── .kiro/
│   ├── steering/          # AI steering (always active)
│   ├── specs/             # AIDLC spec templates
│   ├── hooks/             # Automation hooks
│   └── settings/
├── hubspot.config.yml.example
├── .gitignore
├── package.json
└── README.md
```

## Customization

Add client-specific steering in `.kiro/steering/`:

```yaml
---
title: [Client] HubSpot Standards
inclusion: always
---

# Client-Specific Requirements
- Brand colors and fonts
- Portal-specific module requirements
- Content governance rules
```

## License

Internal use only — g/d/n/a agency

---

*Maintained by g/d/n/a — global digital needs agency*
