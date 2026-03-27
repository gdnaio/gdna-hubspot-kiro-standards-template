# g/d/n/a HubSpot Kiro Standards

This `.kiro` configuration provides AIDLC-driven development standards for HubSpot CMS projects. It combines g/d/n/a universal coding standards with HubSpot-specific steering for HubL, module development, theme structure, and CMS deployment.

## HubSpot-Specific

| Document | What It Covers |
|----------|---------------|
| `hubspot-development.md` | HubL syntax, module patterns, fields.json, meta.json, CLI workflow, accessibility, SEO, performance |
| `product.md` | Template — fill in per engagement |
| `structure.md` | HubSpot theme directory layout |
| `tech.md` | HubSpot CMS toolchain and deployment flow |

## Inherited g/d/n/a Standards

| Document | What It Covers |
|----------|---------------|
| `coding-standards.md` | Core philosophy, error handling |
| `git-standards.md` | Conventional commits, branching |
| `security-standards.md` | Secret management, validation |
| `grc-compliance.md` | Data classification, WCAG, audit logging |
| `testing-standards.md` | Testing approach |

## Spec Templates

- `specs/landing-page/` — Requirements, design, tasks for landing pages
- `specs/support-portal/` — Requirements, design, tasks for knowledge bases

## How It Works

All steering files with `inclusion: always` load automatically into every Kiro interaction. Kiro will follow HubSpot best practices, enforce HubL patterns, and guide module development without you having to remind it.
