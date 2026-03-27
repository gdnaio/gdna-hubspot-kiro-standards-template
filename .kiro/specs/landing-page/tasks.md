# Landing Page — Tasks

## Task 1: Theme Scaffold
- [ ] Create `src/theme/theme.json` with metadata
- [ ] Create `src/theme/fields.json` with brand tokens (colors, fonts, spacing)
- [ ] Create `src/theme/templates/layouts/base.html` with standard includes, meta tags, skip nav
- [ ] Create `src/theme/css/_variables.css` and `main.css`
- [ ] Verify upload to dev portal with `hs upload`

## Task 2: Hero Banner Module
- [ ] Create `modules/hero-banner/` with all 5 files
- [ ] Fields: heading (text), subheading (richtext), cta (link), background_image (image), style (choice)
- [ ] Responsive: full-width desktop, stacked mobile
- [ ] Accessibility: semantic heading, alt text, focus-visible CTA

## Task 3: Feature Grid Module
- [ ] Create `modules/feature-grid/` with repeater field for features
- [ ] Each feature: icon (image), title (text), description (richtext)
- [ ] Responsive: 3-col → 2-col → 1-col
- [ ] Accessibility: list semantics, alt text on icons

## Task 4: Testimonial Carousel Module
- [ ] Create `modules/testimonial-carousel/` with repeater field
- [ ] Each testimonial: quote (richtext), name (text), title (text), company (text), photo (image)
- [ ] Vanilla JS carousel with prev/next, pause on hover
- [ ] Accessibility: aria-live region, keyboard controls, pause button

## Task 5: Form Section Module
- [ ] Create `modules/form-section/` with HubSpot form embed
- [ ] Fields: heading (text), description (richtext), form_id (text), privacy_text (richtext)
- [ ] Use `{% module "form" path="@hubspot/form" %}` for native form embedding
- [ ] Hidden fields for UTM passthrough

## Task 6: FAQ Accordion Module
- [ ] Create `modules/faq-accordion/` with repeater field
- [ ] Each FAQ: question (text), answer (richtext)
- [ ] Vanilla JS toggle with smooth expand/collapse
- [ ] Accessibility: `<details>`/`<summary>` or aria-expanded pattern

## Task 7: CTA Section Module
- [ ] Create `modules/cta-section/`
- [ ] Fields: heading (text), description (richtext), cta (link), style (choice)
- [ ] Reuse button macro from hero module

## Task 8: Landing Page Template
- [ ] Create `templates/pages/landing-page.html`
- [ ] Compose all modules with `{% dnd_area %}` for drag-and-drop editing
- [ ] Set default module order matching design spec
- [ ] Include `standard_header_includes` and `standard_footer_includes`

## Task 9: System Templates
- [ ] Create `templates/system/404.html`
- [ ] Create `templates/system/500.html`

## Task 10: QA and Deploy
- [ ] Upload to dev portal, verify all modules render
- [ ] Test form submission → CRM workflow
- [ ] Test responsive breakpoints
- [ ] Accessibility audit (keyboard nav, screen reader, contrast)
- [ ] Upload to staging for stakeholder review
- [ ] Stakeholder sign-off
- [ ] Upload to production
