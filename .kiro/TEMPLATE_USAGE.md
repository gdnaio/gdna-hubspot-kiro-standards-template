# Using the g/d/n/a HubSpot Standards Template

## Quick Start

1. Use this repo as a GitHub template or clone it
2. Customize the 3 project steering files:
   - `.kiro/steering/product.md` — Your HubSpot project description
   - `.kiro/steering/structure.md` — Project-specific modules and templates
   - `.kiro/steering/tech.md` — Portal IDs, deployment targets
3. Copy a spec template from `.kiro/specs/landing-page/` or `.kiro/specs/support-portal/`
4. Fill in the spec requirements, iterate on design, then work through tasks
5. `pnpm install` and `pnpm dlx hs init` to authenticate

## Spec Workflow (AIDLC)

1. Copy a spec folder: `cp -r .kiro/specs/landing-page .kiro/specs/my-feature`
2. Fill in `requirements.md` with stakeholder input
3. Iterate on `design.md` — module composition, field design, responsive behavior
4. Break down `tasks.md` into implementable chunks
5. Work through tasks with Kiro — steering enforces HubSpot best practices automatically

## Adding New Spec Templates

Create a new folder in `.kiro/specs/` with `requirements.md`, `design.md`, and `tasks.md`. Common additions:
- `email-template/` — HubSpot email templates
- `blog-theme/` — Blog listing and post templates
- `website-redesign/` — Full theme overhaul
