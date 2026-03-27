# Landing Page — Design

## Page Template

- Template: `templates/pages/landing-page.html`
- Layout: `templates/layouts/base.html`
- Extends base layout with page-specific `{% block content %}`

## Module Composition (Top to Bottom)

1. `hero-banner` — Full-width hero with heading, subheading, CTA, optional background image
2. `feature-grid` — 3-4 column grid with icon, title, description per feature
3. `testimonial-carousel` — Rotating quotes with name, title, company, optional photo
4. `form-section` — HubSpot form embed with heading, description, and privacy note
5. `faq-accordion` — Expandable Q&A pairs
6. `cta-section` — Final conversion CTA with heading and button

## Module Field Design

Each module exposes fields in HubSpot's content editor so marketing can update without dev:
- Text fields for headlines and body copy
- Image fields with alt text for all visuals
- Link fields for CTAs (supports internal, external, email)
- Choice fields for style variants (light/dark/brand)
- Boolean toggles for optional sections
- Repeater fields for lists (features, FAQs, testimonials)

## Responsive Behavior

- Desktop: Full layout as designed
- Tablet (768px): Stack columns, reduce grid to 2-col
- Mobile (480px): Single column, hamburger nav, full-width CTAs

## Style Approach

- Use theme `fields.json` for brand colors, fonts, spacing
- Module-scoped CSS in `module.css` — no global style leaks
- CSS custom properties for theme tokens: `var(--brand-primary)`, `var(--font-heading)`
- No CSS frameworks — lightweight custom styles only
